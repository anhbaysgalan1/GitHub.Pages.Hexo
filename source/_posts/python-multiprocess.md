---
title: Python 多进程编程
date: 2018-07-09 18:53:07
tags:
---
# Python 多进程对效率的提升

一篇[《Python 中单线程、多线程和多进程的效率对比实验》][multiprocess-efficiency]的文章中提到：

> Python是运行在解释器中的语言，有一个全局锁（GIL），
> 在使用多进程(Thread)的情况下，不能发挥多核的优势。
> 而使用多进程(Multiprocess)，则可以发挥多核的优势真正地提高效率。

文章中对 Python 在多线程、多进程的效率进行了对比：

| 操作类型   | CPU 密集型     | IO 密集型      | 网络请求密集型 |
| ---------- | -------------: | -------------: | -------------: |
| 线性操作   | 94.91824996469 | 22.46199995279 | 7.3296000004   |
| 多线程操作 | 101.1700000762 | 24.8605000973  | 0.5053332647   |
| 多进程操作 | 53.8899999857  | 12.7840000391  | 0.5045000315   |

可见：

1. 多线程操作只在网络请求密集型操作中具有非常明显的优势，其开销小于多进程，可用于网络爬虫。
2. 多进程操作在各种操作中都有效率提升，在 IO 密集型操作中的优势更大。

最近在处理一套出租车数据，出租车数据量非常大，自己搭建数据库，
查询效率非常低。因此采用 Python 脚本进行处理，

# Python 开启多进程

Python 中的 `multiprocess` 包提供了多进程支持。可以使用三种方法来创建进程。

## 创建 Process 对象

最简单的开启 Python 进程的方法，是直接构造 `multiprocess.Process` 对象

```python
from multiprocess import Process

process = Process()
```

### 构造参数

`Process` 对象在构造时主要接收三个参数：

- `target`：进程调用的函数；
- `args`：进程调用函数时给函数传递的参数，为一个元祖；
- `name`：别名。

### 属性

`Process` 的类型有以下属性：

- `daemon`：当这个属性设置为 `True` 时，子进程会随着主进程的结束而结束。否则，主进程结束后，子进程依然会继续进行；
- `exitcode`：进程在运行时为 None 、如果为 –N ，表示被信号 N 结束；
- `name`
- `pid`
- `authkey`

### 方法

`Process` 有如下方法：

- `start()`：调用 `start()` 函数时，子进程开始执行，主进程继续执行。
- `join()`：“[阻塞当前进程，直到调用 join 方法的那个进程执行完，再继续执行当前进程。][join-explain]”
- `run()`：当构造时如果没有制定 `target` 参数，那么 `start()` 方法默认执行 `run()` 函数。
- `is_alive()`：判断当前进程是否活动。

### 调用示例

```python
from multiprocess import Process
import os
import math

def work_fun(work_list):
    pass

def distrib_works(work_list, process_num):
    group_length = math.ceil(len(filename_list) / process_num)
    return [work_list[(i*group_length):((i+1)*group_length)] for i in range(process_num)]


work_list = os.listdir("../data")
process_num = 4
group = distrib_works(work_list, process_num)

process_list = [Process(target=work_fun, args=(g,)) for g in group_list]
for p in process_list:
    p.daemon = True
    p.start()
for p in process_list:
    p.joi()
```

## 将进程定义为类

利用 Python 面向对象的特性，我们可以创建一个类，继承 `Process` 类，
将一些数据直接在构造的时候保存下来，可以无需在调用的时候传入。
例如，当我们在多进程程序中使用 `tqdm` 库显示进度条时，会用到其 `position`
参数来指定当前进度条在控制台中显示的位置，这个参数的值，我们可以直接保存在进程类中，
无需调用的时候再传入。

将进程定义为类的方法如下：

```py
from multiprocess import Process

class MyProcess(Process):
    position = 0
    works = None

    def __init__(self, position, works)
        Process.__init__(self)
        self.position = poisition
        self.works = works

    def run():
        pass
```

如果在构造函数中，调用的 `Process` 的构造函数没有指定 `target`，
进程同样默认执行 ***不带参数的 `run` 函数，即使你的 `run` 函数定义了形参！***

在创建进程时，只需要将原来调用的 `Process` 的构造函数，改为调用 `MyProcess` 的构造函数即可。
这种创建进程方式的实例如下：

```py
from multiprocess import Process, Lock

def distrib_works(work_list: List[str], process_num) -> List[List[str]]:
    group_length = math.ceil(len(work_list) / process_num)
    return [g for g in [work_list[(i*group_length):((i+1)*group_length)] for i in range(process_num)] if len(g) > 0]

class FindTargetTaxiProcess(multiprocessing.Process):

    def __init__(self, input_files, index, lock, log_file):
        multiprocessing.Process.__init__(self, target=find_target, args=(lock, log_file))
        self.input_files = input_files
        self.index = index

    def find_target(self, lock, log_file):
        for filename in self.input_files:
            with open(filename) as in_file:
                for row in tqdm(in_file, ncols=80, position=self.index):
                    cells = row.split(",")
                    if int(cells[0]) == 11865:
                        with lock:
                            with open(log_file, mode="a") as log:
                                print(row, file=log)

if __name__ == '__main__':
    LOG_FILE = r"E:\出租车点\上下车点\scripts\data\find_error.log"
    lock = multiprocessing.Lock()
    # ROOT_DIR = "../data/201502/temp"
    ROOT_DIR = r"E:\出租车点\201502\RawCSV"
    INPUT_FILES = [os.path.join(ROOT_DIR, f) for f in os.listdir(ROOT_DIR)]
    GROUP_LIST = distrib_works(INPUT_FILES, 4)
    PROCESS_LIST = [FineTargetTaxiProcess(element, i, lock) for i, element in enumerate(GROUP_LIST)]
    for process in PROCESS_LIST:
        process.start()
```

> 这种定义为类的方式有一个好处，在用 VSCode 调试的时候，在子进程中打断点是无效的。
> 如果用这种方式，可以将调用的 `start()` 函数改为 `run()` 或其他实际进程执行的函数，
> 这样就可以调试进程内部了。当解决了 Bug 后，就可以换回 `start()` 函数并行执行。

## 进程池（Pool）

可以发现，上面两种创建进程的方式，都是用到了一个 `distrib_works()` 函数来分配各个进程的任务。
这一过程可以被一个叫做进程池的类型代替。

### 进程池构造

构造进程池的方法非常简单，导入 `Pool` 之后，直接构造 `Pool` 对象。
构造时可以指定最多的进程数量，默认是 CPU 核心数。

```py
from multiprocess import Pool

p = Pool(process=6)
```

### 进程池方法

`Pool` 类型主要有以下方法：

- `apply_async()` 和 `apply()`：这两个函数都是让进程池开始执行任务，`apply_async()` 是非阻塞的（主进程继续执行），`apply()` 是阻塞的（主进程等待子进程执行完成后继续执行）。
- `close()`：关闭进程池，不再接收新任务。
- `join()`：[主进程阻塞，等待子进程的退出， join方法要在close或terminate之后使用。][python多进程-cnblogs]

### 进程池调用示例

将上面一段使用 `FindTargetTaxiProcess` 类编写的代码用 `Pool` 重写：

```py
from multiprocess import Process, Pool, Lock

def find_target(in_file, lock, log_file):
    with open(filename) as in_file:
        for row in tqdm(in_file, ncols=80, position=self.index):
            cells = row.split(",")
            if int(cells[0]) == 11865:
                with lock:
                    with open(log_file, mode="a") as log:
                        print(row, file=log)

if __name__ == '__main__':
    LOCK = multiprocessing.Lock()
    LOG_FILE = r"E:\出租车点\上下车点\scripts\data\find_error.log"
    ROOT_DIR = r"E:\出租车点\201502\RawCSV"
    INPUT_FILES = [os.path.join(ROOT_DIR, f) for f in os.listdir(ROOT_DIR)]
    POOL = Pool(process=6)
    for f in INPUT_FILES:
        POOL.apply_async(find_target, (f, LOCK, LOG_FILE))
    POOL.close()
    POOL.join()
```

[multiprocess-efficiency]:https://segmentfault.com/a/1190000007495352
[join-explain]:https://www.cnblogs.com/lipijin/p/3709903.html
[python多进程-cnblogs]:http://www.cnblogs.com/kaituorensheng/p/4445418.html