# ROS入门21讲

## 基本信息

**学习时间**：2021/3/3

**学习资料**：

- 视频链接：https://www.bilibili.com/video/BV1zt411G7Vn
- github：https://github.com/huchunxu/ros_21_tutorials

**文档内容**：Notes

**学习目标**

- 强化ROS发展历史以及相关概念
- 重点学习ROS框架下代码编写，各种通信协议
- 分析一个仿真SLAM包的结构

**学习平台**：Ubuntu18.04+Melodic

## 学习笔记

### 1 课程介绍

ROS是机器人领域的普遍标准；

参考资料：google，精确搜索；

### 3 Linux系统基础操作

linux下还是命令行好用，很多操作（比如复制删除）是不能通过鼠标操作的，鼠标控制方式都是在user的权限下而不是root。

命令：`rm --help`  查看rm的相关使用方法和其他参数。

### 4 C++&Python极简基础

g++是C++的编译器，gcc是C的编译器，两者不能混用；Python的解析器（python是解析性语言，不需要编译）就是python；

类中定义的变量-（称为）属性，函数-方法；

### 6 ROS是什么

手写ROS底层？回头可以感受一下

ROS下各种通信协议速率问题？速率快慢比较

ROS2可以直接再windows下开发使用，ROS1在windows下支持的不是很好；2018微软引入ROS在windows下；亚马逊推出Robomaker云服务平台

利用手机做主控开发ROS； 

### 7 ROS的核心概念

ROS的通信机制可以传递任意数据信息吗？比如视频流等等

启动文件（Launch File）是ROS中一种同时启动多个节点的途径，还可以**自动启动ROS Master**节点管理器，而且可以实现每个节点的各种配置，为多个节点的操作提供了很大便利。

在rviz中，这样的机器人模型是通过urdf文件描述的。

参数通信机制：全局共享字典   话题和服务的底层通信协议都是基于TCP/UDP的

### 8 ROS命令行工具的使用

**常用命令：**

- rostopic:关于话题的命令（直接输入rostopic会有帮助信息）`rostopic list`：显示出系统中所有话题的名称，rostopic pub 话题名 消息类型“数据”：发布话题
- rosservice:`rosservice list`:查看系统中服务，
- rosnode:显示系统中所有节点信息的指令：使用方式：rosnode+参数，rosnode（直接敲会有帮助提示），`rosnode list`:显示系统中所有节点、，rosnode info+具体节点名称：查看这个节点的具体信息，
- rosparam:
- rosmsg:rosmsg show +消息名称：查看消息的数据结构
- rossrv:
- roscore：启动ROS master，ROS中的节点管理器
- rosrun：运行节点（运行某个功能包中某个节点的命令）后面需要+**两个参数：功能包名、节点名** 
- rosbag：话题记录的工具：这个用于记录、保存数据，供后面调试；

rqt工具：rqt是ros下基于qt做的可视化工具

- `rqt_graph`:显示ROS系统下的计算图

/rosout节点：是ros环境中默认启动的话题，主要是采集其他节点的日志信息给其他做显示

### 9  创建工作空间与功能包

**工作空间：**是一个存放ROS工程开发相关文件的文件夹

- src：代码空间（Source Space）：存放功能包
- build：编译空间（Build Space）:编译过程中产生中间文件的文件夹
- devel：开发空间（Development Space）：编译生成的可执行文件（库、脚本）
- install：安装空间（install space）：用install安装的文件（开发空间和安装空间有很多重复的，所以在ROS2中只保留了开发空间）

**创建工作空间**

- `mkdir -p ~/catkin_ws/src`     `cd  ~/catkin_ws/src`
- `catkin_init_workspace`    初始化，将这个文件夹初始化为ROS的workspace
- `cd ~/catkin_ws/`    编译工作空间：`catkin_make`(必须在根目录编译catkin_ws)
- `catkin_make install`   初始化install文件夹(开发结束后，分享给客户的结果文件)
- `source devel/setup.bash`   设置环境变量
- `echo $ROS_PACKAGE_PATH`   检查环境变量

**创建功能包**

- `cd  ~/catkin_ws/src`  `catkin_create_pkg  test_pkg std_msgs  rospy roscpp`  test_pkg是要创建的功能包名，std_msgs  rospy roscpp 为要依赖的功能包名。
- `cd ~/catkin_ws`   `catkin_make` 编译功能包，`source ~/catkin_ws/devel/setup.bash`  设置工作空间的环境变量（只有这样系统才能找到功能包）

同一工作空间下，不允许存在同名功能包；不同工作空间下，允许存在同名功能包；

（都添加都环境变量中）多个功能包启动顺序：看环境变量的顺序？

所有源码都必须放在功能包下面；

在功能包文件夹里面，

- src是放置功能包代码；
- include是放置头文件的；
- cmakelist.txt和package.xml是功能包必须要有的文件；
- cmakelist.txt:描述功能包的编译规则，使用的是cmake的语法；
- package.xml:文件内容是xml语言描述的，主要就是功能包的信息；

### 10 发布者Publisher的编程实现

![4](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/ROS/4.png)

```c++
/*
	*该历程将发布turtle/cmd_vel话题，消息类型geometry_msgs::Twist
*/
#include<ros/ros.h>
#include<geometry_msgs/Twist.h>

int main(int argc,char **argv)
{
	//ROS节点初始化
	ros::init(argc,argv,"velocity_publisher");
	//创建节点句柄
	ros::NodeHandle n;
	//创建一个publisher，发布名为/turtle1/cmd_vel的topic，消息类型为gemetry_msgs::Twist,队列长度10
	ros::Publisher turtle_vel_pub = n.advertise<geometry_msgs::Twist>("/turtle1/cmd_vel",10);
	//创建循环的频率
	ros::Rate loop_rate(10);
	
	int count = 0;
	while(ros::ok())
	{
		//初始化geometry_msgs::Twist类型的消息
		geometry_msgs::Twist vel_msg;
		vel_msg.linear.x =0.5;
		vel_msg.angular.z =0.2;

		//发布消息
		turtle_vel_pub.publish(vel_msg);
		ROS_INFO("Publish turtle velocity command[%0.2f m/s,%0.2f rad/s]",vel_msg.linear.x,vel_msg.angular.z);
		
		//按照循环频率延时
		loop_rate.sleep();

	}
	return 0;
}
```

发布者代码设计流程：

- 节ROS点初始化
- 创建发布者对象
- 创建消息数据
- 按照一定频率循环发布消息
- cmakelist.txt:`add_executeable(velocity_publisher  src/velocity_publisher.cpp)`   #将velocity_publisher.cpp编译成可执行文件velocity_publisher
  `target_link_libraries(velocity_publisher ${catkin_LIBRARIES})`    #链接ROS库

`rosrun learning_topic velocity_publisher` learning_topic功能包下velocity_publisher`节点；

```python
#!/usr/bin/env python
# -*- coding: utf-8 -*-

########################################################################
####          Copyright 2020 GuYueHome (www.guyuehome.com).          ###
########################################################################

# 该例程将发布turtle1/cmd_vel话题，消息类型geometry_msgs::Twist

import rospy
from geometry_msgs.msg import Twist

def velocity_publisher():
	# ROS节点初始化
    rospy.init_node('velocity_publisher', anonymous=True)

	# 创建一个Publisher，发布名为/turtle1/cmd_vel的topic，消息类型为geometry_msgs::Twist，队列长度10
    turtle_vel_pub = rospy.Publisher('/turtle1/cmd_vel', Twist, queue_size=10)

	#设置循环的频率
    rate = rospy.Rate(10) 

    while not rospy.is_shutdown():
		# 初始化geometry_msgs::Twist类型的消息
        vel_msg = Twist()
        vel_msg.linear.x = 0.5
        vel_msg.angular.z = 0.2

		# 发布消息
        turtle_vel_pub.publish(vel_msg)
    	rospy.loginfo("Publsh turtle velocity command[%0.2f m/s, %0.2f rad/s]", 
				vel_msg.linear.x, vel_msg.angular.z)

		# 按照循环频率延时
        rate.sleep()

if __name__ == '__main__':
    try:
        velocity_publisher()
    except rospy.ROSInterruptException:
        pass

```

### 11 订阅者Subscriber的编程实现

实现一个订阅者

- 初始化ROS节点
- 订阅需要的话题
- 循环等待话题消息，接受到消息后进入回调函数
- 在回调函数中完成消息处理

**代码**

```c++
/*
	该例程将订阅/turtle1/pose话题，消息类型turtlesim::Pose
*/

#include<ros/ros.h>
#include "turtlesim/Pose.h"

void poseCallback(const turtlesim::Pose::ConstPtr& msg)
{
	//将接受到的消息打印出来
	ROS_INFO("Turtle pose: x:%0.6f,y:%0.6f",msg->x,msg->y);
}


int main(int argc,char **argv)
{
	//初始化ROS节点
	ros::init(argc,argv,"pose_subscriber");
	
	//创建节点句柄
	ros::NodeHandle n;


	//创建一个Subscriber,订阅名为/turtle1/pose的topic，注册回调函数poseCallback
	ros::Subscriber pose_sub = n.subscribe("/turtle1/pose",10,poseCallback);

	//循环等待回调函数
	ros::spin();

	return 0;

}
```

使用python写的脚本是不需要编译的，就是不需要添加到Cmakelist.txt中，直接运行。

订阅者的回调函数不能太长，这样会影响数据的接收；

### 12 话题消息的定义与使用

![5](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/ROS/5.png)

**自定义话题消息结构类型**：

- 定义msg文件（和语言无关）
- 在package.xml中添加功能包依赖
- 在Cmakelist.txt添加编译选项
- 编译生成语言相关文件

### 13 客户端Client的编程实现

![6](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/ROS/6.png)

**Client编程：**

-  初始化ROS节点
-  创建一个Client实例
-  发布服务请求数据
-  等待Server处理之后的应答结果

**代码如下：**

```c++
/***********************************************************************
Copyright 2020 GuYueHome (www.guyuehome.com).
***********************************************************************/

/**
 * 该例程将请求/spawn服务，服务数据类型turtlesim::Spawn
 */

#include <ros/ros.h>
#include <turtlesim/Spawn.h>

int main(int argc, char** argv)
{
    // 初始化ROS节点
	ros::init(argc, argv, "turtle_spawn");

    // 创建节点句柄
	ros::NodeHandle node;

    // 发现/spawn服务后，创建一个服务客户端，连接名为/spawn的service
	ros::service::waitForService("/spawn");
	ros::ServiceClient add_turtle = node.serviceClient<turtlesim::Spawn>("/spawn");

    // 初始化turtlesim::Spawn的请求数据
	turtlesim::Spawn srv;
	srv.request.x = 2.0;
	srv.request.y = 2.0;
	srv.request.name = "turtle2";

    // 请求服务调用
	ROS_INFO("Call service to spwan turtle[x:%0.6f, y:%0.6f, name:%s]", 
			 srv.request.x, srv.request.y, srv.request.name.c_str());

	add_turtle.call(srv);

	// 显示服务调用结果
	ROS_INFO("Spwan turtle successfully [name:%s]", srv.response.name.c_str());

	return 0;
};
```

### 14 服务器Server的编程实现

**Server编程：**

- 初始化ROS节点
- 创建Server实例
- 循环等待服务请求，进入回调函数
- 在回调函数中完成服务功能的处理，并反馈应答数据

### 15 服务数据的定义与使用

自定义服务数据。

### 16 参数的使用与编程方法

![7](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/ROS/7.png)

在ROS Master配置的变量可以让全局访问。

YAML参数文件：将所有参数一次性列出吧，方便一次性加载

参数命令行的使用  rosparam

- rosparam list：列出当前所有参数
- rosparam get param_key （参数名）：获取（显示）这个参数的值
- rosparam set param_key param_value:设置某个参数值
- rosparam dump file_name：保存参数到文件    经常保存为.yaml文件
- rosparam load file_name：从文件读取参数
- rosparam delete param_key：删除参数

**编程实现：**

- 初始化ROS节点
- get函数获取参数
- set函数设置参数

### 17 ROS中的坐标系管理系统tf

TF其实是一个功能包；默认会记录10s之内的坐标系关系

TF坐标变换的实现   TF树

- 广播TF变换
- 监听TF变换

所有针对一个车：需要激光雷达坐标系（base_laser）、车坐标系（base_link）...两个坐标系是有平移和旋转的关系

`tf tf_echo turtle1 turtle2`  查询ROS的tf树下任意两个坐标系的变换关系

也可以用RVIZ显示TF

### 18 tf坐标系广播与监听的编程实现

如何编程实现一个TF广播器

- 定义TF广播齐齐
- 创建坐标变换值
- 发布坐标变换

### 19 launch启动文件的使用方法

lauch启动文件：通过XML文件实现多节点的配置和启动(可自动启动ROS Master)

启动launch文件的时候会检测是否启动ROS  Master，没启动的话会默认启动

![8](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/ROS/8.png)

XML文件

- launch文件中的根元素采用<launch>标签定义

<node>   启动节点

- pkg：节点所在的功能包的名称
- type：节点的可执行文件的名称
- name：节点运行时的名称(重新命名，相当于节点名，取代初始化的，这样就可以启动多个一样的节点)

<param>  加载参数

- name：变量名
- value：参数值

<arg>  launch文件内部使用的局部变量

- name：参数名
- value：参数值

<remap> 重映射：重映射ROS计算图资源的命名(重新对资源命名)

- from：原命名
- to：映射之后的命名

<include> 嵌套：包含其他的launch文件，类似于C语言的头文件中的包含

- file：包含的其他launch文件路径



**lauch文件运行方法**：roslaunch  功能包名  功能包内lauch文件名

### 20 常用可视化工具的使用

**Qt工具箱**

- rqt_graph:计算图可视化工具
- rqt_image_view:图像渲染工具
- rqt_plot:数据绘图工具； 匹配话题信息
- rqt_console:日志输出工具
- rqt：所有工具

rqt的工具选择显示匹配的都是**话题名**；

**Rviz**：机器人开发过程中的数据可视化界面（其订阅的也都是话题）

**Gazebo**：三维物理仿真平台

### 21 课程总结与进阶攻略

学完21讲之后:做仿真，在gazebo自己画一个机器人实现SLAM功能，然后再从sw导入模型完成SLAM功能。

算法和硬件平台的适配的问题；

## 总结

**学完ROS入门21讲应该学什么进阶**：

- 找一本教程，系统学习   ：这里就使用那本ROS机器人编程实践吧

- 找一个项目，小试牛刀  ：这里就用RM那个官步吧

- 找准一个学习方向，深入学习 ： 

  ![1](https://github.com/GRF-Sunomikp31/WorkSpace/blob/main/Img/ROS/1.png)

## 其他思考

抛开机械臂和SLAM这些实际应用的方案，ROS的真正优势在于什么？编译工具？

手写ROS底层？

ubuntu下添加源的问题，本身系统安装软件有软件源，但是像ROS需要单独的源，如何分类这些？Ubutnu系统的软件商城，软件与更新 可以配置软件商城的软件源。

roboware安装包 https://blog.csdn.net/iwanderu/article/details/104414746  这个好用吗   有人推荐Roboware Studio

ROS2怎么使用？

ROS有那么多优势，那么ROS的劣势是什么？

功能包就是一个包含很多节点的组合吗？

为什么在linux下不能直播？

## 其他资料

古月居：

- 官网：https://www.guyuehome.com/
- 论坛：https://www.guyuehome.com/forums

其他跟个人笔记：https://github.com/HuangCongQing/ROS

机器人学基本理论的两门课

- 斯坦福大学公开课——机器人学:https://www.bilibili.com/video/av4506104

- 台湾交通大学--机器人学：:https://www.bilibili.com/video/av18516816

zhangrelay的专栏：https://www.csdn.net/zhangRelay

开源机器人学学习指南：https://github.com/qqfly/how-to-learn-robotics
