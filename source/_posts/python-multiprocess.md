---
title: Python 多进程编程
date: 2018-07-09 18:53:07
tags:
<<<<<<< HEAD
    - Python
categories: Python
---
Python 在处理大数据的时候，启用多进程是有效提高计算效率的手段。
Python 已经提供了非常好用的 multiprocess 包来支持多进程编程，
但是在多进程编程时仍然会遇到一些难以处理的问题，需要一些技巧来解决。
<!-- more -->
=======
---
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc
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
<<<<<<< HEAD
=======
        - [通过 Manager 管理](#%E9%80%9A%E8%BF%87-manager-%E7%AE%A1%E7%90%86)
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc
- [进程间通信](#%E8%BF%9B%E7%A8%8B%E9%97%B4%E9%80%9A%E4%BF%A1)
    - [通过事件（Event）通信](#%E9%80%9A%E8%BF%87%E4%BA%8B%E4%BB%B6%EF%BC%88event%EF%BC%89%E9%80%9A%E4%BF%A1)
    - [通过队列（Queue）通信](#%E9%80%9A%E8%BF%87%E9%98%9F%E5%88%97%EF%BC%88queue%EF%BC%89%E9%80%9A%E4%BF%A1)
    - [通过管道（Pipe）通信](#%E9%80%9A%E8%BF%87%E7%AE%A1%E9%81%93%EF%BC%88pipe%EF%BC%89%E9%80%9A%E4%BF%A1)
- [其他](#%E5%85%B6%E4%BB%96)
    - [tqdm 多进度条](#tqdm-%E5%A4%9A%E8%BF%9B%E5%BA%A6%E6%9D%A1)
<<<<<<< HEAD
    - [Windows 上 Lock 的问题](#windows-%E4%B8%8A-lock-%E7%9A%84%E9%97%AE%E9%A2%98)
=======
    - [Windows 上 Lock 对象的异常](#windows-%E4%B8%8A-lock-%E5%AF%B9%E8%B1%A1%E7%9A%84%E5%BC%82%E5%B8%B8)
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc

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
<<<<<<< HEAD
- `args`：进程调用函数时给函数传递的参数，为一个元组；
=======
- `args`：进程调用函数时给函数传递的参数，为一个元祖；
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc
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
<<<<<<< HEAD
    p.join()
=======
    p.joi()
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc
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

<<<<<<< HEAD
信号量是一个非负整数，所有通过它的进程都会将该整数减一，
当该整数值为零时，所有试图通过它的进程都将处于等待状态。

```python
from multiprocessing import Process, current_process, Semaphore
import time

def worker(s, i):
    s.acquire()
    print(current_process().name + "acquire");
    time.sleep(i)
    print(current_process().name + "release\n");
    s.release()

if __name__ == "__main__":
    s = Semaphore(2)
    for i in range(5):
        p = Process(target = worker, args=(s, i*2))
        p.start()
```

## 共享变量

在多进程中，是无法直接使用全局变量作为共享变量的，因为不同进程具有不同的内存空间。
但是，共享变量也是不能避免的。Python 中也提供了一些创建共享变量的方法。

- Multiprocess 包内置类型
- 通过 Manager 创建共享变量

### multiprocess 包内置类型

multiprocess 包提供了两种类型的共享变量：

- `Value(typecode_or_type, *args, lock=True)`：表示一个值类型变量。
- `Array(typecode_or_type, size_or_initializer, *, lock=True)`：表示一个数组。这种创建数组的方式能力比较有限，它不支持除了 C 数据类型以外的类型。

`typecode_or_type` 描述了元素的类型，可取值是：

| typecode | type            |
| -------- | --------------- |
| 'c'      | ctypes.c_char   |
| 'u'      | ctypes.c_wchar  |
| 'b'      | ctypes.c_byte   |
| 'B'      | ctypes.c_ubyte  |
| 'h'      | ctypes.c_short  |
| 'H'      | ctypes.c_ushort |
| 'i'      | ctypes.c_int    |
| 'I'      | ctypes.c_uint   |
| 'l'      | ctypes.c_long   |
| 'L'      | ctypes.c_ulong  |
| 'f'      | ctypes.c_float  |
| 'd'      | ctypes.c_doubl  |

创建后，只要将这些变量传递给子进程即可。

### 通过 Manager 创建共享变量

Manager() 返回的 manager 对象提供一个服务进程，使得其他进程可以通过代理的方式操作 Python 对象。
Manager 支持 list、dict 等多种数据类型。
（[多进程multiprocess][liujiang]）

把之前的共享变量的代码中，共享的变量由 list 改为 Manager 对象创建的 list，可以得到正确结果。

```python
from multiprocessing import Process, Lock, Manager
import time

def work(lock, var, index):
    with lock:
        var.append(index)
        print(f"Process {index} apped {index}")

if __name__ == '__main__':
    var = Manager().list()
    lock = Lock()
    process_list = [Process(target=work, args=(lock, var, i)) for i in range(8)]
    for p in process_list:
        p.start()
    for p in process_list:
        p.join()
    print(var)
```

# 进程间通信

进程间通信，可以起到共享变量的效果，也可以起到锁的效果。

进程间通信的方式有三种：

- 事件（Event）
- 队列（Queue）
- 管道（Pipe）

## 通过事件（Event）通信

Event 是同步通信的方式，有些类似于条件锁。由于是它是同步的，而且不能传递数据。
因此这里就不仔细研究 Event 的作用。

这个例子示例了主进程与子进程之间通过 Event 进行通信的方法。

```python
import multiprocessing
import time
def wait_for_event(e):
    print("wait_for_event: starting")
    e.wait()
    print("wairt_for_event: e.is_set()->" + str(e.is_set()))

def wait_for_event_timeout(e, t):
    print("wait_for_event_timeout:starting")
    e.wait(t)
    print("wait_for_event_timeout:e.is_set->" + str(e.is_set()))

if __name__ == "__main__":
    e = multiprocessing.Event()
    w1 = multiprocessing.Process(target=wait_for_event, args=(e,))
    w2 = multiprocessing.Process(target=wait_for_event_timeout, args=(e, 6))
    w1.start()
    w2.start()
    time.sleep(10)
    print("main: event setting")
    e.set()
    print("main: event is set")
```

## 通过队列（Queue）通信

Queue 是多进程安全的队列，可以使用 Queue 实现多进程之间的数据传递。
Queue 有两个方法：

- `put()`：将数据插入队列中。
- `get()`：从队列读取并且删除一个元素。

这两个方法都有两个参数：`blocked`, `timeout`，
控制队满和队空两种情况：

- `put`：当队满时，如果 `blocked=True` ，那么会阻塞 `timeout` 指定的时间，直到队列有空间。如果超时，或 `blocked=False` ，则抛出 `Queue.Full` 异常。
- `get`：当队满时，如果 `blocked=True` ，那么会阻塞 `timeout` 指定的时间，直到队列有元素。如果超时，或 `blocked=False` ，则抛出 `Queue.Empty` 异常。

调用实例：

```python
class FineTargetTaxiProcess(mp.Process):
    '''
    处理进程：多进程方式处理文件，结果全部传递给打印进程。
    '''
    def __init__(self, input_files, index, queue):
        mp.Process.__init__(self, target=self.pick, args=(queue,))
        self.input_files = input_files
        self.index = index
        self.lock = lock

    def pick(self, queue):
        for filename in tqdm(self.input_files, ncols=80, position=self.index, desc=f"Process {self.index}"):
            with open(filename, encoding="GB2312") as in_file:
                for row in in_file:
                    cells = row.split(",")
                    if int(cells[0]) == 11865:
                        try:
                            queue.put(",".join(cells), block=False)
                        except:
                            print("Queue full")

class PrinterProcess(mp.Process):
    '''
    打印进程：维持对输出文件的打开状态，打印数据。
    可避免频繁打开、关闭结果文件造成的系统开销，
    但是引入了消息传递的开销。
    '''
    def __init__(self, output_file, log, queue):
        mp.Process.__init__(self, target=self.write, args=(queue,))
        self.output_file = output_file
        self.log_file = log

    def write(self, queue):
        with open(self.output_file, mode="w", newline="\n") as printer, open(self.log_file, mode="w") as log:
            while True:
                try:
                    row = queue.get(block=True, timeout=1)
                    print(row, file=printer)
                except:
                    print("Queue empty", file=log)

if __name__ == '__main__':
    lock = mp.Lock()
    ROOT_DIR = r"/mnt/e/出租车点/201502/RawCSV"
    INPUT_FILES = [os.path.join(ROOT_DIR, f) for f in os.listdir(ROOT_DIR)]
    GROUP_LIST = distrib_works(INPUT_FILES, 6)
    QUEUE = mp.Queue()
    PROCESS_LIST = [FineTargetTaxiProcess(element, i, QUEUE) for i, element in enumerate(GROUP_LIST)]
    PRINTER_PROCESS = PrinterProcess("./data/usequeue.txt", "./data/usequeue.log", QUEUE)
    for process in PROCESS_LIST:
        process.daemon = True
        process.start()
    PRINTER_PROCESS.daemon = True
    PRINTER_PROCESS.start()
    for p in PROCESS_LIST:
        p.join()
```

## 通过管道（Pipe）通信

Pipe 是一个可以双向通信的对象，返回 `(conn1, conn2)`，
代表一个管道的两个端， `conn1` 只负责接受消息， `conn2` 只负责发送消息。
如果设置了 `duplex=True` ，那么这个管道是全双工模式，
`conn1` 和 `conn2` 均可收发。

```python
class FineTargetTaxiProcess(mp.Process):
    '''
    处理进程：多进程方式处理文件，结果全部传递给打印进程。
    '''
    def __init__(self, input_files, index, pipe):
        mp.Process.__init__(self, target=self.pick, args=(pipe,))
        self.input_files = input_files
        self.index = index
        self.lock = lock

    def pick(self, pipe):
        for filename in tqdm(self.input_files, ncols=80, position=self.index, desc=f"Process {self.index}"):
            with open(filename, encoding="GB2312") as in_file:
                for row in in_file:
                    cells = row.split(",")
                    if int(cells[0]) == 11865:
                        try:
                            pipe.send(",".join(cells))
                        except e as Exception:
                            print("Pipe send error")

class PrinterProcess(mp.Process):
    '''
    打印进程：维持对输出文件的打开状态，打印数据。
    可避免频繁打开、关闭结果文件造成的系统开销，
    但是引入了消息传递的开销。
    '''
    def __init__(self, output_file, log, pipe):
        mp.Process.__init__(self, target=self.write, args=(pipe,))
        self.output_file = output_file
        self.log_file = log

    def write(self, pipe):
        with open(self.output_file, mode="w", newline="\n") as printer, open(self.log_file, mode="w") as log:
            while True:
                try:
                    row = pipe.recv()
                    print(row, file=printer)
                except e as Exception:
                    print("Pipe read error", file=log)

if __name__ == '__main__':
    lock = mp.Lock()
    # ROOT_DIR = "../data/201502/temp"
    ROOT_DIR = r"/mnt/e/出租车点/201502/RawCSV"
    INPUT_FILES = [os.path.join(ROOT_DIR, f) for f in os.listdir(ROOT_DIR)]
    GROUP_LIST = distrib_works(INPUT_FILES, 6)
    (RECEIVER, SENDER) = mp.Pipe()
    PROCESS_LIST = [FineTargetTaxiProcess(element, i, SENDER) for i, element in enumerate(GROUP_LIST)]
    PRINTER_PROCESS = PrinterProcess("./data/usepipe.txt", "./data/usepipe.log", RECEIVER)
    for process in PROCESS_LIST:
        process.daemon = True
        process.start()
    PRINTER_PROCESS.daemon = True
    PRINTER_PROCESS.start()
    for p in PROCESS_LIST:
        p.join()
```

=======
## 共享变量

### multiprocess 包内置类型

### 通过 Manager 创建共享变量

### 通过 Manager 管理

# 进程间通信

## 通过事件（Event）通信

## 通过队列（Queue）通信

## 通过管道（Pipe）通信

>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc
# 其他

## tqdm 多进度条

<<<<<<< HEAD
是一个快速，可扩展的 Python 进度条，可以在 Python 长循环中添加一个进度提示信息，
用户只需要封装任意的迭代器 `tqdm(iterator)` 。

这里有一些参数：

- `ncols`：整个进度条（包括条以及其他文字）的宽度。最好设置一个小于控制台总宽的值。
- `mininterval`：进度条更新的最小间隔。默认为 0.1。
- `position`：进度条的位置，从0开始。对不同的 tqdm 对象设置不同的 position，可以在控制台的不同位置显示出来，适用于多进程与多线程。

> 由于 Windows 上多进程时 tqdm 无法获取默认的锁，所以会出现进度条错乱。在 Linux 上是没有问题的。

## Windows 上 Lock 的问题

其实每次传入子进程函数内部的 `Lock`，在各个进程中的 `id` 都不一样。在 Linux 下没有这个问题。
这往往会导致一些程序在 Windows 上不正确。
因此，在 Windows 上最好少用 `Lock`，多采用消息传递或共享变量的方式设计程序。
=======
## Windows 上 Lock 对象的异常
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc

[multiprocess-efficiency]:https://segmentfault.com/a/1190000007495352
[join-explain]:https://www.cnblogs.com/lipijin/p/3709903.html
[python多进程-cnblogs]:http://www.cnblogs.com/kaituorensheng/p/4445418.html
[Python中Lock与RLock]:https://blog.csdn.net/cnmilan/article/details/8849895
[使用Lock互斥锁]:https://www.jb51.net/article/63508.htm
<<<<<<< HEAD
[Python线程同步机制]:https://yoyzhou.github.io/blog/2013/02/28/python-threads-synchronization-locks/
[liujiang]:http://www.liujiangblog.com/course/python/82
=======
[Python线程同步机制]:https://yoyzhou.github.io/blog/2013/02/28/python-threads-synchronization-locks/
>>>>>>> dfffe1e9f0238337aec71fdc37267c233b38c5bc
