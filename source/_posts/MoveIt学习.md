layout: ros
title: MoveIt学习
date: 2017-06-05 16:08:30
tags: [机器人,ROS,MoveIt]
categories: [机器人,ROS,MoveIt]
comments: true
---

# 参考文档
1. <http://moveit.ros.org/>
1. <http://moveit.ros.org/documentation/concepts/>
1. <http://docs.ros.org/indigo/api/moveit_tutorials/html/>
1. <http://www.ncnynl.com/archives/201610/1028.html>
1. <http://www.roswiki.com/read.php?tid=402>

<!-- more -->
# MoveIt介绍
MoveIt！是目前针对移动操作最先进的软件。它结合了运动规划，操纵，三维感知，运动学，控制和导航的最新进展。提供了一个易于使用的平台，开发先进的机器人应用程序，评估新的机器人设计和建筑集成的机器人产品。它广泛应用于工业，商业，研发和其他领域。

## 结构
![](http://moveit.ros.org/wordpress/wp-content/uploads/2013/12/Overview.0012.jpg)

## move_group
move_group是一个ROS node，整合多个独立组件，并提供ROS风格的Action和Service

## User Interface
1. 用户通过user interface与move_group提供的action和service进行交互，user interface提供了C++、Python、GUI三种方式
1. move_group的配置参数在ROS param server中，URDF或SRDF模型文件数据也是保存在ROS param server供move_group调用

## Configuration
move_group是一个常规的ROS node，通过ROS param server获取以下三类信息：

1. URDF：获取描述机器人模型的URDF模型文件数据

1. SRDF：获取SRDF(robot_description_semantic)数据，通常由用户使用MoveIt!Setup Assistant工具生成

1. Configuration：move_group冲param server中获取关节描述、运动学、动作规划、传感器等信息。配置文件由MoveIt! Setup Assistant生成，存放在MoveIt! config目录中。

## Robot Interface
move_group通过ROS topic和action和机器人进行通信，从而获取位置、关节等状态信息，点云等传感器数据，并且控制机器人运动

### Joint State Information
move_group订阅/joint_states主题，来接收关节信息。

### Transform Information
move_group使用ros的tf库监听变换信息，从而使move_group能够获取到机器人的全局位置信息。举例说明：导航模块将机器人的移动时，现对于地图的变换信息发布到tf库，move_group再从tf库获取到这个位置变换信息。

注：move_group不会发布变换信息到tf，因此想从机器人发布tf变换信息，需要自己运行robot_state_publisher的node

### Controller Interface
move_group使用FollowJointTrajectoryAction来控制机器人运动，FollowJointTrajectoryAction是一个ROS的action接口，需要在机器人上对这个action进行响应，这个action由机器人创建，move_group只是作为客户端和action server进行通信

### Planning Scene
move_group使用规划场景监视器来管理规划场景，场景用于表达世界状态及机器人状态，机器人状态包括机器人本体及所有刚性连接到机器人上的物体。后续有planning scene章节对词进行详细介绍

### Extensible Capabilities
move_group的结构被设计为易于扩展，独立的如抓取、运动学、运动规划等，实际上是扩展自通用基类的单独插件，这些单独的插件可通过一些ros yaml参数和ros pluginlib库进行配置。大多数情况下，move_group插件由MoveIt Setup Assistant自动生成的launch文件完成自动配置，而不需要用户手动配置

# Motion Planning

## The Motion Planning Plugin
MoveIt使用插件进行运动规划，这种方式允许MoveIt使用不同的运动规划器。MoveIt和运动规划器之间通过move_group提供的一个ros action或service进行通信，MoveIt的默认运动规划器由MoveIt Setup Assistant完成配置，使用OMPL和OMPL的MoveIt接口

## The Motion Plan Request
Motion Plan Request表示需要运动规划器做的事情，例如想让运动规划器移动手臂到不同位置，或末端执行器变成新的姿势，默认情况下碰撞检测是打开的，还可以附加其他物体到末端执行器或机器人其他部位，这样，运动规划器在规划路径时，可以考虑到新增物体的运动。

另外还可以制定运动规划的约束条件，内置的约束条件是运动学约束：

1. 位置约束：限制连接位置位于某个空间区域
1. 方向约束：限制连接方向位于roll、pitch、yaw限制范围
1. 可视化约束：限制某个点在某个传感器的可视锥形范围
1. 关节约束：限制关节值处于某个范围
1. 自定义约束：可以使用用户自定义回调，来指定自己的约束

## The Motion Plan Result
接收到Motion Plan Request，move_group节点会产生一系列规划路径，来将机器人手臂、关节等移动到目标位置。move_group产生的规划路径不仅包括运行路线，还会根据速度、加速度约束来生成运动轨迹

## The Motion Planning Pipeline
运动规划器和规划请求适配器

一个完整的运动规划链，包括一个运动规划器和多个规划请求适配器。规划请求适配器可以接收运动规划预处理请求，以及运动规划后处理响应。前处理可以用在一些场景，如：当机器人初始状态轻微偏离关节限制时。很多操作需要用到后处理，如：将生成的路径装换成带时间参数的路径规划。

MoveIt提供一些默认的运动规划适配器来完成特定功能。

### FixStartStateBounds
适配器修复机器人开始状态到符合URDF定义的关节约束内。当设置错误等原因造成机器人初始状态位于关节约束以外时，运动规划器无法完成路径规划，这时“FixStartStateBounds”可以将关节移动到约束以内，以修复初始状态。并不是任何时候都可以使用FixStartStateBounds来进行初始状态修复，因此FixStartStateBounds有参数可以设置可修复的范围。

### FixWorkspaceBounds
适配器设置一个默认的工作空间进行运动规划：10米x10米x10米的立方体空间。当发送到规划器的规划请求没有工作空间的，这个适配器会生效

### FixStartStateCollision
适配器小范围调整关节值，来用户设置值附近选取一个无碰撞的参数。用户可以设置“jiggle_factor”来控制调节范围百分比（相对于该关节最大运动范围），还有其他参数设置适配器选取到合适的无碰撞参数前的最大尝试次数

### FixStartStatePathConstraints
当运动规划的初始状态不满足路径约束时，适配器尝试将机器人移动到满足路径约束的位置，并以新的位置作为初始状态进行路径规划

### AddTimeParameterization
运动规划器通常产生运动学路径（Kinematic paths），该运动学路径不包含任何速度、加速度约束及时间参数，适配器通过引入速度、加速度约束，来为规划路径产生时间参数

# OMPL
OMPL是一个开源的运动规划库，主要使用随机运动规划器。MoveIt直接集成了OMPL，并且使用OMPL的运动规划器作为默认规划期。OMPL中的规划器都是基于抽象概率，如OMPL中没有机器人概念。在MoveIt中，配置了OMPL并为OMPL提供后端工具，用于处理机器人问题。

## 规划场景
![](http://moveit.ros.org/wordpress/wp-content/uploads/2013/12/Overview.0031.jpg)

规划场景用于保存和表示机器人本身及所处环境状态，通过move_group中的planning scene monitor实现，规划场景监视器监听以下信息：
1. 状态信息：在joint_states topic下
1. 传感器信息：通过下文中的geometry monitor
1. 世界几何信息：在planning_scene topic中的用户输入

## World Geometry Monitor
世界几何信息监视器通过传感器、用户输入来创建世界几何信息，通过occupancy map monitor来创建机器人周围环境的三维表示，使用planning_scene中的附带信息来增加物体信息。

## 3D Perception
![](http://moveit.ros.org/wordpress/wp-content/uploads/2013/12/Overview.004.jpg)

可运动地图监视器完成三维感知，使用插件结构处理不同的传感器输入。MoveIt内建的直接支持以下两种输入数据：
1. 点云：由point cloud occupancy map updater插件处理
2. 深度图像：depth image occupancy map updater插件处理

用户可以自由添加其他数据插件到occupancy map monitor

### Octomap
Occupancy map monitor使用[Octomap](http://octomap.github.io/)库表示环境的可运动区域地图，Ocotmap数据可以直接在MoveIt的碰撞检测库FCL中使用

### Depth Image Occupancy Map Updater
深度图像可行区域更新插件有自过滤器，会将机器人本体的可见区域从深度图像中去除（使用机器人当前状态信息来完成这个操作）

# Kinematics
## 运动学插件
MoveIt使用插件系统，允许用户编写自己的逆运动学算法。正运动学算法、查找雅克比矩阵已经集成到RobotState类中，默认的逆运动学插件由MoveIt! Setup Assistant完成配置，使用[KDL](http://www.orocos.org/kdl) numerical jacobian-based solver

## IKFast插件
用户通常希望使用自己的运动学求解器，如PR2有自己的运动学求解器，最常使用的方法是使用IKFast来生成适用于自己机器人的C++求解器

# 碰撞检测
MoveIt的碰撞检测在Planning Scene中使用CollisionWorld对象完成，碰撞检测在FCL中集成，用户不需要额外操作。

MoveIt支持一下物体的碰撞检测:
1. mesh：网格模型
1. 基本形状：立方体、圆柱、圆锥、球、平面等
2. Octomap：Octomap物体可以直接用于碰撞检测

## 运行碰撞矩阵（ACM, Allowed Collision Matrix）
碰撞检测通常会消耗掉路径规划的90%计算资源，因此通过设置ACM矩阵定义哪些物体之间不需要进行碰撞检测，可以大大节省计算资源，当对应于两个物体的值在ACM中设置为1，则不会对这两个物体之间进行碰撞检测。

