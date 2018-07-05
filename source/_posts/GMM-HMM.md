---
title: GMM-HMM
date: 2017-03-13 16:24:23
tags: [机器学习,语音识别,GMM,HMM]
categories: [机器学习,语音识别]
comments: true
---

# 参考链接
1. <http://blog.csdn.net/abcjennifer/article/details/27346787>
1. <http://blog.csdn.net/wbgxx333/article/details/20479825>
1. <http://blog.csdn.net/wbgxx333/article/details/18516053>
1. <http://blog.csdn.net/wbgxx333/article/details/3900688><!--more-->

# HMM
1. 一个具有隐藏状态和可视表象的马尔科夫模型
1. 在语音识别识别中，隐藏状态是音素序列；从录音中提取MFCC特征后，使用混合高斯模型进行特征拟合，可视表象就是混合高斯模型的均值方差
1. 初始化时，手动设置HMM的参数。训练HMM时，给定n个时序信号y1...yT（训练样本），用MLE（typically implemented in EM）估计参数：
	1. N个状态的初始概率
	1. 状态转移概率a
	1. 输出概率b
1. 在语音处理中
	* 一个word由若干phoneme（音素）组成；
	* 每个HMM对应于一个word或者音素（phoneme）
	* 一个word表示成若干states，每个state表示为一个音素

# GMM
1. 高斯混合模型是几个高斯模型的叠加

![](http://img.blog.csdn.net/20140528180736578?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWJjamVubmlmZXI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

1. 每个state有一个GMM，包含k个高斯模型参数，如”hi“（k=3）：

![](http://img.blog.csdn.net/20140528200425421?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWJjamVubmlmZXI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

其中，每个GMM有一些参数，就是我们要train的输出概率参数

![](http://img.blog.csdn.net/20140528200531906?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvYWJjamVubmlmZXI=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

# 语音识别过程
1. 将音频文件切分成等长（如20ms）的frames，对每frame提取特征（如MFCC）
1. 对每个frame运行GMM，得到每个frame属于每个状态的概率
1. 根据每个单词的HMM状态转移概率计算每个状态生成该frame的概率
1. 哪个词的HMM序列跑出来概率最大，就判断这段语音属于改词