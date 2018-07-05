---
title: Kaldi中的决策树
date: 2017-03-13 16:03:53
tags: [机器学习,Kaldi]
categories: [机器学习,语音识别,Kaldi]
comments: true
---

# 参考链接
1. <http://www.kaldi-asr.org/doc/tree_externals.html>
1. <http://www.kaldi-asr.org/doc/tree_internals.html>
1. [HTK语音识别中的决策树](http://blog.sina.com.cn/s/blog_6df8f2620102v0gb.html)
1. [决策树在Kaldi中如何使用](http://blog.csdn.net/chenhoujiangsir/article/details/51613144?from=singlemessage&isappinstalled=1)<!--more-->

# 为什么使用决策树
1. Kaldi在语音识别时，最小识别单位是音素。在单音素(monophone)识别时，不考虑音素所处的位置，因此单音素模型不能反映音素在不同位置时不同的发音情况，因此还需要使用三音素(triphones)模型。
1. 在使用三音素时，如果但音素总数为40，则三音素最多有40x40x40种情况，但其中只有很少一部分音素在单音素和三音素中具有不同发音，因此为了更好的利用训练数据，需要对三音素模型进行决策，将三音素模型中发音相似的聚类在一起。这个工作就由使决策树来完成（`注意`:在建立决策树时，是对每个音素的每个状态都建立一个决策树，而不是只对某一个音素来建立。这里，我们以音素ih的首状态为例，详细说明决策树的建立过程。）

![](http://s6.sinaimg.cn/mw690/0020RyYWzy6Lt1uZVVH75&690)