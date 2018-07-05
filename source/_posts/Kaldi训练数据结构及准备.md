---
title: Kaldi训练数据结构及准备
date: 2017-03-02 21:09:55
tags: [机器学习,Kaldi]
categories: [机器学习,语音识别,Kaldi]
comments: true
---
# 参考链接
1. <http://www.kaldi-asr.org/doc/data_prep.html>

# 简介
1. 文中内容均以最新s5脚本为例
1. s5/local目录通常适合数据库相关的脚本
1. 除了网上下载语言模型外，local目录中还有很多和本地训练语言模型相关的脚本
1. 数据准备阶段主要涉及两部分：
	1. 数据(data/train/):包括收集到的录音文件
	1. 语言(data/lang/):和语言相关的内容，包括词典、发音音素等。<!--more-->

# data Part
## text
包含每个句子的标注文本

```
s5# head -3 data/train/text
sw02001-A_000098-001156 HI UM YEAH I'D LIKE TO TALK ABOUT HOW YOU DRESS FOR WORK AND
sw02001-A_001980-002131 UM-HUM
sw02001-A_002736-002893 AND IS
```
1. 每行第一部分是句子ID(utterance-id)，如果有不同发音人，为方便排序，发音人ID(speaker-id)作为句子ID前缀
1. 第二部分是每个句子的标注文本，对于字典意外的词，会自动在data/lang/oov.txt中记录

## wav.scp
wav.scp举例内容如下：

```
s5# head -3 data/train/wav.scp
sw02001-A /home/dpovey/kaldi-trunk/tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 1 /export/corpora3/LDC/LDC97S62/swb1/sw02001.sph |
sw02001-B /home/dpovey/kaldi-trunk/tools/sph2pipe_v2.5/sph2pipe -f wav -p -c 2 /export/corpora3/LDC/LDC97S62/swb1/sw02001.sph |
```

文件的固定格式：

```
<recording-id> <extended-filename>
```
1. extended-filename：可以指向一个真实文件，也可以是一条可以生成wav文件的命令（注意最后的|为linux管道）
1. recording-id指录音文件id，如果没有segments文件，也就是说一个录音文件对于一句话，那么recording-id也就是utterance-id
1. wav.scp指向的录音文件必须是单声道，多声道文件必须通过命令制定某一具体声道

## segments（非必需）
segments举例内容如下：

```
s5# head -3 data/train/segments
sw02001-A_000098-001156 sw02001-A 0.98 11.56
sw02001-A_001980-002131 sw02001-A 19.8 21.31
sw02001-A_002736-002893 sw02001-A 27.36 28.93
```
文件的固定格式：

```
<utterance-id> <recording-id> <segment-begin> <segment-end>
```
1. 非必须，标记录音文件中的句子位置，如果每个录音文件只有一句话，则不需要segments文件
1. 标记一句话在录音文件中的开始和结束为止，单位：秒

## reco2file_and_channel（非必需）
reco2file_and_channel只用来计算错误率评分，用到的工具是NIST的“sclite”

reco2file_and_channel举例内容如下：

```
s5# head -3 data/train/reco2file_and_channel
sw02001-A sw02001 A
sw02001-B sw02001 B
sw02005-A sw02005 A
```
文件的固定格式：

```
<recording-id> <filename> <recording-side (A or B)>
```
1. filename指.sph文件名(不包括后缀)(也可以指示stm文件中的内容)
1. recording-side特别用于电话录音等文件中存在双方的情况，如果只有一方说话，可以直接标记为A

## utt2spk:
utt2spk文件指明每一句话的发言人

utt2spk举例内容如下：

```
s5# head -3 data/train/utt2spk
sw02001-A_000098-001156 2001-A
sw02001-A_001980-002131 2001-A
sw02001-A_002736-002893 2001-A
```
文件的固定格式：

```
<utterance-id> <speaker-id>
```
1. speaker-id指示大致标注不同发音人，比如区分电话录音双方，并不要求特别准确
1. 在不清楚发音人情况下，可以使用让speaker-id和utterance-id相同

## 注意
1. 以上所有文件都必须进行排序，一边支持在管道等命令中的正确读取。
1. 在排序时应该定义变量LC_ALL为C，让排序按照C++字符串排序方式进行，否则会找出Kaldi运行出错
	
	```
	export LC_ALL=C
	```
1. 如果数据包括测试集，可以以stm或glm命名，相应的计算错误率的脚本位于local/score.sh

## 其它文件
以下文件都能根据上面提供的文件，使用Kaldi提供的命令生成

### spk2utt
1. spk2utt使用utt2spk_to_spk2utt.pl脚本，依据utt2spk文件生成

	```
	utils/utt2spk_to_spk2utt.pl data/train/utt2spk > data/train/spk2utt
	```

### feats.scp
feats.scp指示计算得到的句子MFCC特征，其内容太如下：

```
s5# head -3 data/train/feats.scp
sw02001-A_000098-001156 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:24
sw02001-A_001980-002131 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:54975
sw02001-A_002736-002893 /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:62762
```
文件的固定格式：

```
<utterance-id> <extended-filename-of-features>
```
1. 每个feature文件包括一个Kaldi能够识别的矩阵
1. 通常这个特征矩阵以10ms位单位，每个单位为13维特征向量
1. “/home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark:24”意思是在文件“/home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/raw_mfcc_train.1.ark”中调用fseek()定位到位置24，并读取当前数据
1. feats.scp文件由以下命令生成
	
	```
	steps/make_mfcc.sh --nj 20 --cmd "$train_cmd" data/train exp/make_mfcc/train $mfccdir
	```
$mfccdir是用户指定的用于生成ark文件的目录

### cmvn.scp:
cmvn.scp文件以speaker-id为index，包含统计得到的倒谱均值和方差归一化。每一个统计为一个2x15的矩阵。

	```
	s5# head -3 data/train/cmvn.scp
	2001-A /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:7
	2001-B /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:253
	2005-A /home/dpovey/kaldi-trunk/egs/swbd/s5/mfcc/cmvn_train.ark:499
	```
cmvn.scp文件通过以下命令生成：

```
steps/compute_cmvn_stats.sh data/train exp/make_mfcc/train $mfccdir
```

## 其它
1. 为防止数据准备错误，可以使用以下脚本检查数据格式是否正确

	```
	utils/validate_data_dir.sh data/train
	```
1. 以下脚本会检查并解决排序错误，并删除掉确实特征文件或标注文本的utterance

	```
	utils/fix_data_dir.sh data/train
	```
	

# lang Part
## phones.txt, words.txt
这两个文件都是OpenFst格式的表文件，每行包含一个符号和一个整数

```
s5# head -3 data/lang/phones.txt
<eps> 0
SIL 1
SIL_B 2
s5# head -3 data/lang/words.txt
<eps> 0
!SIL 1
-'S 2
```
这两个文件用于Kaldi在整数和符号之间进行对应，通常在utils/int2sym.pl和utils/sym2int.pl、以及OpenFst的fstcompile和fstprint中用到

## L.fst
1. L.fst是字典的FST存储格式。[ "Speech Recognition with Weighted Finite-State Transducers"](http://www.cs.nyu.edu/~mohri/pub/hbka.pdf)
1. L.fst表示的FST，接收音素作为输入，并得到词作为输出

## L_disambig.fst
1. L_disambig.fst和L.fst一样是字典的FST，另外还包括一些特殊符号用于消除词与词之间容易产生混淆的连接关系


## oov.txt
oov.txt只有一行，这一行表面，对于字典中没有收录的词，将会用这一行中的符号进行替代

```
s5# cat data/lang/oov.txt
<UNK>
```
1. UNK在字典必须只具有一个音素发音，通常为SPN(spoken noise)

```
s5# grep -w UNK data/local/dict/lexicon.txt
<UNK> SPN
```
## oov.int
oov.int中包含oov.txt中未收录词的整数符号

## topo
topo文件是用的HMM模型，内容如下：

```
s5# cat data/lang/topo
<Topology>
<TopologyEntry>
<ForPhones>
21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100 101 102 103 104 105 106 107 108 109 110 111 112 113 114 115 116 117 118 119 120 121 122 123 124 125 126 127 128 129 130 131 132 133 134 135 136 137 138 139 140 141 142 143 144 145 146 147 148 149 150 151 152 153 154 155 156 157 158 159 160 161 162 163 164 165 166 167 168 169 170 171 172 173 174 175 176 177 178 179 180 181 182 183 184 185 186 187 188
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.75 <Transition> 1 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.75 <Transition> 2 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 2 0.75 <Transition> 3 0.25 </State>
<State> 3 </State>
</TopologyEntry>
<TopologyEntry>
<ForPhones>
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20
</ForPhones>
<State> 0 <PdfClass> 0 <Transition> 0 0.25 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 </State>
<State> 1 <PdfClass> 1 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 2 <PdfClass> 2 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 3 <PdfClass> 3 <Transition> 1 0.25 <Transition> 2 0.25 <Transition> 3 0.25 <Transition> 4 0.25 </State>
<State> 4 <PdfClass> 4 <Transition> 4 0.75 <Transition> 5 0.25 </State>
<State> 5 </State>
</TopologyEntry>
</Topology>
```

## phones
data/lang/phone/目录中有多个和音素相关的文件，通常具有三个文件版本：
1. ".txt"

	```
	s5# head -3 data/lang/phones/context_indep.txt
	SIL
	SIL_B
	SIL_E
	```
2. ".int"

	```
	s5# head -3 data/lang/phones/context_indep.int
	1
	2
	3
	```
3. ".csl"
	
	```
	s5# cat data/lang/phones/context_indep.csl
	1:2:3:4:5:6:7:8:9:10:11:12:13:14:15:16:17:18:19:20
	```

这三种文件格式通常包含相同的信息，context_indep.txt包含一些不算真正发音的音素，如silence(SIL),spoken noise(SPN),non-spoken noise(NSN),laughter(LAU)

```
# cat data/lang/phones/context_indep.txt
SIL
SIL_B#词语开始处的SIL
SIL_E#词语结束处的SIL
SIL_I#词语中间处的SIL
SIL_S#独立的SIL
SPN
SPN_B
SPN_E
SPN_I
SPN_S
NSN
NSN_B
NSN_E
NSN_I
NSN_S
LAU
LAU_B
LAU_E
LAU_I
LAU_S
```
1. sets.txt

包含所有音素的集合

1. roots.txt

roots.txt中包含建立音素相关的决策树所需的信息，示例内容如下：

```
head data/lang/phones/roots.txt
shared split SIL SIL_B SIL_E SIL_I SIL_S
shared split SPN SPN_B SPN_E SPN_I SPN_S
shared split NSN NSN_B NSN_E NSN_I NSN_S
shared split LAU LAU_B LAU_E LAU_I LAU_S
...
shared split B_B B_E B_I B_S
```
同一样的shared split SIL SIL_B SIL_E SIL_I SIL_S表示SIL SIL_B SIL_E SIL_I SIL_S五个音素在决策树中具有同一个根节点

# 建立lang目录
```
utils/prepare_lang.sh data/local/dict "<UNK>" data/local/lang data/lang
```
1. 输入data/local/dict
1. "\<UNK>",替换OOV
1. data/local/lang/脚本需要用得到临时输出文件夹
1. data/lang/结果输出文件夹
1. 其中需要人工创建的是data/local/dict/目录下的内容