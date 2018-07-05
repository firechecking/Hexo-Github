---
title: Pyke规则引擎学习
date: 2017-06-21 16:14:08
tags: [Python,Pyke,规则引擎]
categories: [Python,Pyke]
comments: true
---

# 简介
<!-- more -->

# 安装
1. <http://sourceforge.net/projects/pyke/files/>中下载pyle-<release>.zip
1. 解压后运行python setup.py build
1. 运行python setup.py install

# 组成

Pyke主要包括事实库、规则库

1. 事实库陈述事实，文件后缀为kfb
1. 规则库后缀为krb，分类正向推理和反向推理
	1. 正向推理，格式为foreach...assert...，简单理解为遍历事实库，对满足foreach下的内容，运行assert下的内容
	2. 反向推理，格式为use..when..with，简单理解为use中定义了调用接口，when中判断接口是否满足某一条件，with中为满足when中条件后执行的内容。

# 使用