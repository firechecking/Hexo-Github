---
title: 20170508-NeuralNetworksForMachineLearning_Week1
date: 2017-03-17 17:26:23
tags: [机器学习,小组学习,机器人组]
categories: [机器人组,小组学习]
comments: true
---
# 简介
课程第二讲：首先介绍了神经网络的三种主要组织结构，然后着重介绍了感知机相关知识

<!-- more -->
# Types of neural network architecture
1. Feed-forward neural network(前馈神经网络)
	1. 第一层为输入层，最后一层为输出层，信息进行单向流通
	2. The last layer is the output
	3. 每一层获得输入层的新的表示方式，因此有机会让相似的地方变得不相似，或者让不相似的内容变得相似。
		1. 以语音识别为例，不同人的同样内容，需要相似输出
		2. 相同人的不同内容，需要不同输出
1. Recurrent neural network（循环神经网络）
	1. 隐藏神经元->隐藏神经元
	2. 通常循环神经网络中，每一层循环共享权重
	3. 之所以具有记忆效应，从图中能看出每一层循环计算出output后，output和input会继续传入下一层循环
1. Symmetrically connected network（对称连接网络）
	1. 循环神经网络的一种，双向权重相等
	1. 只有一层的对称连接网络：“Hopfield nets”
	1. 具有隐藏层的对称连接网路：“Boltzmann machines”
	1. <http://blog.csdn.net/omenglishuixiang1234/article/details/49531937>

# Perceptrons：The first generation of neural networks（感知机）
1. 标准的感知机框架
	1. 输入数据人为转化为特征向量
	1. 学习感知机中特征向量权重
	1. 由阈值判断output
1. 感知机训练流程
	1. 设置所有输入权重为1
	1. 然后将训练样本依次送入感知机进行训练
		1. 如果输出正确，保持权重不变
		1. 如果应该输出1，而错误输出了0，将1平均加到所有权重
		1. 如果应该输出0，而错误输出了1，将1从所有权重中平均减掉

# A geometrical view of Perceptrons（感知机的几何表示）
1. 想象一个高维空间（weight space）中，每一个点表示所有权重的一种取值
1. 每一个训练样本表示空间中的一个平面
1. 一个训练样本的某一面的所有点都是正确的，而另一面的所有点都是错误的
1. 当输入向量和权重向量位于平面同方向时，输出为正；位于平面反方向时，输出为负。
1. 因此当能找到一个区域的点，位于训练样本形成的所有平面的正确区域的共同区域，这些区域中的点都是感知机能正确工作的权重

# why the learning works
1. 在存在能让所有训练样本正确分类的权重情况下，称有效的权重为feasible weight vector
1. 训练过程中，每当有错误分类时，就将权重向量朝输入向量的方向靠近一点

# What Perceptrons can't do
1. 感知机的限制主要取决于特征向量的选取
1. 比较典型的是异或运算中，感知机无法正确工作
1. 在感知机中，需要正反输入数据能够线性可分
1. 在感知机中，硬编码的特征具有局限性，仅仅通过学习权重，在应用中受限制，因此需要能够学习到特征。
1. 在感知机中增加了隐含层，学习隐含层的权重，实际就是学习输入向量的特征。