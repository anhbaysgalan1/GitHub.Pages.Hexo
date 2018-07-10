---
title: Python 多进程编程
date: 2018-07-09 18:53:07
tags:
    - Python
categories: Python
---
Python 在处理大数据的时候，启用多进程是有效提高计算效率的手段。
Python 已经提供了非常好用的 multiprocess 包来支持多进程编程，
但是在多进程编程时仍然会遇到一些难以处理的问题，需要一些技巧来解决。
<!-- more -->
目录：

- [Python 多进程对效率的提升](#python-%E5%A4%9A%E8%BF%9B%E7%A8%8B%E5%AF%B9%E6%95%88%E7%8E%87%E7%9A%84%E6%8F%90%E5%8D%87)
- [Python 开启多进程](#python-%E5%BC%80%E5%90%AF%E5%A4%9A%E8%BF%9B%E7%A8%8B)
    - [创建 Process 对象](#%E5%88%9B%E5%BB%BA-process-%E5%AF%B9%E8%B1%A1)
        - [构造参数](#%E6%9E%84%E9%80%A0%E5%8F%82%E6%95%B0)
        - [属性](#%E5%B1%9E%E6%80%A7)
        - [方法](#%E6%96%B9%E6%B3%95)
        - [调用示例](#%E8%B0%83%E7%94%A8%E7%A4%BA%E4%BE%8B)
    - [将进程定义为类](#%E5%B0%86%E8%BF%9B%E7%A8%8B%E5%AE%9A%E4%B9%89%E4%B8%BA%E7%B1%BB)
    - [进程池（Pool）](#%E8%BF%9B%E7%A8%8B%E6%B1%A0%EF%BC%88pool%EF%BC%89)
        - [进程池构造](#%E8%BF%9B%E7%A8%8B%E6%B1%A0%E6%9E%84%E9%80%A0)
        - [进程池方法](#%E8%BF%9B%E7%A8%8B%E6%B1%A0%E6%96%B9%E6%B3%95)
        - [进程池调用示例](#%E8%BF%9B%E7%A8%8B%E6%B1%A0%E8%B0%83%E7%94%A8%E7%A4%BA%E4%BE%8B)
- [多进程共享资源](#%E5%A4%9A%E8%BF%9B%E7%A8%8B%E5%85%B1%E4%BA%AB%E8%B5%84%E6%BA%90)
    - [锁](#%E9%94%81)
        - [互斥锁（Lock）](#%E4%BA%92%E6%96%A5%E9%94%81%EF%BC%88lock%EF%BC%89)
        - [可重入锁（RLock）](#%E5%8F%AF%E9%87%8D%E5%85%A5%E9%94%81%EF%BC%88rlock%EF%BC%89)
        - [条件锁（Condition)](#%E6%9D%A1%E4%BB%B6%E9%94%81%EF%BC%88condition)
        - [信号量（Semaphore）](#%E4%BF%A1%E5%8F%B7%E9%87%8F%EF%BC%88semaphore%EF%BC%89)
    - [共享变量](#%E5%85%B1%E4%BA%AB%E5%8F%98%E9%87%8F)
        - [multiprocess 包内置类型](#multiprocess-%E5%8C%85%E5%86%85%E7%BD%AE%E7%B1%BB%E5%9E%8B)
        - [通过 Manager 创建共享变量](#%E9%80%9A%E8%BF%87-manager-%E5%88%9B%E5%BB%BA%E5%85%B1%E4%BA%AB%E5%8F%98%E9%87%8F)
        - [通过 Manager 管理](#%E9%80%9A%E8%BF%87-manager-%E7%AE%A1%E7%90%86)
- [进程间通信](#%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1)
    - [通过事件（Event）通信](#%E9%80%9A%E8%BF%87%E4%BA%8B%E4%BB%B6%EF%BC%88event%EF%BC%89%E9%80%9A%E4%BF%A1)
    - [通过队列（Queue）通信](#%E9%80%9A%E8%BF%87%E9%98%9F%E5%88%97%EF%BC%88queue%EF%BC%89%E9%80%9A%E4%BF%A1)
    - [通过管道（Pipe）通信](#%E9%80%9A%E8%BF%87%E7%AE%A1%E9%81%93%EF%BC%88pipe%EF%BC%89%E9%80%9A%E4%BF%A1)
- [其他](#%E5%85%B6%E4%BB%96)
    - [tqdm 多进度条](#tqdm-%E5%A4%9A%E8%BF%9B%E5%BA%A6%E6%9D%A1)
    - [Windows 上 Lock 对象的异常](#windows-%E4%B8%8A-lock-%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%BC%82%E5%B8%B8)

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
                        print(row)

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

# 多进程共享资源

当我们把多个任务分解到 $n$ 个进程上执行时，这 $n$ 个进程往往会存在某种共享的资源，
如共享一个控制台、文件系统、列表或字典。这里存在两个问题：

- 当多个进程同时访问这些资源时，就会产生冲突。例如，两个进程同时对控制台输出文本，写入的结果可以错综复杂，并不是两段文本的顺序组合。
- 各个进程有自己的内存空间，变量无法共享。例如，当想要利用多个进程操作主进程的一个列表时，各个进程操作结束后，主进程仍然是原来的状态。

这两个问题的解决，前者靠“锁”机制，后者靠“共享变量”机制。

## 锁

对于冲突的情况，当使用 tqdm 显示多个进度条时比较明显。在 Windows 上，由于
“tqdm 无法获取默认锁”，因此控制台输出会比较乱，下面是一段程序在 Windows 上运行的效果：

```plain
λ python3 find_errors.py
Process 0: 0it [00:00, ?it/s]
Process 1: 0it [00:00, ?it/s]
Process 0: 273516it [00:00, 523719.79it/s]
Process 0: 995883it [00:01, 510379.67it/s]
Process 0: 1107387it [00:02, 510326.10it/s]
Process 0: 1224813it [00:02, 512761.81it/s]
Process 0: 3483799it [00:06, 539191.83it/s]
Process 1: 3683852it [00:06, 571536.15it/s]
Process 0: 3550015it [00:06, 540296.03it/s]
Process 0: 3615558it [00:06, 540947.45it/s]
Process 0: 3742521it [00:06, 542112.37it/s]
```

而在 Linux 系统中的运行结果是

```plain
Process 0: 2045720it [00:03, 647073.52it/s]
Process 1: 2092184it [00:03, 661530.01it/s]
Process 2: 2065411it [00:03, 652446.31it/s]
Process 3: 2093610it [00:03, 661782.04it/s]
```

可见在访问共享资源的时候，加锁是非常有必要的。

### 互斥锁（Lock）

`Lock` 属于“互斥锁”，即[保证在任一时刻，只能有一个线程访问该对象。][使用Lock互斥锁]
通过 `Lock` 类型创建互斥锁后，将其传递到子进程内部，即可在子进程中使用。

使用 `Lock` 时，可以使用 `with` 语句加锁， `with` 语句块执行完成后自动解锁；
也可以通过其 `acquire()` 函数来加锁，使用 `release()` 函数解锁。

使用 `with` 语句进行加锁的示例代码如下：

```py
def run(self):
    for filename in self.input_files:
        with open(filename, encoding="GB2312") as in_file:
            for row in tqdm(in_file):
                cells = row.split(",")
                if int(cells[0]) == 11865:
                    with self.lock:
                        with open(TARGET_TAXI_FILE, mode="a") as log:
                            print(row, file=log)
```

> 这段代码在 Windows 上运行时，子进程内部的 lock 和 主进程传递进去的 lock 的 id 值不相同。
> 但是在 Linux 系统上时相同的。因此 Windows 上这段代码有可能会出错。
> 不过当文件被一个进程打开时，是无法被另一个进程打开的，因此这段程序的结果倒没出什么错。

### 可重入锁（RLock）

互斥锁可以解决简单的避免资源冲突的问题，但当一个线程加锁后仍需要再次访问共享资源时，
就形成了嵌套锁，而使用互斥锁时就形成了“死锁”问题。这时我们需要使用 `RLock` 类型，
即“可重入锁”。

> 死锁的含义是：是指两个或两个以上的进程在执行过程中，因争夺资源而造成的一种互相等待的现象，
> 若无外力作用，它们都将无法推进下去。
> 此时称系统处于死锁状态或系统产生了死锁，这些永远在互相等待的进程称为死锁进程。
> 避免死锁主要方法是正确有序的分配资源。

`multiprocess` 中的 `RLock` 类型与 `Lock` 类型的区别在于：
[RLock允许在同一线程中被多次申请。而 Lock 却不允许这种情况。][Python中Lock与RLock]
因此，[如果使用 `RLock` ，那么 acquire() 和 release() 必须成对出现][Python中Lock与RLock]，
调用了几次 `acquire()`，就需要调用几次 `release()`。

### 条件锁（Condition)

条件同步机制是指：线程 $B$ 等待特定条件 $C$ ，而另一个线程 $A$ 发出特定条件满足的信号 $C$ 。
$B$ 在收到信号 $C$ 时，继续执行。

可以通过“生产者-消费者”模型来理解这一过程。
生产者获取锁，生产一个随机整数，通知消费者并释放锁。
消费者获取锁，如果有整数则消耗一个整数并释放锁，如果没有就等待生产者继续生产。

示例代码如下（参考[《Python 线程同步机制》][Python线程同步机制]并进行修改）：

```py
import multiprocess


class Producer(multiprocessing.Process):
    def __init__(self, productList, condition):
        multiprocessing.Process.__init__(self)
        self.productList = productList  # type: List
        self.condition = condition  # type: multiprocess.Condition

    def run(self):
        while True:
            product = random.randint(0, 100)
            with self.condition:
                print("条件锁：被 生产者 获取")
                self.productList.append(product)
                print(f"生产者：产生了 {product}。")
                print("生产者：唤醒消费者线程")
                self.condition.notify()
                print("条件锁：被 生产者 释放")
            time.sleep(1)


class Customer(multiprocessing.Process):

    def __init__(self, productList, condition):
        multiprocessing.Process.__init__(self)
        self.productList = productList  # type: List
        self.condition = condition  # type: multiprocess.Condition

    def run(self):
        while True:
            with self.condition:
                print("条件锁：被 消费者 获取")
                while True:
                    if self.productList:
                        product = self.productList.pop()
                        print(f"消费者：消费了 {product}")
                        break
                    print("消费者：等待生产者")
                    self.condition.wait()
                print("条件锁：被 消费者 释放")


def main():
    manager = multiprocessing.Manager()
    productList = manager.list()
    condition = multiprocessing.Condition()
    process_producer = Producer(productList, condition)
    process_customer = Customer(productList, condition)
    process_producer.start()
    process_customer.start()
    process_producer.join()
    process_customer.join()

if __name__ == '__main__':
    main()
```

运行的部分结果是：

```plain
条件锁：被 生产者 获取
生产者：产生了 47。
生产者：唤醒消费者线程
条件锁：被 生产者 释放
条件锁：被 消费者 获取
消费者：消费了 47
条件锁：被 消费者 释放
条件锁：被 消费者 获取
消费者：等待生产者
条件锁：被 生产者 获取
生产者：产生了 100。
生产者：唤醒消费者线程
条件锁：被 生产者 释放
消费者：消费了 100
条件锁：被 消费者 释放
条件锁：被 消费者 获取
消费者：等待生产者
条件锁：被 生产者 获取
生产者：产生了 95。
生产者：唤醒消费者线程
条件锁：被 生产者 释放
消费者：消费了 95
条件锁：被 消费者 释放
条件锁：被 消费者 获取
消费者：等待生产者
```

### 信号量（Semaphore）

## 共享变量

### multiprocess 包内置类型

### 通过 Manager 创建共享变量

### 通过 Manager 管理

# 进程间通信

## 通过事件（Event）通信

## 通过队列（Queue）通信

## 通过管道（Pipe）通信

# 其他

## tqdm 多进度条

## Windows 上 Lock 对象的异常

[multiprocess-efficiency]:https://segmentfault.com/a/1190000007495352
[join-explain]:https://www.cnblogs.com/lipijin/p/3709903.html
[python多进程-cnblogs]:http://www.cnblogs.com/kaituorensheng/p/4445418.html
[Python中Lock与RLock]:https://blog.csdn.net/cnmilan/article/details/8849895
[使用Lock互斥锁]:https://www.jb51.net/article/63508.htm
[Python线程同步机制]:https://yoyzhou.github.io/blog/2013/02/28/python-threads-synchronization-locks/