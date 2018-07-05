---
title: Hello World
mathjax: true
tags: [Hexo]
categories: [Hexo,使用教程]
comments: true
---

Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)<!--more-->

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

### 修改主题
1. 下载主题到Hexo目录/theme/[主题名称]
1. 编辑Hexo目录下的 _config.yml，将theme的值改为[主题名称]。

### 增加侧边栏
1. 修改theme目录的_config.yml文件，内容如下：

	```
	menu:
		home: /
		archives: /archives
		categories: /categories
		tags: /tags
		about: /about
	```

2. 对应上一步的路径，Hexo目录命令行下输入

	```
	hexo new page categories|tags|about
	```
3. 步骤2命令会生成source/page目录，以及index.md文件，修改index.md文件

	```
	---
	title: categories
	date: 2017-02-07 14:08:34
	type: "categories|tags"
	comments: false
	---
	```
### 添加MathJax公式
1. next主题默认支持MathJax，修改theme目录的_config.yml文件

	```
	# MathJax Support
	mathjax:
  	enable: true
	```
公式如下：
$$
\begin{eqnarray}
\nabla\cdot\vec{E} &=& \frac{\rho}{\epsilon_0}
\nabla\cdot\vec{B} &=& 0 \
\nabla\times\vec{E} &=& -\frac{\partial B}{\partial t} \
\nabla\times\vec{B} &=& \mu_0\left(\vec{J}+\epsilon_0\frac{\partial E}{\partial t} \right)
\end{eqnarray}
$$

### 添加评论功能
1. 在<http://www.duoshuo.com>注册账号，并建立域名
1. 在next主题的_config.yml文件中，修改

	```
	# Duoshuo ShortName
	duoshuo_shortname: xxx #刚才建立的域名名称，没有前缀http://和后缀.duoshuo.com
	```

2. 在多说“工具”中获取“通用代码”
3. 编辑文件[Hexo目录]/theme/next/layout/_partials/comments.swig

	将如下代码
	
	```
	<script type="text/javascript">
var duoshuoQuery = {short_name:"firechecking"};
	(function() {
		var ds = document.createElement('script');
		ds.type = 'text/javascript';ds.async = true;
		ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
		ds.charset = 'UTF-8';
		(document.getElementsByTagName('head')[0] 
		 || document.getElementsByTagName('body')[0]).appendChild(ds);
	})();
	</script>
	```
	添加在

	```
	<div class="ds-thread" data-thread-key="{{ page.path }}"
           data-title="{{ page.title }}" data-url="{{ page.permalink }}">
      </div>
	```
	后面
1. 在需要添加评论的文章标题中添加comments: true
2. 在不需要添加评论的文章标题中添加comments: false。格式如下：

	```
	---
	title: Hello World
	mathjax: true
	comments: true
	---
	```

1. 插入图片
	1. hexo根目录_config.yml中修改post_asset_folder:true
	1. hexo根目录执行
	
			npm install hexo-asset-image --save

	1. 将资源文件放在文章同级目录。例如：
			
			/source/_posts/hello-world.md
			/source/_posts/hello-world/test.jpg
			
	1. 按照markdown正常语法，使用以下语句插入图片
		
			![testImage](hello-world/test.jpg)

1. 首先不显示全文
	1. 在文章中不需要显示在首页的前面插入如下语句即可
	
			<!--more-->