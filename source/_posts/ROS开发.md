---
title: ROS开发
date: 2017-08-21 15:46:37
tags: [机器人,ROS]
categories: [机器人,ROS]
comments: true
---

# msg、srv

<!--more-->
1. msg：msg目录下.msg后缀的文本文件
	1. 每行包括数据类型、变量名称，支持以下数据类型

	```
	Header
	int8, int16, int32, int64 (plus uint*)
	float32, float64
	string
	time, duration
	other msg files
	variable-length array[] and fixed-length array[C]
	```
	1. 包括Header、string、和其他msg的样例：
	
	```
	Header header
  string child_frame_id
  geometry_msgs/PoseWithCovariance pose
  geometry_msgs/TwistWithCovariance twist
	```
1. srv：srv目录下.srv后缀的文本文件
	1. 和msg格式类似
	1. 区别是不仅包括请求，还包括答复格式，以“---”分隔
	1. 样例：

	```
	int64 A
	int64 B
	---
	int64 Sum
	```
	
##  msg使用
### 创建msg
1. 在msg目录创建并编辑Num.msg文本文件
1. 在package.xml中添加依赖

```
<build_depend>message_generation</build_depend>
<run_depend>message_runtime</run_depend>
```
1. 在CMakeLists.txt中添加支持

```
find_package(catkin REQUIRED COMPONENTS
   roscpp
   rospy
   std_msgs
   message_generation
)
```
```
catkin_package(
  ...
  CATKIN_DEPENDS message_runtime ...
  ...)
```
```
add_message_files(
  FILES
  Num.msg
)
```
```
generate_messages(
  DEPENDENCIES
  std_msgs
)
```
1. 工具
	1. rosmsg show
	1. rosmsg list

## srv使用
1. 在srv目录创建并编辑AddTwoInts.srv文本文件
1. 在package.xml中添加依赖：

```
  <build_depend>message_generation</build_depend>
  <run_depend>message_runtime</run_depend>
```
```
find_package(catkin REQUIRED COMPONENTS
  roscpp
  rospy
  std_msgs
  message_generation
)
```
```
add_service_files(
  FILES
  AddTwoInts.srv
)
```
1. 工具
	1. rossrv show
	1. rossrv list

## 编译使用
1. 在workspace根目录运行`catkin_make`
1. ros会根据msg、srv文件创建c++、python、list的代码