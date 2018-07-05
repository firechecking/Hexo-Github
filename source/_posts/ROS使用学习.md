---
title: ROS使用学习
date: 2017-05-27 16:39:09
tags: [机器人,ROS]
categories: [机器人,ROS]
comments: true
---

# 参考文档
1. <http://www.ros.org/support/>
1. <http://wiki.ros.org/>
1. <http://wiki.ros.org/cn>
<!-- more -->

# 安装
## ROS Lunar Loggerhead

# 相关地址
# Learning ROS
1. [概览](http://wiki.ros.org/ROS/Introduction)
1. [学习教程](http://wiki.ros.org/ROS/Tutorials)
1. [ROS核心文档](http://wiki.ros.org/ROS)

# Finding Answers
1. <http://wiki.ros.org/>
2. <http://answers.ros.org/>
3. <http://answers.ros.org/questions/ask/>

# 教程
## Core ROS
### 安装与配置
1. [Ubuntu install of ROS Lunar](http://wiki.ros.org/lunar/Installation/Ubuntu)

1. 安装Package

```
sudo apt-get install ros-kinetic-PACKAGE
```
1. 管理ROS Package
	1. <http://wiki.ros.org/ROS/Tutorials/rosdep>
		
	```
	rosdep install [package]
	```

1. 配置环境
	1. 检查ROS_ROOT、ROS_PACKAGE_PATH环境变量:`$ printenv | grep ROS`
	1. 如果环境变量不存在，需要运行setup.bash:`$ source /opt/ros/lunar/setup.bash`
1. 创建ROS工作空间

```
$ mkdir -p ~/catkin_ws/src
$ cd ~/catkin_ws/
$ catkin_make
$ source devel/setup.bash
$ echo $ROS_PACKAGE_PATH
```
### ROS文件系统教程
1. 基本概览
	1. Package：程序代码的组织单元，Package可以包含库、可执行文件、脚本
	1. Manifests(package.xml):Package的描述，用于定义package的元信息间的依赖关系（如版本、维护者、许可协议等）
1. 文件系统工具
	1. rospack：获取package信息，如rospack find [package_name]

### 创建ROS Package
Package必须包含：
1. package.xml：提供package的元信息
2. CMakeLists.txt两个文件文件。
Workspace是一组Package的集合，其组织形式如下：

```
workspace_folder/        -- WORKSPACE
  src/                   -- SOURCE SPACE
    CMakeLists.txt       -- 'Toplevel' CMake file, provided by catkin
    package_1/
      CMakeLists.txt     -- CMakeLists.txt file for package_1
      package.xml        -- Package manifest for package_1
    ...
    package_n/
      CMakeLists.txt     -- CMakeLists.txt file for package_n
      package.xml        -- Package manifest for package_n
```

1. 创建Package

```
$ cd ~/catkin_ws/src
$ catkin_create_pkg beginner_tutorials std_msgs rospy roscpp
```
### 编译ROS Package
1. 编译之前首先确保已经运行`$ source ~/catkin_ws/devel/setup.bash`
1. 编译命令`$ catkin_make [make_targets] [-DCMAKE_VARIABLES=...]`
1. 在catkin_ws目录，运行catkin_make，catkin_make install，将编译catkin_ws/src目录下找到的所有package

### ROS节点认识
1. Nodes：一个节点是一个可执行程序，通过ROS与其他节点通信
1. Messages：订阅和发布主题时，用到的ROS数据类型
2. Topics：节点发布消息到一个主题，或订阅一个主题的消息
3. Master：ROS名称服务（帮助节点找到彼此）
4. rosout：ros中相当于stdout/stderr
5. roscore：Master+rosout+参数服务器（parameter server）

# URDF
[URDF学习笔记](../../../06/06/URDF学习/)
# MoveIt
[如何利用ROS MoveIt快速搭建机器人运动规划平台？](https://www.leiphone.com/news/201612/nxlXgriSLasNgAcX.html?viewType=weixin)