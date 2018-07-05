---
title: Supervisor学习
date: 2017-08-16 11:18:48
tags: [Python,守护进程]
categories: [系统,守护进程]
comments: true
---

# Supervisor简介
1. Supervisor以server/client的方式，守护unix类系统中多个进程
1. Supervisor以子进程（subprocess）的形式启动需要守护的进程，因此能准确监控进程的运行状态
1. Supervisor可以管理进程组，也可以管理进程之间启动、结束的优先级顺序
<!--more-->

1. Supervisor通过一个conf文件配置所有信息，及守护进程列表
1. Supervisor支持控制台或web界面的控制
1. Supervisor支持简单的事件监听，可以使用其他语音编写程序监听Supervisor内部事件，也可以通过XML-RPC接口进行控制。Supervisor还內建了扩展机制支持Python扩展
1. Supervisor除Windows外的操作系统，暂不支持Python3
1. 组成
	1. supervisord：响应命令、守护进程、记录日志等，配置文件默认位于**/etc/supervisord.conf**
	1. supervisorctl：supervisor命令行客户端，通过socket或TCP与supervisord连接，能连接到不同的supervisord，能获取status、能控制start、stop、能获取running list，配置位于supervisord相同配置文件的**[supervisorctl]**类下
	1. Web Server：和supervisorctl功能相似的web服务器，配置位于supervisord相同配置文件的**[inet_http_server]**类下
	1. XML-RPC Interface：Web Server除了支持http外，还支持XML-RPC接口，见[XML-RPC API Documentation](http://supervisord.org/api.html#xml-rpc)

# [Supervisor安装](http://supervisord.org/installing.html)
1. 安装
		
		pip install supervisor

1. 配置文件

```
>> echo_supervisord_conf
>> echo_supervisord_conf > /etc/supervisord.conf
```
# Supervisor使用
## 运行Supervisor
1. 添加被守护进程
	1. 在configrue文件中，添加如下内容

	```
	[program:foo]
command=/bin/cat
	```
1. 在Terminal中运行：`location/supervisord`
	1. supervisord默认后台运行，并将日志保存在`$CWD/supervisor.log`
	1. 如果因为调试需要让supervisord前台运行，需要在Terminal中加-n命令
	1. supervisord支持的控制台参数见[官方文档](http://supervisord.org/running.html#supervisord-command-line-options)

1. 在Terminal中运行：`location/supervisorctl`
	1. supervisorctl以client方式和supervisord通信
	1. supervisorctl支持的控制台参数见[官方文档](http://supervisord.org/running.html#supervisorctl-command-line-options)
	1. supervisorctl支持的操作见[官方文档](http://supervisord.org/running.html#supervisorctl-actions)

1. supervisord开机自启
	1. 通过pip、easy install等方式安装的supervisord已经集成了开机自启
	1. <http://serverfault.com/questions/96499/how-to-automatically-start-supervisord-on-linux-ubuntu>
	1. <https://github.com/Supervisor/initscripts>

# Supervisor二次开发
# Supervisor代码解读与技术学习