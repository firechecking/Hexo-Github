---
title: SRILM学习
date: 2017-02-22 16:46:43
tags: [机器学习,语音识别,语言模型]
categories: [机器学习,语音识别,语言模型]
comments: true
---
# 参考文章
* <http://blog.csdn.net/zhoubl668/article/details/8365716>

# SRILM简介
SRILM主要用来构建和应用统计语言模型，主要用于语音识别，统计标注和切分，以及机器翻译，它主要包含以下几个部分:<!--more-->
* 一组实现的语言模型、支持这些模型的数据结构和各种有用的函数的C++类库；
* 一组建立在这些类库基础上的用于执行标准任务的可执行程序，如训练语言模型，在数据集上对这些语言模型进行测试，对文本进行标注或切分等任务。
* 一组使相关任务变得容易的各种脚本。

SRILM的主要目标是支持`语言模型的估计和评测`。

* 估计是从训练数据（训练集）中得到一个模型，包括最大似然估计及相应的平滑算法；
	* [最大似然估计](http://blog.csdn.net/yanqingan/article/details/6125812)
	
		已知某个随机样本满足某种概率分布，但是其中具体的参数不清楚，参数估计就是通过若干次试验，观察其结果，利用结果推出参数的大概值。最大似然估计是建立在这样的思想上：已知某个参数能使这个样本出现的概率最大，就把这个参数作为估计的真实值。

		求最大似然函数估计值的一般步骤： 
 		* 写出似然函数(给定输出x时，关于参数θ的似然函数等于给定参数θ后变量X的概率)
		* 对似然函数取对数，并整理
		* 求导数
		* 解似然方程
		
* 评测则是从测试集中计算其困惑度（MIT自然语言处理概率语言模型有相关介绍）。
	* 迷惑度/困惑度/混乱度（preplexity）
	
		其基本思想是给测试集的句子赋予较高概率值的语言模型较好。当语言模型训练完之后，测试集中的句子都是正常的句子，那么训练好的模型就是在测试集上的概率越高越好。困惑度越小，句子概率越大，语言模型越好。

其最基础和最核心的模块是n-gram模块，这也是最早实现的模块，包括两个工具：ngram-count和ngram，相应的被用来估计语言模型和计算语言模型的困惑度。

# [SRILM使用](http://www.tuicool.com/articles/Mjuiym)

## 分词与预处理

## 小数据

假设有去除特殊符号的训练文本trainfile.txt，以及测试文本testfile.txt，那么训练一个语言模型以及对其进行评测的步骤如下：

1. 词频统计

		ngram-count -text trainfile.txt -order 3 -write trainfile.count
	其中-order 3为3-gram，trainfile.count为统计词频的文本

2. 模型训练

		ngram-count -read trainfile.count -order 3 -lm trainfile.lm  -interpolate -kndiscount
	其中trainfile.lm为生成的语言模型，-interpolate和-kndiscount为插值与折回参数

3.	测试（困惑度计算）

		ngram -ppl testfile.txt -order 3 -lm trainfile.lm -debug 2 > file.ppl
	其中testfile.txt为测试文本，-debug 2为对每一行进行困惑度计算，类似还有-debug 0 , -debug 1, -debug 3等，最后  将困惑度的结果输出到file.ppl。

## 大数据（BigLM）

对于大文本的语言模型训练不能使用上面的方法，主要思想是将文本切分，分别计算，然后合并。步骤如下：

1. 切分数据

		split -l 10000 trainfile.txt filedir/
	即每10000行数据为一个新文本存到filedir目录下。

1. 对每个文本统计词频

		make-bath-counts filepath.txt 1 cat ./counts -order 3
	其中filepath.txt为切分文件的全路径，可以用命令实现：
	
		ls $(echo $PWD)/* > filepath.txt
	将统计的词频结果存放在counts目录下

1. 合并counts文本并压缩

		merge-batch-counts ./counts

1. 训练语言模型

		make-big-lm -read ../counts/*.ngrams.gz -lm ../split.lm -order 3
	用法同ngram-counts

1. 测评（计算困惑度）

		ngram -ppl filepath.txt -order 3 -lm split.lm -debug 2 > file.ppl