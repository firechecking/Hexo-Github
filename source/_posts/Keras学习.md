---
title: Keras学习
date: 2017-05-03 10:22:35
tags: [机器学习,Keras]
categories: [机器学习,Keras]
comments: true
---

# Keras介绍

## Keras设计原则
1. 用户友好
1. 模块性：网络层、损失函数、优化器、初始化策略、激活函数、正则化方法都是独立的模块，你可以使用它们来构建自己的模型。
1. 易扩展性：添加新模块超级容易，只需要仿照现有的模块编写新的类或函数即可。
1. 与Python协作
<!-- more -->

## Keras快速上手
1. Keras核心数据结构是**模型**，模型是一种组织网络层的方式，主要模型是Sequential模型，Sequential是一系列网络层按顺序构成的栈
1. 在Sequential中，通过.add()将网络层堆叠起来，就构成一个模型
1. 完成模型搭建后，使用.compile()来编译模型
	1. 编译模型时，必须指明损失函数和优化器，也可以自己定制损失函数。Keras中，用户可以根据自己的需要定制自己的模型、网络层、甚至修改源代码
	
```
from keras.models import Sequential
from keras.layers import Dense,Activation

model = Sequential()
model.add(Dense(units=64,input_dim=100))
model.add(Activation("relu"))
model.add(Dense(units=10))
model.add(Activation("softmax"))
model.compile(loss='categorical_crossentropy',optimizer='sgd',metrics=['accuracy'])
```
1. 完成编译后，调用.fit()进行模型训练。也可以手动调用一个个batch进行训练
1. 随后使用一行代码对模型进行评估
1. 使用.predict()对新数据进行预测

```
model.fit(x_train,y_train,epochs=5,batch_size=32)
#model.train_on_batch(x_batch,y_batch)
loss_and_metrics = model.evaluate(x_test,y_test,batch_size=128)
classes = model.predict(x_test,batch_size=128)
```
## Keras中的一些概念
### Keras后端
Keras底层库使用Theano或TensorFlow，Theano和TensorFlow都是“符号式”的库。符号这一的计算首先定义各种变量，然后建立一个“计算图”，计算图规定了各种变量之间的计算关系，建立好的计算图需要编译以确定其内部细节，此时的计算图还是一个“空壳子”，没有任何实际数据，当把需要运算的输入放进去后，才能在整个模型中形成数据流，从而形成输出值。
### tensor（张量）
张量可以看做向量、矩阵的自然推广。张量的阶数也称为维度或轴。

1. 规模最小的张量是0阶张量，及标量，就是一个数
1. 一些数有序排列，就形成1阶张量，就是一个向量
1. 2阶张量，就是一个矩阵
1. 3阶张量，就是一个立方体

### Sequential模型
1. Keras中，Model接收一个或一些张量作为输入，然后输出一个或一些张量，最特殊的模型是Sequential序贯模型。

### batch
1. 深度学习的优化算法就是应用梯度下降，每次参数更新有两种方式
	1. 遍历全部数据集算一次损失函数，然后算函数对各个参数的梯度，更新梯度。成为batch gradient descent，批梯度下降。
	1. 每看一个数据就算一下损失函数，然后求梯度更新参数，称为随机梯度下降（stochastic gradient descent，SGD）。
	1. 一种折中方法，mini-batch gradient descent，小批梯度下降，这种方法把数据分为若干批，按批更新参数，这样，一组数据共同决定了本次梯度的方向，下降起来就不容易跑偏，减少了随机性。另一方面批的样本数与整个数据集相比小了很多，计算量也不是很大。
	1. Keras的batch_size就是指一次训练的数据大小

### epochs
epochs指的就是训练过程中数据将被重复使用多少次。

# Keras使用
## Sequential序贯模型
1. 序贯模型是多个网络层的线性堆叠
1. 可以向序贯模型传递一个layer的list，也可以调用.add()方法将layer一个个加入模型。

### 输入数据shape
1. Sequential第一层需要接受一个输入数据shape的参数，有以下机种方式制定第一层shape
	1. 传递tuple类型的数据input_shape给第一层
	1. 如Dense层，支持input_dim指定输入数据shape，一些3D层支持input_dim和input_length指定输入shape

### 编译
1. 在训练模型之前，我们需要通过compile来对学习过程进行配置，compile接受三个参数：
	1. 优化器optimizer：可指定已预定义的优化器名，如`rmsprop`、`adagrad`，或一个Optimizer类的对象
	1. 损失函数loss：模型试图最小化的目标函数，可制定预定义的损失函数名，如`categorical_crossentropy`、`mse`，也可以为一个损失函数
	1. 指标列表metrics：对分类问题一般设置为`metrics=['accuracy']`。指标可以是一个预定义指标的名字，也可以是用户定制的函数。指标函数应返回单个张量或一个完成`metric_name->metric_value`映射的字典
		1. 性能评估模块提供一些列用于模型性能评估的函数
		1. 性能评估函数类似目标函数，只不过性能评估结果不会用于训练
		1. 性能评估函数输入参数：
			1. y_true:真实标签,theano/tensorflow张量
			1. y_pred:预测值, 与y_true形式相同的theano/tensorflow张量
		1. 性能评估函数返回值：单个用以代表输出各个数据点上均值的值

### 训练
1. Keras以Numpy数组作为输入数据和标签的数据类型，训练模型一般使用.fit()函数。

## Functional模型
1. Keras函数式模型接口，是用户定义多出输出模型、非循环有向模型或具有共享层的模型等复杂模型的途径。
1. 函数式模型是最广泛的一类模型，序贯模型（Sequential）是函数式模型的特殊情况

### 全连接网络
### 多输入和多输出模型
1. 使用如下模型进行新闻转发和点赞次数预测。主要输入问新闻本身，额外输入是新闻发布日期等，主损失函数评估基于新闻和额外信息的预测情况，辅助损失函数评估基于新闻本身做出的预测。

![](http://keras-cn.readthedocs.io/en/latest/images/multi-input-multi-output-graph.png)

代码如下：

```
from keras.layers import Input,Embedding,LSTM,Dense
from keras.models import Model

#主输入接受新闻本身，是一个整数序列，采用one-hot编码，序列长度100（每句话100个单词）
main_input = Input(shape=(100,),dtype="int32",name='main_input')

# Embedding嵌入层只能作为模型的第一层，用于对one-hot表示的稀疏矩阵进行压缩编码
x = Embedding(output_dim=512,input_dim=10000,input_length=100)(main_input)

lstm_out = LSTM(32)(x)

# 插入一个额外损失
auxiliary_output = Dense(1,activation='sigmoid',name='aux_output')(lstm_out)

# 然后将LSTM与额外输入数据串联起来组成输入
auxiliary_input= Input(shape=(5,),name='aux_input')
x = keras.layers.concatenate([lstm_out,auxiliary_input])
x = Dense(64,activation='relu')(x)
x = Dense(64,activation='relu')(x)
x = Dense(64,activation='relu')(x)

main_output = Dense(1,activation='sigmoid',name='main_output')(x)

# 最后定义整个2输入，2输出模型
model = Model(inputs=[main_input,auxiliary_input],outputs=[main_output,auxiliary_output]

# 这里将binary_crossentropy损失函数应用于所有输出，并给额外损失赋0.2权重，然后编译模型
model.compile(optimizer='rmsprop',loss='binary_crossentropy',loss_weights=[1.,0.2])
```