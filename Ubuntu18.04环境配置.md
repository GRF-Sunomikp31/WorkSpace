# Ubuntu18.04环境配置

Introduction:  ubuntu18.04软件配置

### 1/安装clion  

参考教程：https://blog.csdn.net/weixin_43784003/article/details/116458216

### 2/安装typora（二进制源码安装方式）

下载typora软件安装包：https://typora-download.oss-cn-shanghai.aliyuncs.com/linux/typora_1.0.3_amd64.deb

`sudo dpkg -i typora_1.0.3_amd64.deb` 

### 4/安装sougou输入法（仅配置中文输入法，不改变系统语言）

参考：https://blog.csdn.net/ibiao/article/details/89333581

ctrl+空格可以切换输入法；

如果安装了搜狗输入法有显示图标，但是切换到搜狗输入法输入仍为英语，这时缺少一些相关库，安装命令如下：

`sudo apt install libqt5qml5 libqt5quick5 libqt5quickwidgets5 qml-module-qtquick2`

`sudo apt install libgsettings-qt1`

参考官方教程：https://pinyin.sogou.com/linux/help.php

## Todo list

docker

ROS

