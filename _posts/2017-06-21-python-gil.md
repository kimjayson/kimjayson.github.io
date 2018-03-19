---
layout: post
title: 一篇文章理解Python里GIL
categories: Python
description: python GIL全局解释锁。
keywords: 基础, Python, GIL
---
刚接触Python时候，经常听到GIL，经常有人说python下多线程是鸡肋，一直以来都人云亦云这么认为，最近整理技术笔记，深入研究了下GIL。

### GIL的概念

GIL全称是`Global Interpeter Lock`（全局解释器锁）。每个CPU在同一时间只能执行一个线程（在单核CPU下的多线程其实都只是并发，不是并行，并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。）

需要明确一点的是，GIL并不是Python的特性，是CPython实现引入的概念，而CPython解释器是目前应用最广的Python解释器，另外PyPy，IronPython也依赖GIL，所以很多人想当然的把GIL归为Python的缺陷，其实Python可以不依赖GIL。

其他语言比如ruby也是有GIL的。


### GIL PYTHON虚拟机执行方式
多线程下，每个线程的执行方式：
1. 获取GIL
2. 切换到一个线程去运行
3. 运行：
    a. 指定数量的字节码指令,虚拟机将其挂起
    b. 线程主动让出控制（可以调用time.sleep(0)）
4. 把线程设置为睡眠状态
5. 解锁GIL
6. 再次重复以上所有步骤

python2.x里，GIL释放逻辑是：
> 当前线程遇见IO操作或者ticks计数达到100（ticks可以看作是python自身的一个计数器，专门做用于GIL，每次释放后归零，这个计数可以通过 sys.setcheckinterval 来调整），进行释放

而在python3.x中，GIL释放逻辑是：
> 不使用ticks计数，改为使用计时器（执行时间达到阈值后，当前线程释放GIL），这样对CPU密集型程序更加友好，但依然没有解决GIL导致的同一时间只能执行一个线程的问题，所以效率依然不尽如人意。


### 原因

为了利用多核，Python开始支持多线程。而解决多线程之间数据完整性和状态同步的最简单方法自然就是加锁。 于是有了GIL这把超级大锁，而当越来越多的代码库开发者接受了这种设定后，他们开始大量依赖这种特性（即默认python内部对象是thread-safe的，无需在实现时考虑额外的内存锁和同步操作）。
很多C扩展库都在必要的时候释放GIL以提高线程的并发性，尽管非常有限，但是用来解决阻塞IO并发肯定是没问题牵扯到从底层重新设置cPython的核心，包括资源同步、gc策略等


### GIL工作原理
#### 协同多任务处理：

本质上两个线程各自分别连接一个套接字：
```
def do_connect():
    s = socket.socket()
    s.connect(('python.org', 80))  # drop the GIL
 
for i in range(2):
    t = threading.Thread(target=do_connect)
    t.start()
```
在socket连接时，如何放弃GIL
```
/* s.connect((host, port)) method */
static PyObject *
sock_connect(PySocketSockObject *s, PyObject *addro)
{
    sock_addr_t addrbuf;
    int addrlen;
    int res;
 
    /* convert (host, port) tuple to C address */
    getsockaddrarg(s, addro, SAS2SA(&addrbuf), &addrlen);
 
    **Py_BEGIN_ALLOW_THREADS**
    res = connect(s->sock_fd, addr, addrlen);
    **Py_END_ALLOW_THREADS**
 
    /* error handling and so on .... */
}
```
当然 Py_END_ALLOW_THREADS 重新获取锁。一个线程可能会在这个位置堵塞，等待另一个线程释放锁；一旦这种情况发生，等待的线程会抢夺回锁，并恢复执行你的Python代码。简而言之：当N个线程在网络 I/O 堵塞，或等待重新获取GIL，而一个线程运行Python。

#### 抢占式多任务处理
python程序运行过程，首先，Python文本被编译成一个名为字节码的简单二进制格式。第二，Python解释器的主回路，一个名叫 pyeval_evalframeex() 的函数，流畅地读取字节码，逐个执行其中的指令。

当解释器通过字节码时，它会定期放弃GIL，而不需要经过正在执行代码的线程允许，这样其他线程便能运行：
```
for (;;) {
    if (--ticker < 0) {
        ticker = check_interval;
 
        /* Give another thread a chance */
        PyThread_release_lock(interpreter_lock);
 
        /* Other threads may run now */
 
        PyThread_acquire_lock(interpreter_lock, 1);
    }
 
    bytecode = *next_instr++;
    switch (bytecode) {
        /* execute the next instruction ... */ 
    }
}
```
默认情况下，检测间隔是1000 字节码。所有线程都运行相同的代码，并以相同的方式定期从他们的锁中抽出。在 Python 3 GIL 的实施更加复杂，检测间隔不是一个固定数目的字节码，而是15 毫秒。


### 多线程,多进程在GIL下表现
不论标准的，还是第三方的扩展模块，都被设计成在进行密集计算任务是，释放GIL。
还有，就是在做I/O操作时，GIL总是会被释放。对所有面向I/O 的(会调用内建的操作系统C 代码的)程序来说，GIL 会在这个I/O 调用之前被释放，以允许其它的线程在这个线程等待I/O 的时候运行。如果是纯计算的程序，没有 I/O 操作，解释器会每隔 100 次操作就释放这把锁，让别的线程有机会执行（这个次数可以通过 sys.setcheckinterval 来调整）如果某线程并未使用很多I/O 操作，它会在自己的时间片内一直占用处理器（和GIL）。也就是说，I/O 密集型的Python 程序比计算密集型的程序更能充分利用多线程环境的好处。


1、CPU密集型代码(各种循环处理、计数等等)，在这种情况下，ticks计数很快就会达到阈值，然后触发GIL的释放与再竞争（多个线程来回切换当然是需要消耗资源的），所以python下的多线程对CPU密集型代码并不友好。

2、IO密集型代码(文件处理、网络爬虫等)，多线程能够有效提升效率(单线程下有IO操作会进行IO等待，造成不必要的时间浪费，而开启多线程能在线程A等待时，自动切换到线程B，可以不浪费CPU的资源，从而能提升程序执行效率)。所以python的多线程对IO密集型代码比较友好。

3.多核情况下，线程唤醒获得GIL，但从release GIL到acquire GIL之间几乎是没有间隙的，所以其他线程被唤醒，大部分主线程又再一次拿到GIL，这样子线程白浪费CPU时间，看着主线程执行，然后达到切换时间进入就绪状态，恶性循环。


### 使用python多线程
如果一个线程可以随时失去 GIL，你必须使让代码线程安全。 然而 Python 程序员对线程安全的看法大不同于 C 或者 Java 程序员，因为许多 Python 操作是原子的。

在列表中调用 sort()，就是原子操作的例子。线程不能在排序期间被打断，其他线程从来看不到列表排序的部分，也不会在列表排序之前看到过期的数据。原子操作简化了我们的生活，但也有意外。例如，+ = 似乎比 sort() 函数简单，但+ =不是原子操作。

代码的一行中， n += 1，被编译成 4 个字节码，进行 4 个基本操作：

- 将 n 值加载到堆栈上
- 将常数 1 加载到堆栈上
- 将堆栈顶部的两个值相加
- 将总和存储回 n

>记住，一个线程每运行 1000 字节码，就会被解释器打断夺走 GIL 。如果运气不好，这（打断）可能发生在线程加载 n 值到堆栈期间，以及把它存储回 n 期间。

lst.sort()一行被编译成 3 个字节码：

- 将 lst 值加载到堆栈上
- 将其排序方法加载到堆栈上
- 调用排序方法

>即使这一行  lst.sort() 分几个步骤，调用 sort 自身是单个字节码，因此线程没有机会在调用期间抓取 GIL 。我们可以总结为在 sort() 不需要加锁。

为了避免担心哪个操作是原子的，遵循一个简单的原则：始终围绕共享可变状态的读取和写入加锁。


### 如何避免GIL的负面影响
用multiprocess代替thread，但是会引入线程数据同步和通信的困难。thread来讲，利用thread.lock的context包裹就可以；
而multiprocess需要用Queue，put再get或者share memory的方法。

### pypy对gil的改进
f函数不是线程安全的：
```
def f ( list1 ,   list2 ) :
    x   =   list1. pop ( )
    list2. append ( x )
```

Jpython的方式：所有 mutable 内置类型加锁
```
def f ( list1 ,   list2 ) :
    acquire_all_locks ( list1. lock ,   list2. lock )
    x   =   list1. pop ( )
    list2. append ( x )
    release_all_locks ( list1. lock ,   list2. lock )
```
Cpython的方式：GIL
```
def f ( list1 ,   list2 ) :
    global_lock. acquire ( )
    x   =   list1. pop ( )
    list2. append ( x )
    global_lock. release ( )
```

pypy未来准备采用的模型：STM
```
def f ( list1 ,   list2 ) :
      while   True :
        t   =   transaction ( )
        x   =   list1. pop ( t )
        list2. append ( t ,   x )
          if   t. commit ( ) :
              break
```
创建一个局部线程事务，存储它到一个可以进行读写操作的地方，这和GIL类似，代码不会互相干扰，能够真正运行多线程。

如果Cpython这样改，所有的函数都将改变，list.pop(t)传递事务，必需用事务记录改变。这个想法在2013年就开始，现在我们也并没有看到改变...

---
写在最后，理解一个语言特性，并在此基础上合理利用，才是学习的正确态度。


