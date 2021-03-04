# Python Multithreading

## 前言

**基本学习目标**

- 学习Python多线程基本语法
- 巩固Python编程知识
- 实际设计：设计多线程收发通信程序

## 学习笔记

### theading核心函数大致用法

#### 添加线程：

代码如下：

```python
import threading

def thread_job():
    #查看运行当前程序的线程
    print('This is a thread of %s \n'%threading.current_thread())   # \反斜杠，/正斜杠
    #查看当前线程数
    print(threading.active_count())

def main():
    t1 = threading.Thread(target = thread_job)
    t1.start()
    #print(threading.active_count())  这行代码不能放在这，因为主线程和t1线程不知道谁执行的快，所以有时输出1有时输出2

if __name__ == '__main__':
    main()
    
 #output:
	This is a thread of <Thread(Thread-1, started 1028)> 
	2
```

#### join函数：

- 功能描述：控制线程执行
- 用法描述：在线程1中对线程2使用join方法，线程1会停在此处等待线程2执行完成之后再执行；
- 代码分析：

```python
import threading
import time

def t1_job():
    print('t1 start\n')
    for i in range(10):
        time.sleep(0.1)
    print('t1 finish\n')

def t2_job():
    print('t2 start\n')
    print('t2 finish\n')

def main():
    t1 = threading.Thread(target=t1_job,name='t1')   #这里如果函数加（），程序会先执行一遍函数
    t2 = threading.Thread(target=t2_job,name='t2')
    t1.start()
    t2.start()
    t1.join()
    t2.join()

    print('all done\n')

if __name__ == '__main__':
    main()
```

#### Queue功能：

- 功能描述：多线程的运算结果是不能有返回值的，queue模块实现了各种【多生产者-多消费者】队列，可用于在执行的多个线程之间安全的交换信息。
- 用法描述：把各个线程的运算结果，放进一个常用的队列之中，再到主线程拿出来。
- 代码分析：

```python
import threading
import time
from queue import Queue

def job(l,q):
    for i in range(len(l)):
        l[i] = l[i]**2

    q.put(l)   #put是Queue的方法

def multthreading():
    q = Queue()
    threads = []
    data = [[1,2,3],[3,4,5],[2,2,2],[5,5,5]]
    for i in range(4):
        t = threading.Thread(target=job,args=[data[i],q])
        t.start()
        threads.append(t)
    for thread in threads:
        thread.join()
    result = []
    for _ in range(4):
        result.append(q.get())
    print(result)

if __name__ == '__main__':
    multthreading()
```

#### GIL机制：

- 问题描述：GIL（Global Interpreter Lock），一个时间只有一个线程在运算；

  *In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple native threads from executing Python bytecodes at once.*
   *This lock is necessary mainly because CPython’s memory management is not thread-safe.*
   (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)

  ![20210304095726](C:\Users\lenovo\Desktop\Python多线程\20210304095726.png)

- 代码分析：

```python
import threading
from queue import Queue
import copy
import time

def job(l, q):
    res = sum(l)
    q.put(res)

def multithreading(l):
    q = Queue()
    threads = []
    for i in range(4):
        t = threading.Thread(target=job, args=(copy.copy(l), q), name='T%i' % i)
        t.start()
        threads.append(t)
    [t.join() for t in threads]
    total = 0
    for _ in range(4):
        total += q.get()
    print(total)

def normal(l):
    total = sum(l)
    print(total)

if __name__ == '__main__':
    l = list(range(1000000))
    s_t = time.time()
    normal(l*4)
    print('normal: ',time.time()-s_t)
    s_t = time.time()
    multithreading(l)
    print('multithreading: ', time.time()-s_t)
    
```

- 结果分析：

  输出：normal:  0.15059614181518555   multithreading:  0.14162278175354004

  分析：为什么节约了一点点时间？扣除了读写(IO)的时间，

  单线程下有 IO操作会进行 IO等待，造成不必要的时间浪费，而开启多线程能在 线程 A等待时，自动切换到线程 B，可以不浪费 CPU的资源，从而能提升程序执行效率 ；

   python中想要某个线程要执行必须先拿到 GIL这把锁，且 python只有一个 GIL，拿到这个 GIL才能进入 CPU执行， 在遇到 I/O 操作时会释放这把锁。如果是纯计算的程序，没有 I/O 操作，解释器会每隔 100次操作就释放这把锁，让别的线程有机会 执行（这个次数可以通sys.setcheckinterval来调整）。所以虽然 CPython 的线程库直接封装操作系统的原生线程，但 CPython 进程做为一个整体，同一时间只会有一个获得了 GIL 的线程在跑，其它的线程都处于等待状态等着 GIL 的释放。 

  总结：进行 IO密集型的时候可以进行分时切换 所有这个时候多线程快过单线程

#### lock用法：

- 功能描述：对于多个线程同时需要访问一个变量，lock可以控制锁住变量，（线程）不互相干扰

- 用法描述：mutex = threading.Lock() # 创建锁mutex.acquire([timeout]) # 锁定mutex.release() # 释放

- 代码分析：

  ```python
  import threading
  import time
  
  def job1():
      time.sleep(5)
      global A,lock
      lock.acquire()
  
      for i in range(10):
          A += 1
          print('job',A)
  
      lock.release()
  
  def job2():
      global A,lock
      lock.acquire()
      for i in range(10):
          A += 10
          print('job2',A)
      lock.release()
  
  if __name__ == '__main__':
      lock = threading.Lock()
      A = 0
      t1 = threading.Thread(target=job1())
      t2 = threading.Thread(target=job2())
      t1.start()
      t2.start()
      t1.join()
      t2.join()
  ```

结果分析：

- 问题1：这里有一个问题，lock的作用还是说锁住了这个变量的相关代码还是锁住了整个调用代码的线程？
- 分析：lock锁住了整个调用变量的线程，线程start顺序决定lock顺序
- 问题2：有了GIL还需要lock吗？
- 分析：lock让线程程序更加顺序的执行；
- 问题3：lock的本质是join吗
- 分析：不是；

### 其他总结

并发编程分为多线程和多进程；Python的多线程是并发而不是并行（多进程是并行）；并发和并行从宏观上来讲都是同时处理多路请求的概念； 但并发和并行又有区别，并行是指两个或者多个事件在同一时刻发生；而并发是指两个或多个事件在同一时间间隔内发生。

## 其他思考

问题1：理论上来说，python程序可以虚拟出任意数量的线程，但如何选择最优线程数呢？

分析：这里多线程并不能加速运算过程，所以无最优线程数一说；这个问题可以留给多进程；

问题2：当代码中有多个线程时，需要多个join，这时候依次加入各个函数的join，这时候主线程是否会卡在第一个join？

分析：是这样的，但是这并不妨碍多个线程运行。

问题3：线程阻塞的概念

分析：阻塞线程的情况下，程序会先等待线程任务执行完，再往下执行其他代码；其实就是join；

## 学习总结

- 函数，对象，模块的区别要记住；python中Object对象是大写的；函数或者都是小写的；

- Pycharm中快捷键“ctrl+/”注释选中的代码；
- 进程是从 main() 方法开始的；

- 在创建线程中，输入函数名和输入函数名()的区别，输入函数名()就相当于执行完次函数再运行;

## 参考资料

### 视频资料

1、【莫烦Python】Threading 学会多线程 Python ：https://www.bilibili.com/video/BV1jW411Y7Wj

- 莫烦Python github https://github.com/MorvanZhou/tutorials
- 多线程部分github：https://github.com/MorvanZhou/tutorials/tree/master/threadingTUT

2、【黑马程序员】2小时玩转python多线程编程 ：https://www.bilibili.com/video/BV1fz4y1D7tU 

### 文档资料

1、https://www.runoob.com/python3/python3-multithreading.html

### 官方资料

1、https://docs.python.org/3/library/threading.html