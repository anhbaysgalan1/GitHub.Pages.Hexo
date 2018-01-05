---
title: Path-Inference-Filter(PIF) 算法调用的实现
date: 2018-01-04 17:29:57
categories:
    - 科研工作
tags:
    - 路网速度
---
Path Inference Filter(PIF, 道路推断滤波)算法是一种基于概率的路网匹配算法。其核心代码已经开源在 [GitHub][PIF-GitHub] 上，作者也提供了一个`example.py`文件，来介绍如何使用。我由于需要将此算法真实用于路网匹配中，因此需要根据实际路网对此算法进行调用。之前使用 ArcPy 写了一个，但是运行速度非常慢，而且跑着跑着电脑就自动关机了。所以现在就用 PostGIS 和 pgRouting 重新实现一个版本。
<!-- more -->
# PIF 的数学原理

此处详细情况请参考论文 *[The Path Inference Filter: Model-Based Low-Latency Map Matching of Probe Vehicle Data][PIF-Paper]* 。后面我会补充一个自己整理的简略版原理。

# PIF 的接口调用

PIF 核心库提供了一些类供我们使用。比如：

- `State`和`StateCollection`：描述状态值的类及状态值的集合。
- `LatLng`：表示经纬度的对象。
- `PathBuilder`：路径建立类。用于生成状态值之间的可达路径。
- `LearningTrajectory`：用于获取轨迹的描述。
- `TrajectoryViterbi1`：用于计算概率最大的轨迹。
- `TrajectorySmoother1`：用于滤波。

在调用过程中，可以创建`PathBuilder`的子类，来实现自己的路径搜索。子类中只需要实现方法`getPaths()`，此方法用于在方法`getPathsBetweenCollections()`中调用以获取路径。必须实现，否则会抛出`NotImplementedError`的错误。

`State`、`StateCollection`、`LatLng`三个类可以直接使用。对于我们拿到的 GPS 数据来说，一般都会有一些附加的属性值，例如速度、方向、车辆状态等。可以创建自己的继承自`LatLng`和`State`的子类，添加这些属性。

# 利用 PostGIS 和 pgRouting 实现调用 PIF 算法

借助 PostGIS 和 pgRouting 可以实现路径搜索功能，而使用 PostGIS 提供的大量的关于几何和地理数据的函数，可以方便实现对 PIF 算法的调用。但是这里需要编写的数据库函数，需要一定的数据库编程的能力。

## 准备路网数据

假设我们要导入的路网数据名字叫`road`。一般情况下，我们拿到的数据，坐标系为 WGS84 或常用的投影坐标系（如CGCS2000）。在 WGS84 坐标系下的数据，以`Geography`类型存储在数据库中，在投影坐标系下的数据，以`Geometry`类型存储在数据库中。在 [PostGIS][PostIGS-Doc] 的文档中，建议使用`Geometry`类型存储数据，以减小计算量。

> The geography type allows you to store data in longitude/latitude coordinates, but at a cost: there are fewer functions defined on GEOGRAPHY than there are on GEOMETRY; those functions that are defined take more CPU time to execute. 
> 
> *Geography 类型允许你以经纬度坐标的方式存储数据，但是代价是：Geography 类型比 Geometry 类型的函数少；Geography 类型的函数执行起来消耗更多的 CPU 时间。*
>
> The type you choose should be conditioned on the expected working area of the application you are building. Will your data span the globe or a large continental area, or is it local to a state, county or municipality?
> 
> - If your data is contained in a small area, you might find that choosing an appropriate projection and using GEOMETRY is the best solution, in terms of performance and functionality available.
> - If your data is global or covers a continental region, you may find that GEOGRAPHY allows you to build a system without having to worry about projection details. You store your data in longitude/latitude, and use the functions that have been defined on GEOGRAPHY.
> - If you don't understand projections, and you don't want to learn about them, and you're prepared to accept the limitations in functionality available in GEOGRAPHY, then it might be easier for you to use GEOGRAPHY than GEOMETRY. Simply load your data up as longitude/latitude and go from there.

因此，我们采用投影坐标系的路网数据。

### 数据导入

导入时数据的方法非常简单，使用 PostGIS Shapefiel Import/Export Manager(PostGIS 2.0 Shapefile and DBF Loader Exporter) 工具导入即可。

{% asset_img PostGIS-Shp-Tool.png %}

在这个工具上设置数据库的连接，选择要导入的 shp 文件。选定 shp 文件后，最好点击 Options 按钮打开选项，选中最后一个复选框。

{% asset_img shp-options.png %}

确定后，点击 Improt 按钮开始导入。等待其完成即可。

需要注意的是，shp 文件的名字，会作为最终导入到数据库中表的名字。

### 拓扑关系建立

我们导入的数据集，在网络结构中数据“边”类型，pgRouting 称之为 *edge*。pgRouting 需要知道每个 edge 的起点和终点的序号是什么，需要知道路径的长度是多少。需要给`road`数据集添加三个字段：`source`、`target`和`length`：

``` SQL
ALTER TABLE road ADD COLUMN source integer;
ALTER TABLE road ADD COLUMN target integer;
ALTER TABLE road ADD COLUMN length double precision;
UPDATE road set length = ST_Length(geom); -- 为路段长度赋值
```

当然这两个字段的名字可以自己取。路径长度字段如果已经有了，也可以不用重新建立。

然后是拓扑关系的建立。使用 pgRouting 提供的函数 [`pgr_createTopology()`][create-topology-ref] 来创建拓扑，该函数有以下参数：

参数名|说明
------|---
`edge_table` | 路网表名（也可以包含数据库名）。文本类型。
`tolerance` | 路段不连续误差。8字节浮点数类型。（一般长度单位测量误差最高是毫米级的，因此将该参数设置在0.001左右即在理论上可以包含任何连续的路段，避免测量误差。如果不忽略测量误差，设置在0.001以下即可。如果忽略更大的测量误差，设置在0.001以上）
`the_geom` | 路网表中 Geometry 列的名称（默认是"the_geom"）。文本类型。
`id` | 路网表中主键列的名称（默认是"id"）。文本类型。
`source` | 路网表中 source 列的名称（默认是"source"）。文本类型。
`target` | 路网表中 target 列的名称（默认是"target"）。文本类型。
`rows_where` | 用于选择一个子集或多行的 SELECT 条件。默认值是选择所有 source 和 target 为空的行。文本类型。
`clean` | 是否清除之前的拓扑关系（默认是 false ）。布尔型。

具体调用实例为：

``` SQL
SELECT pgr_createTopology('public.road', 0.001, 'geom', 'gid');
```

路网数据准备完成。

## 获取每个 GPS 点对应的状态值

状态值的获取是在一个路段上计算一个联合分布 $$ π(x|g) = ω(g|x)Ω(x) $$ 其中 $ x $ 是状态值，$ g $ 是 GPS 观测值。但是由于分布 $ ω(g|x) $ 服从正态分布，$ Ω(x) $ 在没有先验知识的情况下，服从均匀分布，因此直接取路段上距离 GPS 观测值最近的点即可。

### 几个需要用到的函数

# 利用 ArcPy 实现调用 PIF 算法



[PostIGS-Doc]: http://postgis.net/docs/manual-2.4/using_postgis_dbmanagement.html#PostGIS_Geography
[PIF-Paper]: http://bayen.eecs.berkeley.edu/sites/default/files/journals/The_Path_Inference_Filter.pdf
[PIF-github]: https://github.com/tjhunter/Path-Inference-Filter
[create-topology-ref]: http://docs.pgrouting.org/2.4/en/pgr_createTopology.html#pgr-createtopology