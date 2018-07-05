---
title: 使用Nginx进行TCP负载均衡
date: 2017-03-23 14:49:37
tags: [服务器,Nginx,负载均衡]
categories: [服务器,Nginx]
comments: true
---

# Ubuntu 16.04
1. 参考文档
	* <http://www.linuxidc.com/Linux/2015-05/117654.htm>
	* <http://www.linuxidc.com/Linux/2016-08/134080.htm>
	* <http://nginx.org/en/docs/>
<!-- more -->
1. 更新系统

	```
	sudo apt-get update
	sudo apt-get upgrade
	```

1. 安装Nginx的依赖包  zlib pcre openssl（可以源码安装也可以直接系统安装）
	
	```
	sudo apt-get install libpcre3 libpcre3-dev zlib1g-dev libssl-dev build-essential
	sudo apt-get install openssl
	```
	
1. 编译安装Nginx
	* <http://nginx.org/download/nginx-1.10.3.tar.gz>
	
	```
	tar -zxvf nginx-1.10.3.tar.gz
	cd nginx-1.10.3
	sudo ./configure --prefix=/usr/local/nginx --with-openssl=/usr/include/openssl --with-stream
	sudo make && sudo make install
	```

1. 修改conf/nginx.conf

	```
	worker_processes auto;
	#error_log /var/log/nginx/error.log info;
	events {
	    use epoll;
	    worker_connections  1024;
	}
	stream {
	    upstream backend {
	        server 127.0.0.1:9990 weight=1;
	        server 127.0.0.1:9991 weight=1;
	    }
	    server {
	        listen 127.0.0.1:12345;
	        proxy_connect_timeout 60s;
	        proxy_timeout 60s;
	        proxy_pass backend;
	    }
	}
	```
1. 运行Nginx

	```
	cd /usr/local/nginx/sbin
	sudo ./nginx
	```