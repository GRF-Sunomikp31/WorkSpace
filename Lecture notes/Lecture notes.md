# Lecture notes

## 讲座内容

### 1 如何发表一篇顶会

**Time：**2021/2/8

**Video link：**https://www.bilibili.com/video/BV1Pi4y1F7VG

**Note content：**

![1](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Lecture%20notes/IMG/1.png)

入门一个领域：泛读几百篇文章，精读几十篇文章

**科研书籍**：像外行一样思考，像科学家一样实践

### 2  3D相机性能评估

**Time：**2021/2/9

**Video link：**https://www.bilibili.com/video/BV1VT4y1K7mj

**Note content：**

利用3D相机做物体姿态识别

3D相机分类

​    结构光：intel、mircrosoft（kinect）、apple、prbbec

​           缺陷：holes

​    双目：三角定位、leap、dji

​    TOF: mircrosoft

​    Lidar:光学雷达

​    Rader：传统电磁波雷达

深度相机的性能指标：利用一个标准的平台的估计、

物体姿态估计：神经网络

![2](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Lecture%20notes/IMG/2.png)

### 3 相机标定的基本原理与经验分享

**Time：**2021/2/10

**Video link：**https://www.bilibili.com/video/BV1R7411m7ZQ?t=7

**Note content：**

![3-1](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Lecture%20notes/IMG/3-1.png)

畸变这个多项式写到6次方就基本上够了，后面几乎为0，没有写的必要了

产生的畸变量是和这个点到图像中心的距离成正比的（中心点是唯一不变的坐标）

标定要标定什么：内参（包括畸变参数）、外参R T 旋转矩阵和平移矩阵

标定方法：

​    优化目标函数:要优化的参数很多，很容易陷入局部最优解中

​    张正友标定法： **首先标定板必须是一个平面，提出了一个非常好的算术解从而得出一个初值**

这个标定之后，如何检测标定的效果（误差大小，有人说他标定的误差只有0.07个像素点）

至少3张照片才能完成标定，但是多了的话可以冗余方程更加精确

张正友IEEE论文

Opencv中的标定函数和matlab标定函数的区别

​    Calibratecamera函数

标定靶：圆和棋盘，测试结果圆更好提取中心（精度和鲁棒性），检测圆是通过拟合的，所以精度更高，但是具有偏心误差（棋盘不存在） 有相应的算法论文解决这个问题（**精度非常高**，纠正偏心误差）-如何不打算解决偏心误差推荐使用棋盘格

![3-2](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Lecture%20notes/IMG/3-2.png)

![3-3](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Lecture%20notes/IMG/3-3.png)

照片数量：10-20张

标定靶不动，相机移动；相机三个姿态、，要让标动板图像出现在相机画面的**各个区域（边角处）** 

**判定标定的的结果**

1、 重投影误差（这个是否合理）

2、 其他方法

3、 实际项目效果

应用：

​    单目：PNP问题  opencv中函数solvepnp 最后求解的是外参3D-2D 

​    双目测量  2d-3d（求解的应该是一个射线单目

标定板带背光板（亮度一至，问商家标定靶的精度多少

自己做一个标定小工具 

棋盘格：长宽是两个质数

### 4 嵌入式开发漫漫之路——从小白到技术骨干

**Time：**2021/2/12

**Video link：**https://www.bilibili.com/video/BV1QD4y1X7oD?t=4

**Note content：**

Wps的linux版本是用qt开发的

针对于上位机算法人员的通信课题专讲（串口UART和can）

### 5 机器人如何感知世界-赵开勇

**Time：**2021/2/12

**Video link：**https://www.bilibili.com/video/BV1Xv411B7tD

**Note content：**

多传感器融合问题

GPS-RTK：GPS差分计算 精度比较高适用于室外环境 一个卫星多个基站差分计算

Radar 天线雷达、激光雷达  信号的波段不同  毫米波雷达：车载使用比较多  可以看出玻璃和塑料瓶，但是其他雷达不能

传感器挂掉的时候不能对整个系统产生较大的影响

多机器共同建图  ---论文  多智能体同时建模  北大的一篇文章



**Markdown笔记原文地址：**E:\studying\往期学习\讲座随笔
