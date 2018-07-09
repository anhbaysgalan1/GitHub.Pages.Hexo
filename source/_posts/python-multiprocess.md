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

Python 中的 `multiprocess` 包提供了多进程支持。

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

- `start`：调用 `start` 函数时，子进程开始执行，主进程继续执行。
- `join`：“[阻塞当前进程，直到调用 join 方法的那个进程执行完，再继续执行当前进程。][join-explain]”
- `run`：当构造时如果没有制定 `target` 参数，那么 `start` 方法默认执行 `run` 函数。
- `is_alive`：判断当前进程是否活动。

### 调用示例

```python
from multiprocess import Process
import os
import math

def work_fun(work_list):
    pass

def distribute_work(work_list, process_num):
    group_length = math.ceil(len(filename_list) / process_num)
    return [work_list[(i*group_length):((i+1)*group_length)] for i in range(process_num)]


work_list = os.listdir("../data")
process_num = 4
group = distribute_work(work_list, process_num)

process_list = [Process(target=work_fun, args=(g,)) for g in group_list]
for p in process_list:
    p.daemon = True
    p.start()
for p in process_list:
    p.joi()
```

[multiprocess-efficiency]:https://segmentfault.com/a/1190000007495352
[join-explain]:https://www.cnblogs.com/lipijin/p/3709903.html