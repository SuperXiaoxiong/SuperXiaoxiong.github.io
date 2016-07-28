---
layout: post
title:  "多线程-python核心编程笔记"
date:   2016-07-27 11:39:24
categories: python
tags:  python threading thread 
---

* content
{:toc}

## 简述

python代码执行由python虚拟机(解释器主循环)来控制，python虚拟机使用全局解释器锁(GIL)来控制，锁保证同一时刻只有一个线程在运行，对于 I/O密集型程序有很大优势。

python中有Thread和Threading等模块支持线程，Thread模块比较偏底层，本文对于Threading模块进行讲解。




## Threading模块介绍

python的threading模块是对thread进行了二次封装，提供了更加便捷的API。经常和Queue结合使用,Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步

### Threading模块对象

```
Thread        表示一个线程的执行对象
Lock          锁原语
Rlock         可重入锁，使单线程可以再次获得已经获得的锁
Condition     条件变量，让一个线程等待
Event         通用条件变量，激活和让多个线程等待
Semaphore     为等待锁的线程提供一个等待室的结构
```

### Thread类

Threading的Thread类是主要的运行对象

```
class threading.Thread(group=None, target=None, name=None, args=(), kwargs={})
应该始终以关键字参数调用该构造函数。参数有：
group应该为None；被保留用于未来实现了ThreadGroup类时的扩展。
target是将被run()方法调用的可调用对象。默认为None，表示不调用任何东西。
name是线程的名字。默认情况下，以“Thread-N”的形式构造一个唯一的名字，N是一个小的十进制整数。
args是给调用目标的参数元组。默认为()。
kwargs是给调用目标的关键字参数的一个字典。默认为{}。
如果子类覆盖该构造函数，它必须保证在对线程做任何事之前调用基类的构造函数(Thread.__init__())
```

#### Thread类运行函数

```
start()             开始线程的执行,在相同的线程对象上调用该方法多次将引发一个RuntimeError
run()               定义线程的功能的函数(一般子类重写)
join(timeout=None)  程序挂起，直到线程结束，time不为None，表示最多挂起时间
getName()           返回线程的名字
setName(name)       设置线程的名字
isAlive()           布尔标志，表示某个线程是否在运行
isDaemon()          返回线程的daemon标志
setDaemon(daemonic) 设置Daemon位为daemonic，在start运行前调用

name                线程表示, 没有任何语义
ident               线程的ID，如果线程还未启动则为None。它是一个非零的整数
```

守护线程：主线程退出，但子线程不会被强行退出(尤其是子线程还在活动时)，守护线程一般是一个等待客户请求的服务器，如果没有客户提出退出，就不会退出

在线程```start```之前，设置```daemon```属性```thread.setDaemon(True)```，就表示主线程要退出时，不用等待子线程结束，也不会强制结束子线程。默认```Daemo```n属性是false，也可以显示调用```thread.setDaemon(False)```

#### 创建线程

* 创建一个Thread实例，传给它一个函数
* 从Thread派生出一个子类，创建这个子类的实例

推荐最后一种

#### 传入函数

创建一个Thread实例，传入函数

创建两个线程，线程0运行4秒（sleep），线程1运行2秒。loop函数是执行的函数体，通过```threading.Thread(target = loop, args = (i, loops[i]))```,在Thread实例中，传入loop函数。

```
import threading
from time import ctime, sleep

loops = [4, 2]

def loop(nloop, nsec):
    print 'start loop', nloop, 'at:', ctime()
    sleep(nsec)
    print 'loop' , nloop, 'done at:', ctime()

def main():
    print 'start at:', ctime()
    threads = []
    nloops = range(len(loops))

    for i in nloops:
        t = threading.Thread(target = loop, args = (i, loops[i]))
        threads.append(t)

    for i in nloops:
        threads[i].start()

    for i in nloops:
        threads[i].join()

    print 'all Done at', ctime()


if __name__ == '__main__':
        main()
```

所有的线程都创建了之后，再一起调用```start()```函数启动。

 ```join()```会等到线程结束，或者在给了```timeout```参数的时候，等到超时为止。此实例为主线程调用了创建的线程0,1的```join()```方法，所以需要等待线程0,1一直退出。

```
start at: Thu Jul 28 09:31:34 2016
start loop 0 at: Thu Jul 28 09:31:34 2016
start loop 1 at: Thu Jul 28 09:31:34 2016
loop 1 done at: Thu Jul 28 09:31:36 2016
loop 0 done at: Thu Jul 28 09:31:38 2016
all Done at Thu Jul 28 09:31:38 2016
```

运行结果符合预期

#### 从Thread派生

从Thread派生出一个子类，创建这个子类的实例

这是目前比较常用的方法，创建一个新的class，把线程执行的代码放到class里，只覆盖该类的```__init__()```和```run()```方法

```
import threading
from time import sleep, ctime

loops = (4, 2)

class MyThread(threading.Thread):
    def __init__(self, func, args, name = ''):
        threading.Thread.__init__(self)    #调用父类的构造函数 
		#super(MyThread, self).__init__()   面向对象一讲中调用父类的构造函数的另外方法
        self.name = name
        self.func = func
        self.args = args

    def run(self):
        apply(self.func, self.args)


def loop(nloop, nsec):
    print 'start loop', nloop, 'at:', ctime()
    sleep(nsec)
    print 'loop' , nloop, 'done at:', ctime()

def main():
    print 'start at:', ctime()
    threads = []
    nloops = range(len(loops))

    for i in nloops:
        t = MyThread(loop, (i, loops[i]), loop.__name__)
        threads.append(t)

    for i in nloops:
        threads[i].start()

    for i in nloops:
        threads[i].join()

    print 'all Done at', ctime()


if __name__ == '__main__':
        main()
```

创建了MyThread类从threading.Thread继承

重写父类run方法，在线程启动后执行该方法内的代码。

 ```apply(func [, args [, kwargs ]]) ```函数用于当函数参数已经存在于一个元组或字典中时，间接地调用函数。args是一个包含将要提供给函数的按位置传递的参数的元组。如果省略了args，任何参数都不会被传递，kwargs是一个包含关键字参数的字典。
 
 ```apply()```的返回值就是```func()```的返回值，```apply()```的元素参数是有序的，元素的顺序必须和```func()```形式参数的顺序一致

### Lock对象

锁是同步原语，并不归某个特定线程所有。最底层的同步原语。

锁有两种状态locked或者unlocked，创建时处于unlocked状态。

锁有两个基本方法```acquire()```和```release()```。

1. 当状态是unlocked时，```acquire()```改变该状态为locked并立即返回。
2. 当状态是locked时，```acquire()```将阻塞直至在另外一个线程中调用```release()```来将它变为unlocked，然后```acquire()```调用将它重置为locked并返回
3.  ```release()```方法应该只在locked状态下调用；它改变状态为unlocked并立即返回。
4. 如果尝试释放一个unlocked的锁，将引发一个ThreadError。

使用实例
	
```
import threading
lock = threading.Lock()	#Lock对象
lock.acquire()
lock.release()
```

加锁

```
Lock.acquire([blocking])
```

1. blocking参数设置为True（默认值).将阻塞直至锁变成unlock状态，直到他的状态为locked并返回True
2. 如果设置为False，当阻塞存在是直接返回False，没有阻塞时locked并返回True

在已经加了一把锁的情况下

```
lock.acquire(True)  
# 等待，要等待lock被释放掉，如果在同一线程就会生成死锁

lock.acquire(False) 
# 直接返回False
```

释放一把锁。

```
Lock.release()
```

当锁是locked时，重置它为unlocked，然后返回。如果存在其他阻塞的线程正在等待锁变成unblocked状态，只会允许它们中的一个继续。

在一把没有锁住的锁上调用时，引发一个ThreadError 。

没有返回值

### Rlock对象

```
>>> import threading
>>> rLock = threading.RLock()
>>> rLock.acquire()
True
>>> rLock.acquire()
1
>>> rLock.acquire()
1
>>>
```

这两种琐的主要区别是：RLock允许在同一线程中被多次acquire。而Lock却不允许这种情况。注意：如果使用RLock，那么acquire和release必须成对出现，即调用了n次acquire，必须调用n次的release才能真正释放所占用的琐。

### Condition

Condiftion被称为条件变量，它提供了比Lock, RLock更高级的功能，允许我们能够控制复杂的线程同步问题。

线程首先acquire一个条件变量，然后判断一些条件。如果条件不满足则wait；如果条件满足，进行一些处理改变条件后，通过notify方法通知其他线程，其他处于wait状态的线程接到通知后会重新判断条件。不断的重复这一过程，从而解决复杂的同步问题。

可以认为Condition对象维护了一个锁（Lock/RLock)和一个waiting池。线程通过acquire获得Condition对象，当调用wait方法时，线程会释放Condition内部的锁并进入blocked状态，同时在waiting池中记录这个线程。当调用notify方法时，Condition对象会从waiting池中挑选一个线程，通知其调用acquire方法尝试取到锁。

Condition对象的构造函数可以接受一个Lock/RLock对象作为参数，如果没有指定，则Condition对象会在内部自行创建一个RLock。

锁被占用时使用(否则将会报RuntimeError异常)

```
Condition.wait([timeout]):
```

wait方法释放内部所占用的琐，同时线程被挂起，直至接收到通知被唤醒或超时（如果提供了timeout参数的话）。当线程被唤醒并重新占有琐的时候，程序才会继续执行下去。


```
Condition.notify():
```

唤醒一个挂起的线程（如果存在挂起的线程）。注意：notify()方法不会释放所占用的琐。

```
Condition.notifyAll()
```

唤醒所有挂起的线程（如果存在挂起的线程）。可以通知waiting池中的所有线程尝试acquire内部锁。由于上述机制，处于waiting状态的线程只能通过notify方法唤醒，所以notifyAll的作用在于防止有线程永远处于沉默状态。

演示条件变量同步的经典问题是生产者与消费者问题：假设有一群生产者(Producer)和一群消费者（Consumer）通过一个市场来交互产品。生产者的"策略"是如果市场上剩余的产品少于1000个，那么就生产100个产品放到市场上；而消费者的"策略"是如果市场上剩余产品的数量多余100个，那么就消费3个产品。用Condition解决生产者与消费者问题的代码如下

```
import threading
import time

class Producer(threading.Thread):
    def run(self):
        global count
        while True:
            if con.acquire():
                if count > 1000:
                    con.wait()
                else:
                    count = count+100
                    msg = self.name+' produce 100, count=' + str(count)
                    print msg
                    con.notify()
                con.release()
                time.sleep(1)

class Consumer(threading.Thread):
    def run(self):
        global count
        while True:
            if con.acquire():
                if count < 100:
                    con.wait()
                else:
                    count = count-3
                    msg = self.name+' consume 3, count='+str(count)
                    print msg
                    con.notify()
                con.release()
                time.sleep(1)

count = 500
con = threading.Condition()

def test():
    for i in range(2):
        p = Producer()
        p.start()
    for i in range(5):
        c = Consumer()
        c.start()
if __name__ == '__main__':
    test()
```

### Semaphore/BoundedSemaphore

Semaphore（信号量），同步指令。Semaphore管理一个内置的计数器，每当调用acquire()时-1，调用release() 时+1。计数器不能小于0；当计数器为0时，acquire()将阻塞线程至同步锁定状态，直到其他线程调用release()。

基于这个特点，Semaphore经常用来同步一些有“访客上限”的对象，比如连接池。

BoundedSemaphore 与Semaphore的唯一区别在于前者将在调用release()时检查计数器的值是否超过了计数器的初始值，如果超过了将抛出一个异常。

构造方法： 

```
Semaphore(value=1)
```

value是计数器的初始值。

实例方法：
 
```
acquire([timeout])
```

请求Semaphore。如果计数器为0，将阻塞线程至同步阻塞状态；否则将计数器-1并立即返回。 

```
release()
```

释放Semaphore，将计数器+1，如果使用BoundedSemaphore，还将进行释放次数检查。release()方法不检查线程是否已获得 Semaphore。

```
# encoding: UTF-8
import threading
import time
 
# 计数器初值为2
semaphore = threading.Semaphore(2)
 
def func():
    
    # 请求Semaphore，成功后计数器-1；计数器为0时阻塞
    print '%s acquire semaphore...' % threading.currentThread().getName()
    if semaphore.acquire():
        
        print '%s get semaphore' % threading.currentThread().getName()
        time.sleep(4)
        
        # 释放Semaphore，计数器+1
        print '%s release semaphore' % threading.currentThread().getName()
        semaphore.release()
 
t1 = threading.Thread(target=func)
t2 = threading.Thread(target=func)
t3 = threading.Thread(target=func)
t4 = threading.Thread(target=func)
t1.start()
t2.start()
t3.start()
t4.start()
 
time.sleep(2)
 
# 没有获得semaphore的主线程也可以调用release
# 若使用BoundedSemaphore，t4释放semaphore时将抛出异常
print 'MainThread release semaphore without acquire'
semaphore.release()
```

传入func函数作为参数来创建线程t1,t2,t3,t4,再依次运行4个线程，线程函数func执行功能是先输出acquire semaphore，然后acquire获取semaphore信号量锁。初始量为2，所以线程对象1和线程对象2获取semaphore信号量，并且sleep(4)，t3,t4等待，在主线程sleep(2)以后，释放semaphore信号量(虽然主线程并没有acquire semaphore信号量)，所以2s后，t3获取semaphore运行，再2s后,t1释放semaphore, t4获取信号量，运行，最后释放锁时，主线程多释放一个，如果使用BoundedSemaphore，会抛出异常。

### Event

Event（事件）是最简单的线程通信机制之一：一个线程通知事件，其他线程等待事件。Event内置了一个初始为False的标志，当调用set()时设为True，调用clear()时重置为 False。wait()将阻塞线程至等待阻塞状态。

Event其实就是一个简化版的 Condition。Event没有锁，无法使线程进入同步阻塞状态。

构造方法： 

```
Event()
```

实例方法： 
 ```isSet()```: 当内置标志为True时返回True。 

 ```set()```: 将标志设为True，并通知所有处于等待阻塞状态的线程恢复运行状态。
 
 ```clear()```: 将标志设为False。
 
 ```wait([timeout])```: 如果标志为True将立即返回，否则阻塞线程至等待阻塞状态，等待其他线程调用set()。


```
# encoding: UTF-8
import threading
import time
 
event = threading.Event()
 
def func():
    # 等待事件，进入等待阻塞状态
    print '%s wait for event...' % threading.currentThread().getName()
    event.wait()
    
    # 收到事件后进入运行状态
    print '%s recv event.' % threading.currentThread().getName()
 
t1 = threading.Thread(target=func)
t2 = threading.Thread(target=func)
t1.start()
t2.start()
 
time.sleep(2)
 
# 发送事件通知
print 'MainThread set event.'
event.set()
```


### threading类的其他函数

#### Timer

```
class threading.Timer(interval, function, args=[], kwargs={})
```

创建一个timer，在interval秒过去之后，它将以参数args和关键字参数kwargs运行function 。

```
import threading

def hello():
    print "hello, world"

t = threading.Timer(3, hello)
t.start()    #    3秒钟之后执行hello函数。
```

这个类表示一个动作应该在一个特定的时间之后运行 — 也就是一个计时器。Timer是Thread的子类， 因此也可以使用函数创建自定义线程

Timers通过调用它们的start()方法作为线程启动。timer可以通过调用cancel()方法（在它的动作开始之前）停止。timer在执行它的动作之前等待的时间间隔可能与用户指定的时间间隔不完全相同。

#### 常用函数

 ```threading.currentThread()```: 返回当前的线程变量。 

 ```threading.enumerate()```: 返回一个包含正在运行的线程的list。正在运行指线程启动后、结束前，不包括启动前和终止后的线程。
 
 ```threading.activeCount()```: 返回正在运行的线程数量，与len(threading.enumerate())有相同的结果。


### ThreadLocal

一个ThreadLocal变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，互不干扰。ThreadLocal解决了参数在一个线程中各个函数之间互相传递的问题。

```
import threading

# 创建全局ThreadLocal对象:
local_school = threading.local()

def process_student():
    # 获取当前线程关联的student:
    std = local_school.student
    print('Hello, %s (in %s)' % (std, threading.current_thread().name))

def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()

t1 = threading.Thread(target= process_thread, args=('Alice',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('Bob',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
```

在多线程环境下，每个线程都有自己的数据。一个线程使用自己的局部变量比使用全局变量好，因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。

全局变量local_school就是一个ThreadLocal对象，每个Thread对它都可以读写student属性，但互不影响。你可以把local_school看成全局变量，但每个属性如local_school.student都是线程的局部变量，可以任意读写而互不干扰，也不用管理锁的问题，ThreadLocal内部会处理。

ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

## 线程优先级队列（Queue）

Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语，能够在多线程中直接使用。可以使用队列来实现线程间的同步。例子是confition条件变量中的消费者生产者的例子

```
#encoding=utf-8
import threading
import time
from Queue import Queue

class Producer(threading.Thread):
    def run(self):
        global queue
        count = 0
        while True:
            for i in range(100):
                if queue.qsize() > 1000:
                     pass
                else:
                     count = count +1
                     msg = '生成产品'+str(count)
                     queue.put(msg)
                     print msg
            time.sleep(1)

class Consumer(threading.Thread):
    def run(self):
        global queue
        while True:
            for i in range(3):
                if queue.qsize() < 100:
                    pass
                else:
                    msg = self.name + '消费了 '+queue.get()
                    print msg
            time.sleep(1)

queue = Queue()


def test():
    for i in range(500):
        queue.put('初始产品'+str(i))
    for i in range(2):
        p = Producer()
        p.start()
    for i in range(5):
        c = Consumer()
        c.start()
if __name__ == '__main__':
    test()
```

## 参考
python核心编程第二版
* [Python爬虫(四)--多线程](http://www.jianshu.com/p/86b8e78c418a)
* [python2.78中文官方文档多线程](http://python.usyiyi.cn/python_278/library/threading.html#thread-objects)
* [python线程AstralWind博客](http://www.cnblogs.com/huxi/archive/2010/06/26/1765808.html)
* [python多线程编程(5): 条件变量同步](http://www.cnblogs.com/holbrook/archive/2012/03/13/2394811.html)
* [Python模块学习：threading 多线程控制和处理](http://python.jobbole.com/81546/)
* [w3school-python多线程](https://wizardforcel.gitbooks.io/w3school-python/content/28.html)
* [廖雪峰python学习](http://www.liaoxuefeng.com/wiki/001374738125095c955c1e6d8bb493182103fac9270762a000/001386832360548a6491f20c62d427287739fcfa5d5be1f000)