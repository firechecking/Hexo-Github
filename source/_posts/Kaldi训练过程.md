---
title: Kaldi训练过程
date: 2017-02-21 12:02:03
tags: [机器学习,Kaldi,语音识别]
categories: [机器学习,语音识别,Kaldi]
comments: true
---
# thchs30 <!--more-->
## 数据下载
1. data_thchs30

	目录结构如下：
	
	```
	data
		*.wav(录音文件)
		*.wav.trn(标注内容，包括词(中文)、字(拼音)、音素)
	{train,dev,test}
		通过软链接到data目录,将data中的数据拆分为训练、dev?、测试数据
	lm_word
		lexicon.txt(以词为单位的发音词典)
		word.3gram.lm(lexicon中词的1-gram、2-gram、3-gram概率，LM是输出的语言模型文件)
	lm_phone
		lexicon.txt(以音素为单位的词典)
		phone.3gram.lm(lexicon中音素的1-gram、2-gram、3-gram概率，LM是输出的语言模型文件)
	```
	数据统计
	
	|数据集|音频长度|句子数量|词数量|
	|----|-------|------|-----|
	|train|  25h  | 10000|198252|
	| dev | 2:14  |  893 |17743|
	| test| 6:15  | 2495 |49085|

1. resoureces
	
	目录结构如下：

	```
	dict:训练数据中词语的字典
	noise:不同场景下的背景噪音音频文件
	```
1. test-noise：噪音文件

## 数据准备(local/thchs-30_data_prep.sh)
1. 创建目录data/train、data/dev、data/test目录
1. 在train、dev、test目录，根据原始数据中的音频文件，分别生成以下文件：
	* wav.scp：文件ID与文件路径的对应表
	* utt2spk：文件ID与发音人的对应表
	* word.txt：文件ID与文件中文词汇标注对应表
	* phone.txt：文件ID与文件中音素标注对应表
	* text：word.txt的复制
1. 根据utt2spk生成spk2utt(utils/utt2spk_to_spk2utt.pl)
	* spk2utt：发言人与文件ID的对应表
1. 复制一份data/test为data/test_phone
1. 在test_phone中，修改文件text为phone.txt的复制

## 生成{% post_link MFCC学习 MFCC模型%}(steps/make_mfcc.sh)
1. 创建目录data/mfcc，并将data/train,data/dev,data/test复制到data/mfcc
1. 调用steps/make_mfcc.sh生成mfcc模型。日志导出到exp/make_mfcc/{train,dev,test}目录，生成的mfcc结果保存到mfcc/{train,dev,test}目录
	* data/mfcc/{train,dev,test}/feats.scp:文件ID与MFCC特征文件对应表
	* mfcc/{train,dev,test}/raw\_mfcc_{train,dev,test}.{1,2,3..n}.ark：MFCC特征文件(n为并行处理的job数)
	* mfcc/{train,dev,test}/raw\_mfcc_{train,dev,test}.{1,2,3..n}.scp：文件ID与MFCC特征文件对应表（feats.scp由这些文件合并生成）
1. 计算CMVN(倒谱均值和方差归一化)，日志及生成路径与mfcc相同
	* data/mfcc/{train,dev,test}/cmvn.scp:发音人与CMVN特征对应表	
	* mfcc/{train,dev,test}/cmvn_train.ark	:针对每个发音人计算得到的CMVN特征
	* mfcc/{train,dev,test}/cmvn_train.scp	:发音人与CMVN特征对应表
1. 将data/mfcc/test/feats.scp,data/mfcc/test/cmvn.scp复制到data/mfcc/test_phone目录

## 构建词典
1. 词
	1. 创建目录data/{dict,lang,graph}
	1. 将下载的resource/dict下的部分文件复制到data/dict
	1. 将resource/dict/lexicon.txt与data_thchs30/lm_word/lexicon.txt合并，并将结果保存到data/dict/lexicon.txt
	1. 生成标准的FST模型(utils/prepare_lang.sh)
		* 根据lexicon.txt(或包含概率的lexiconp.txt)、silence_phones.txt、nonsilence_phones.txt、optional_silence.txt、extra_questions.txt生成data/lang目录下的文件	
	1. 将ARPA格式的语言模型转为FST模型
		1. 将data/lang/目录下的文件复制到data/graph目录
		1. 将ARPA模型转为FST模型，保存到data/graph/G.fst
1. 音素
	1. 创建目录data/{dict,lang,graph}_phone
	1. 将下载的resource/dict下的部分文件复制到data/dict_phone
	1. 将data_thchs30/lm_phone/lexicon.txt进行过滤和排序后，保存到data/dict_phone/lexicon.txt
	1. 生成标准的FST模型(utils/prepare_lang.sh)
		* 同上（词）
	1. 将ARPA格式的语言模型转为FST模型
		* 同上（词）

## monophone单音素
### 参考文章
* <http://blog.csdn.net/chinabing/article/details/42501573?locationNum=12>
* <http://blog.csdn.net/duishengchen/article/details/52575926>
* <http://wenku.baidu.com/link?url=Gu0IqalEZEmTVcCGmy6V3b81x_Um3b_LoMg9KYrQuVrUqmejCL5TEY9D32TypO-WGecvGunaiYD_4_L--XfE6baxw0xLndu3AbkE_PBElHu>
* <http://blog.csdn.net/visionfans/article/details/50255917>
* <http://blog.csdn.net/anymake_ren/article/details/52275412>

### MonoPhone and Triphone
`Monophone`: The pronunciation of a word can be given as a series symbols that correspond to the individual units of sound that make up a word.  These are called 'phonemes' or 'phones'.  A monophone refers to a single phone.

`Triphone`: A triphone is simply a group of 3 phones in the form "L-X+R" - where the "L" phone  (i.e. the left-hand phone) precedes "X" phone and the "R" phone (i.e. the right-hand phone) follows it.  

Below is an example of the conversion of a monophone declaration of the word "TRANSLATE" to a triphone declaration (the first line shows the "monophone" declaration, and the second line shows the "triphone" declaration):

```
TRANSLATE [TRANSLATE] t r @ n s l e t
TRANSLATE [TRANSLATE] t+r t-r+@ r-@+n @-n+s n-s+l s-l+e l-e+t e-t
```

### 训练(steps/train_mono.sh)
对语音数据进行分帧和提取特征以后，语音标注是对一整段话进行标注而没有具体到某一帧，但训练系统需要有每一帧语音的具体对应标注。

kaldi训练monophone脚本的过程，首先根据标注数目对特征序列进行等间隔切分，例如一个具有5个标注的长度为100帧的特征序列，则认为1-20帧属于第1个标注，21-40属于第2个，然后在训练模型的过程中会不断地重新对齐。
训练单音素的基础HMM模型，迭代40次，并按照realign_iters的次数对数据对齐

```
# Begin configuration section.
nj=4
cmd=run.pl
scale_opts="--transition-scale=1.0 --acoustic-scale=0.1 --self-loop-scale=0.1"
num_iters=40    # Number of iterations of training
max_iter_inc=30 # Last iter to increase #Gauss on.
totgauss=1000 # Target #Gaussians.
careful=false
boost_silence=1.0 # Factor by which to boost silence likelihoods in alignment
realign_iters="1 2 3 4 5 6 7 8 9 10 12 14 16 18 20 23 26 29 32 35 38";
config= # name of config file.
stage=-4
power=0.25 # exponent to determine number of gaussians from occurrence counts
norm_vars=false # deprecated, prefer --cmvn-opts "--norm-vars=false"
cmvn_opts=  # can be used to add extra options to cmvn.
# End configuration section.
```

### 测试(steps/thchs-30_decode.sh)
thchs-30_decode.sh测试单音素模型，实际使用mkgraph.sh建立完全的识别网络，并输出一个有限状态转换器，最后使用decode.sh以语言模型和测试数据为输入计算WER.

### 对齐(steps/align_si.sh)
align_si.sh用指定模型对指定数据进行对齐，一般在训练新模型前进行，以上一版本模型作为输入，输出在<align-dir>

## Triphone三因素
以单音素模型为输入训练上下文相关的三音素模型
## trib2进行lda_mllt特征变换
对特征使用LDA和MLLT进行变换，训练加入LDA和MLLT的三音素模型。
LDA+MLLT refers to the way we transform the features after computing the MFCCs: we splice across several frames, reduce the dimension (to 40 by default) using Linear Discriminant Analysis), and then later estimate, over multiple iterations, a diagonalizing transform known as MLLT or CTC.
## trib3进行sat自然语言适应
运用基于特征空间的最大似然线性回归（fMLLR）进行说话人自适应训练
This does Speaker Adapted Training (SAT), i.e. train on fMLLR-adapted features.  It can be done on top of either LDA+MLLT, or delta and delta-delta features.  If there are no transforms supplied in the alignment directory, it will estimate transforms itself before building the tree (and in any case, it estimates transforms a number of times during training).

decode_fmllr.sh ：对做了发音人自适应的模型进行解码
Decoding script that does fMLLR.  This can be on top of delta+delta-delta, or LDA+MLLT features.

## trib4做quick
Train a model on top of existing features (no feature-space learning of any kind is done).  This script initializes the model (i.e., the GMMs) from the previous system's model.That is: for each state in the current model (after tree building), it chooses the closes state in the old model, judging the similarities based on overlap of counts in the tree stats.
## dnn