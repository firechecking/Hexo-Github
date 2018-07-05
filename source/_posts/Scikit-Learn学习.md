---
title: Scikit-Learn学习
date: 2017-03-31 16:25:05
tags: [机器学习,scikit-learn]
categories: [机器学习,scikit-learn]
comments: true
---
# 参考文档
1. <http://scikit-learn.org/stable/tutorial/basic/tutorial.html>
1. <http://www.jianshu.com/p/516f009c0875>
<!-- more -->

---
# 安装
1. 安装Anaconda
2. conda install scikit-learn

---
# 机器学习分类
## 监督学习（supervised learning）
1. 分类（classification）：离散预测
1. 回归（regression）：连续预测

## 监督学习（unsupervised learning）
1. 聚类（clustering）
1. 密度估计（density estimation）

---
# 基础
## 模型持久保存： pickle/joblib
```
from sklearn import svm
from sklearn import datasets
clf = svm.SVC()
iris = datasets.load_iris()
X, y = iris.data, iris.target
clf.fit(X, y)  
import pickle
s = pickle.dumps(clf)
clf2 = pickle.loads(s)
clf2.predict(X[0:1])
```

joblib为sklearn内置发放，效率比pickle更高，但只支持导出到文件，不支持导出到字符串

---
# Examples
## 数字识别
```
from sklearn import datasets
from sklearn import svm
if __name__ == "__main__":
	digits = datasets.load_digits()
	Iclf = svm.SVC(gamma=0.001,C=100.)
	clf.fit(digits.data[:-1],digits.target[:-1])
	print clf.predict(digits.data[-1:])
```
