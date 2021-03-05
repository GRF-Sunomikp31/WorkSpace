# Python Multiprocessing

## 前言

**基本学习目标**

- 学习Python Multiprocessing 基本语法
- 分析**对比**多进程和多线程，以及二者适用场景
- 使用多进程编写serial通信代码

## 学习笔记

### Multiprocessing核心函数大致用法

#### 创建进程：

- 用法分析：python创建进程和创建线程的方法基本一致（python简化了很多重复的东西）；

- 代码分析：

  ```python
  import multiprocessing
  
  
  def job():
      print('aaaa')
      # 查看当前线程数
  
  if __name__ == '__main__':
      p1 = multiprocessing.Process(target=job)
      p1.start()
      p1.join()
  ```


#### Queue进程输出：

- 功能分析：

- 用法分析

- 代码分析：

  ```python
  import multiprocessing
  
  def job(q):
      res = 0
      for i in range(1000):
          res += i+i**2+i**3
      q.put(res)  #queue
  
  if __name__ == '__main__':
      q = multiprocessing.Queue()
      p1 = multiprocessing.Process(target=job,args=(q,)) 
      #这里args只有一个值的时候必须(q,)这样写，说明args是可以迭代的，后续可以更新
      p2 = multiprocessing.Process(target=job,args=(q,))
      p1.start()
      p2.start()
      p1.join()
      p2.join()
      res1 = q.get()
      res2 = q.get()
      print(res1+res2)

  
  分析：Multiprocessing中已经嵌套了Queue

#### 多线程多进程效率对比：

- 功能分析：

- 用法分析

- 代码分析：

  ```python
  import multiprocessing as mp
  import threading as td
  import time
  
  def job(q):
      res = 0
      for i in range(1000000):
          res += i+i**2+i**3
      q.put(res) # queue
  
  def multicore():
      q = mp.Queue()
     
      p1 = mp.Process(target=job, args=(q,))   
      p2 = mp.Process(target=job, args=(q,))
      p1.start()
      p2.start()
      p1.join()
      p2.join()
      res1 = q.get()
      res2 = q.get()
      print('multicore:' , res1+res2)
  
  def normal():
      res = 0
      #’_’ 是一个循环标志，也可以用i，j 等其他字母代替
      for _ in range(2):
          for i in range(1000000):
              res += i+i**2+i**3
      print('normal:', res)
  
  def multithread():  
      q = mp.Queue()
      t1 = td.Thread(target=job, args=(q,))
      t2 = td.Thread(target=job, args=(q,))
      t1.start()
      t2.start()
      t1.join()
      t2.join()
      res1 = q.get()
      res2 = q.get()
      print('multithread:', res1+res2)
  
  if __name__ == '__main__':
      st = time.time()
      normal()
      st1= time.time()
      print('normal time:', st1 - st)
      multithread()
      st2 = time.time()
      print('multithread time:', st2 - st1)
      multicore()
    print('multicore time:', time.time()-st2)
  ```

  分析：多进程>多线程>normal
#### 进程池pool：

- 功能分析：进程池pool是，你可以把运算放进pool中，python会自动分配进程以及运算结果；

- 代码分析：

  ```python
  import multiprocessing as mp
  
  def job(x):
      return x*x
  
  def multicore():
      pool = mp.Pool(processes=2)
      res = pool.map(job, range(10))
      print(res)
      res = pool.apply_async(job, (2,))
      print(res.get())
      multi_res =[pool.apply_async(job, (i,)) for i in range(10)]
      print([res.get() for res in multi_res])
  
  if __name__ == '__main__':
      multicore()
  ```
![QQ截图20210305224724](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/3.png)

分析：进程池pool是一个计算结果的很好办法，他会选择最大进程数。

#### 共享内存shared memory：

- 功能分析：在多进程中，每一个进程都有自己的内存空间；此事使用全局变量是行不通的，要用到shared memory；
- 代码分析：

```python
import multiprocessing

#共享的值  d表示形式 小数，i代码整数...    1是传入的值
value = multiprocessing.Value('d',1)

#这是共享列表  这里Array只能是一维
array = multiprocessing.Array('i',[1,2,3])
```

![QQ截图20210305224724](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/2.png)

分析：只有shared memory才能实现进程间的交流


#### Lock：

- 功能分析：lock是在shared memory的基础上使用的；

- 代码分析：

  ```python
  import multiprocessing
  import time
  
  def job(v,num,l):
      l.acquire()
      for _ in range(10):
          time.sleep(0.1)
          v.value += num
          print(v.value)
      l.release()
  
  def multicore():
      l = multiprocessing.Lock()
      v = multiprocessing.Value('i',0)
      p1 = multiprocessing.Process(target=job,args=(v,1,l))
      p2 = multiprocessing.Process(target=job, args=(v, 3, l))
      p1.start()
      p2.start()
      p1.join()
      p2.join()
  
  if __name__ == '__main__':
      multicore()
  ```

### 其他总结

`print(multiprocessing.cpu_count())`  output：8  ，四核心八线程的电脑

## 其他思考

1、如何选择最优进程数？

分析：在任务量较大时，选择电脑核数的线程；四核心八线程（超线程技术）在多进程中就是8个cpu处理；使用pool选择最优进程。

2、多线程其实是一个时间运行一个线程，这样比如全双工通信，在python使用多线程处理，其实还是半双工的？

分析：其实是半双工（假全双工）。

3、如何查看当前进程数？

 Pid：Return the process ID，通过 os  module下的os.getpid()获取进程号。 

## 学习总结

- RGB三个通道相同就是灰色（0-255）
- 多进程不是能加速所有的计算，因为调用进程也是要耗费时间的，所以在计算量不大的时候使用多进程反而会耗时
- 高io吞吐多线程肯定有有优势，
- 多进程一定要在main下运行，格式要求
- 多核，每个核有单独的**运算空间**和运算能力

## 参考资料

### 视频资料

1、【莫烦Python】Multiprocessing 让你的多核计算机发挥真正潜力 Python：https://www.bilibili.com/video/BV1jW411Y7pv

​		教程代码：https://github.com/MorvanZhou/tutorials/tree/master/multiprocessingTUT

### 文档资料

#### 官方资料

1、https://docs.python.org/3/library/multiprocessing.html
