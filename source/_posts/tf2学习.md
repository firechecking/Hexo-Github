---
title: tf2学习
date: 2017-06-06 20:32:48
tags: [机器人,ROS,tf2]
categories: [机器人,ROS,tf2]
comments: true
---

# 简介
TF(transform)工具包用于追踪不同坐标系下随时间变化的帧位置。TF工具包以树结构保存不同时间、不同坐标系下的位置关系，并运行在求取任意两个坐标系在任意时间的坐标变换。`ROS Hydro之后，tf工具包已经转为tf2工具包`<!--more-->

# 参考链接
1. <http://wiki.ros.org/tf2>

# tf2工具包做什么？怎么用？

[演示demo](http://wiki.ros.org/tf2/Tutorials/Introduction%20to%20tf2)

机器人开发中有多个坐标系随着时间变化，tf2可以跟踪这些变化，并可查询如如下内容：

1. 5秒钟之前，头部坐标相对于时间坐标的值是多少？
2. 手指中的物体坐标相对于手臂坐标是多少？
3. 世界中标中当前位置坐标是多少？

# 内容
## Static Broadcaster
1. 广播一个坐标变换
	
	```
	#!/usr/bin/env python
	import rospy
	# to get commandline arguments
	import sys
	#Because of transformations
	import tf
	import tf2_ros
	import geometry_msgs.msg
		
	rospy.init_node('my_static_tf2_broadcaster')
	# tf2中有StaticTransformBroadcaster类用于广播静态变换
	broadcaster = tf2_ros.StaticTransformBroadcaster()
	# static_transformStamped是需要发送的变换信息
	static_transformStamped = geometry_msgs.msg.TransformStamped()
	# 变换时间
	static_transformStamped.header.stamp = rospy.Time.now()
	# 变换父坐标系
	static_transformStamped.header.frame_id = "world"
	# 变换子坐标系
	static_transformStamped.child_frame_id = sys.argv[1]
	  
	static_transformStamped.transform.translation.x = float(sys.argv[2])
	static_transformStamped.transform.translation.y = float(sys.argv[3])
	static_transformStamped.transform.translation.z = float(sys.argv[4])
	  
	quat = tf.transformations.quaternion_from_euler(float(sys.argv[5]),float(sys.argv[6]),float(sys.argv[7]))
	static_transformStamped.transform.rotation.x = quat[0]
	static_transformStamped.transform.rotation.y = quat[1]
	static_transformStamped.transform.rotation.z = quat[2]
	static_transformStamped.transform.rotation.w = quat[3]
	# 以上几步后，设置了变换的6个自由度（平移+旋转）
		
	broadcaster.sendTransform(static_transformStamped)
	rospy.spin()
	```
1. 执行以下命令运行Broadcaster(详细代码及过程见[此](http://wiki.ros.org/tf2/Tutorials/Writing%20a%20tf2%20static%20broadcaster%20%28Python%29))，参数表示沿z轴平移1米

	```
	 $ rosrun learning_tf2 static_turtle_tf2_broadcaster.py mystaticturtle 0 0 1 0 0 0
	```
1. 当Broadcaster运行后，会向名为tf_static的topic广播变换内容

	```
	transforms:
	-
	header:
	  seq: 0
	  stamp:
	    secs: 1459282870
	    nsecs: 126883440
	  frame_id: world
	child_frame_id: mystaticturtle
	transform:
	  translation:
	    x: 0.0
	    y: 0.0
	    z: 1.0
	  rotation:
	    x: 0.0
	    y: 0.0
	    z: 0.0
	    w: 1.0
	---
	```

## Broadcaster
1. 内容参考上一节[Static Broadcaster](#Static-Broadcaster)，以及ROS官网[Broadcaster](http://wiki.ros.org/tf2/Tutorials/Writing%20a%20tf2%20broadcaster%20%28Python%29)
1. 与Static Broadcaster的区别
	1. Static Broadcaster发送到tf_static主题
	1. Static Broadcaster只在需要的时候发送，通常只在启动的时候发送一次

## Listener
1. Listener代码

	```
	#!/usr/bin/env python  
	import rospy
	
	import math
	import tf2_ros
	import geometry_msgs.msg
	import turtlesim.srv
	
	if __name__ == '__main__':
	    rospy.init_node('tf2_turtle')
	
	    tfBuffer = tf2_ros.Buffer()
	    # 创建tf2_ros.TransformListener用于自动接收变换消息，并缓存10秒内的所有变换
	    listener = tf2_ros.TransformListener(tfBuffer)
	
	    rospy.wait_for_service('spawn')
	    spawner = rospy.ServiceProxy('spawn', turtlesim.srv.Spawn)
	    turtle_name = rospy.get_param('turtle', 'turtle2')
	    spawner(4, 2, 0, turtle_name)
	
	    turtle_vel = rospy.Publisher('%s/cmd_vel' % turtle_name, geometry_msgs.msg.Twist, queue_size=1)
	
	    rate = rospy.Rate(10.0)
	    while not rospy.is_shutdown():
	        try:
	        		# 根据参数查询变换
	        		# 参数1：源坐标系
	        		# 参数2：目标坐标系
	        		# 查询的变换时间：rospy.Time(0)获取最新的可用变换
	            trans = tfBuffer.lookup_transform(turtle_name, 'turtle1', rospy.Time())
	        except (tf2_ros.LookupException, tf2_ros.ConnectivityException, tf2_ros.ExtrapolationException):
	            rate.sleep()
	            continue
	
	        msg = geometry_msgs.msg.Twist()
	
	        msg.angular.z = 4 * math.atan2(trans.transform.translation.y, trans.transform.translation.x)
	        msg.linear.x = 0.5 * math.sqrt(trans.transform.translation.x ** 2 + trans.transform.translation.y ** 2)
	
	        turtle_vel.publish(msg)
	
	        rate.sleep()
	```
1. 执行效果

	当使用键盘移动turtle时，turtle2会跟着一起运动
	
## 创建一个坐标轴
### 创建坐标轴的原因
1. 在应用过程中，创建一个本地内部的本地坐标轴通常更容易思考。
1. 如果在处理传感器时，创建位于每个传感器中心的坐标轴，然后只需要按照新的坐标轴处理，tf2可以处理所有的变换

### 在哪儿创建坐标轴
1. 在tf2中创建了一个树结构来保存所有的坐标轴
1. 坐标轴树中不允许出现循环
1. 任何一个坐标轴只有一个父坐标轴，但可以有多个子坐标轴
1. 完成上文的Braodcaster和Listener后，树中有如下三个坐标轴，新建的坐标轴必须从这三个坐标轴中选择一个作为父坐标轴
![](tf2学习/tree.png)

### 如何创建坐标轴
1. 代码如下

	```
	#!/usr/bin/env python  
	import rospy
	import tf2_ros
	import tf2_msgs.msg
	import geometry_msgs.msg
	
	class FixedTFBroadcaster:
	
	    def __init__(self):
	        self.pub_tf = rospy.Publisher("/tf", tf2_msgs.msg.TFMessage, queue_size=1)
	
	        while not rospy.is_shutdown():
	            # Run this loop at about 10Hz
	            rospy.sleep(0.1)
	
	            t = geometry_msgs.msg.TransformStamped()
	            t.header.frame_id = "turtle1"
	            t.header.stamp = rospy.Time.now()
	            # 新建名为carrot1的坐标轴，其副坐标轴为turtle1
	            t.child_frame_id = "carrot1"
	            t.transform.translation.x = 0.0
	            t.transform.translation.y = 2.0
	            t.transform.translation.z = 0.0
	
	            t.transform.rotation.x = 0.0
	            t.transform.rotation.y = 0.0
	            t.transform.rotation.z = 0.0
	            t.transform.rotation.w = 1.0
	
	            tfm = tf2_msgs.msg.TFMessage([t])
	            self.pub_tf.publish(tfm)
	
	if __name__ == '__main__':
	    rospy.init_node('my_tf2_broadcaster')
	    tfb = FixedTFBroadcaster()
	
	    rospy.spin()
	```
	
## 基于时间的tf2
1. 使用lookup_transform()，可以以阻塞的方式查询接下来一段时间内是否有两个坐标轴间的坐标变换

	```
	tfBuffer.lookup_transform('turtle2', 'turtle1', rospy.Time.now(), rospy.Duration(1.0))
	```

## 使用tf2进行延时坐标变换
1. 目的：让turtle2跟随着5秒前的turtle1运动轨迹运动
1. 错误写法：

	```
	past = rospy.Time.now() - rospy.Duration(5.0)
  trans = tfBuffer.lookup_transform(turtle_name, 'carrot1', past, rospy.Duration(1.0))
	```
	1. 实际效果：查询carrot1 5秒前的状态，然后以turtle2 5秒前的状态进行坐标变换
	1. 想要的效果：查询carrot1 5秒前的状态，然后以turtle2当前状态进行坐标变换
1. 正确写法：

	```
	past = rospy.Time.now() - rospy.Duration(5.0)
	trans = tfBuffer.lookup_transform_full(
    target_frame=turtle_name,
    target_time=rospy.Time.now(),
    source_frame='carrot1',
    source_time=past,
    fixed_frame='world',
    timeout=rospy.Duration(1.0)
	)
	```
	lookupTransform()完整的6个参数含义：
	1. 变换的目标坐标系
	1. 目标坐标系的时间
	1. 变换的参考坐标系
	1. 参考坐标系时间
	1. 不随时间变换的时间坐标系
	1. time-out（查询等待超时时间）

## 坐标在不同数据结构间的转换
[Create Data Conversion Package](http://wiki.ros.org/tf2/Tutorials/Create%20Data%20Conversion%20Package%20%28Python%29)

# 其他

## [tf2设计思路](http://wiki.ros.org/tf2/Design)

## 支持的数据类型
1. tf2提供模块化的数据类型支持，可以与任何外部package合作，主要有：
	1. tf2_bullet
	2. tf2_eigen
	2. tf2_geometry_msgs
	2. tf2_kdl
	2. tf2_sensor_msgs

## 相关工具
1. [tf2_echo](http://wiki.ros.org/tf2#tf2_echo)
1. [view_frames](http://wiki.ros.org/tf2#view_frames)
1. [rviz](http://wiki.ros.org/rviz)
