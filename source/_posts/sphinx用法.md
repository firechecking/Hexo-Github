---
title: sphinx用法
date: 2017-08-18 13:53:07
tags: [Python,Sphinx]
categories: [工具,文档管理]
comments: true
---

# 介绍
Sphinx使用Python编写，最初为Python语言创建文档。可以使用sphinx编写文档、说明、书籍等。
<!--more-->

# 安装
1. 安装python
1. 安装sphinx

<http://jwch.sdut.edu.cn/book/tool/UseSphinx.html>

		easy_install sphinx

# 使用
1. 初始化：运行`sphinx-quickstart`(另外可以使用[sphinx-apidoc](http://www.sphinx-doc.org/en/stable/invocation.html#invocation-apidoc))，按要求选择，也可以全部默认
1. 修改主题
	1. 在conf.py文件中修改`html_theme = 'classic'`
1. 编译
	1. 在项目根目录下执行`make html`
1. [加入markdown格式支持](http://docs.readthedocs.io/en/latest/getting_started.html#in-markdown)
1. 托管到[read the docs](https://www.baidu.com/link?url=6qIB6tnnO4bbTEy4OgymNIltJgX2HpupdFILhOcZHKv85NSazUWlVShzo-wJL6nP&wd=&eqid=8b048f830002cf830000000659968d47)