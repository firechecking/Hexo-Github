---
title: Kaldi关键词识别KWS学习
date: 2017-03-01 17:13:57
tags: [机器学习,Kaldi,语音识别,KWS,关键词识别]
categories: [机器学习,语音识别,Kaldi]
comments: true
---

# 参考文档
* <http://www.kaldi-asr.org/doc/kws.html>

# Introduction
1. Kaldi中的关键词识别包括以下两部分：
	1. 关键词的快速检索
	1. 字典以外(OOV, out-of-vocabulary)的关键词处理<!--more-->
1. 在Kaldi中，关键词识别(KWS)和大词汇量连续识别(LVCSR)都是基于权重有限状态转换(WFST)，因此关键词识别不光能对词进行识别，也能对音节、音素等进行识别
1. 在kaldi官方文档中，KWS介绍内容包括以下三部分：
	1. 一个最基本、最典型的KWS系统组成部分
	1. 如何使用代理关建词(proxy keywords)来处理字典以外的关键词
	1. 位于egs/babel/下，和KWS相关的脚本

# Typical Kaldi KWS System
## 示例
* [Quantifying the Value of Pronunciation Lexicons for Keyword Search in Low Resource Languages", G. Chen, S. Khudanpur, D. Povey, J. Trmal, D. Yarowsky and O. Yilmaz](http://www.clsp.jhu.edu/~guoguo/papers/icassp2013_lexicon_value.pdf)
	
## KWS系统组成
1. LVCSR：对待查询的关键词集合进行解码，并生成相应的lattice
1. KWS：对lattice进行index，并根据生成的index来查找关键词

### LVCSR
1. 基本的LVCSR系统是一个SGMM(子空间混合高斯模型)+MMI(最大互信息)系统，使用PLP([线性预测分析](http://blog.csdn.net/lovewubo/article/details/37937377))获取13位的声学特征
1. 然后采用最大似然声学训练：以flat-start的上下文独立音素HMM模型进行初始化，生成发音人自适应的triphone HMM-GMM模型。
1. 接下来使用不同的发音人训练数据，训练处一个通用模型，然后将模型用于训练HMM发射概率的子空间高斯混合模型
1. 最后，所有的训练语音由SGMM进行编码，并使用BMMI来训练SGMM模型参数，详细的执行脚本：`egs/babel/s5b/run-1-main.sh`
1. 除SGMM+MMI系统之外，还可以使用如DNN(`egs/babel/s5b/run-2a-nnet-gpu.sh`)、 bottleneck feature system(`egs/babel/s5b/run-8a-kaldi-bnf.sh`)等
1. 以上系统的目的都是为同样的搜索集编码并产生lattice，然后再进入KWS模块进行index和search
1. Lattices
	1. LVCSR产生lattices采用论文中的方法["Lattice indexing for spoken term detection", D. Can, M. Saraclar, Audio, Speech, and Language Processing](https://wiki.inf.ed.ac.uk/twiki/pub/CSTR/ListenSemester2201314/taslp_2011.pdf)
	1. 搜索集的lattice由WFST转换成一个factor transducer，记录了开始时间、结束时间、后验概率。
	1. 上文生成的factor transducer是lattice中所有词序列的倒置索引（inverted index）。当给定一个关键词时，建立一个简单的有限状态机，其接受关键词作为输入，并使用factor transducer对关键词进行处理，来表示关键词在搜索集的所有查找信息，并包括每次发现的录音文件id、开始、结束时间，lattice后验概率
	1. 这些查找信息按照后验概率进行保存，并使用"Rapid and Accurate Spoken Term Detection"中的方法进行YES/NO判断

# Proxy keywords
## 参考文献
["Using Proxies for OOV Keywords in the Keyword Search Task", G. Chen, O. Yilmaz, J. Trmal, D. Povey, S. Khudanpur](http://www.clsp.jhu.edu/~guoguo/papers/asru2013_proxy_keyword.pdf)

## 目的
1. 解决lattice中的OOV(未收录词)问题。如果一个关键词没有在LVCSR的词汇表中，即使在搜索集录音文件中有这个关键词的发音，它也不会出现在查找lattice中。因此需要有方法来解决OOV的关键词查找问题。

## 方法
1. Kaldi的KWS使用的方法是，在词汇表中找到一系列和关键词发音近似的词，以这些词作为关键词的代理词汇
1. “Low-Resource Open Vocabulary Keyword Search Using Point Process Models”
1. Proxy keyword方法是模糊搜索方法，不仅可以用来解决OOV关键词问题，还能用于改善IV(in-vocabulary)关键词问题
1. 代理词产生公式如下：
$$K^{'}=Project(ShortestPath(prune(prune(K\cdot L\_2\cdot E^{'})\cdot L\_{1}^{-1})))$$
其中$K$是原始关键词，$L\_2$是一个包含$K$发音的字典，$E^{'}$是编辑距离转换器，包括由训练集计算得到的发音困惑度。$L\_1$是原始字典。$K^{'}$是包含一系列与$K$发音相似IV词的WFST，然后将这些相似词当做关键词进行查找
1. 在上式中的prune(剪枝)是必须的，特别是在词汇量很大的时候。

# Babel Scripts
## 使用Bebel中的脚本进行
1. 进入目录egs/babel/s5b/
1. 安装F4DE，并添加到path.sh
1. 修改cmd.sh，使cmd.sh指向自己的数据集
1. 将conf/languages目录下的某一个文件指向./lang.conf。如：
	```
	ln -s conf/languages/105-turkish-limitedLP.official.conf lang.conf
	```
1. 修改./lang.conf指向自己的数据
1. 运行run-1-main.sh来建立LVCSR系统
1. 运行run-2-segmentation.sh来产生评估数据
1. 运行run-4-anydecode.sh，对评估数据进行解码，产生index并搜索关键词

## 其他
1. 类似的可以建立DNN系统，BNF系统，Semi-supervised系统等。
1. KWS内容在run-4-anydecode.sh中，下文将对关键词搜索进行详细说明。工作前提是已经对搜索集进行解码，并产生了lattices（run-1-main.sh）

# Prepare KWS data
1. 如果有一个搜索集dev10h.uem，则会在相应目录建立data/dev10h.uem/kws
1. 建立data/dev10h.uem/kws中的数据，需要获取或生成以下三个文件：
	1. ecf文件：包含搜索集信息
	1. kwlist文件：列出所有关键词
	1. rttm文件：评分文件（rttm文件的产生可以通过使用一个训练好的模型来强制对齐搜索集）

## 文件格式
1. ECF

	```
	<ecf source_signal_duration="483.825" language="" version="Excluded noscore regions">
	  <excerpt audio_filename="YOUR_AUDIO_FILENAME" channel="1" tbeg="0.000" dur="483.825" source_type="splitcts"/>
	</ecf>
	```

1. KWLIST

	```
	<kwlist ecf_filename="ecf.xml" language="tamil" encoding="UTF-8" compareNormalize="" version="Example keywords">
	  <kw kwid="KW204-00001">
	    <kwtext>செய்றத</kwtext>
	  </kw>
	  <kw kwid="KW204-00002">
	    <kwtext>சொல்லுவியா</kwtext>
	  </kw>
	</kwlist>
	```

1. RTTM

	```
	SPEAKER YOUR_AUDIO_FILENAME 1 5.87 0.370 <NA> <NA> spkr1 <NA>
	LEXEME YOUR_AUDIO_FILENAME 1 5.87 0.370 ஹலோ lex spkr1 0.5
	SPEAKER YOUR_AUDIO_FILENAME 1 8.78 2.380 <NA> <NA> spkr1 <NA>
	LEXEME YOUR_AUDIO_FILENAME 1 8.78 0.300 உம்ம் lex spkr1 0.5
	LEXEME YOUR_AUDIO_FILENAME 1 9.08 0.480 அதான் lex spkr1 0.5
	LEXEME YOUR_AUDIO_FILENAME 1 9.56 0.510 சரியான lex spkr1 0.5
	LEXEME YOUR_AUDIO_FILENAME 1 10.07 0.560 மெசேஜ்டா lex spkr1 0.5
	LEXEME YOUR_AUDIO_FILENAME 1 10.63 0.350 சான்ஸே lex spkr1 0.5
	LEXEME YOUR_AUDIO_FILENAME 1 10.98 0.180 இல்லயே lex spkr1 0.5
	```

## 运行
1. 数据准备
	* 如果进行基础的KWS，只需要提供以上三个文件，以及语言模型，音频文件，并运行kws_setup.sh
	
	```
	local/kws_setup.sh \
  --case_insensitive $case_insensitive \
  --rttm-file $my_rttm_file \
  $my_ecf_file $my_kwlist_file data/lang $dataset_dir
	```
	* 如果需要对OOV关键词进行模糊查找，需要首先得到发音困惑度，并训练一个G2P模型。

1. Indexing and searching
	* 运行前提：已经对训练集进行解码；并且产生了lattices
	* 运行以下脚本完成Indexing和searching
		
		```
		local/kws_search.sh --cmd "$cmd" \
  	--max-states ${max_states} --min-lmwt ${min_lmwt} \
  	--max-lmwt ${max_lmwt} --skip-scoring $skip_scoring \
  	--indices-dir $decode_dir/kws_indices $lang_dir $data_dir $decode_dir
		```