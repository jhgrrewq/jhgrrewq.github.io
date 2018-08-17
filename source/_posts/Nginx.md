---
title: Nginx
tag: 
	- Nginx
---

<!-- markdownlint-disable MD010 -->

> 参考:[Nginx基本配置备忘
](https://zhuanlan.zhihu.com/p/24524057?refer=wxyyxc1992)

### 常用命令

- 更新包列表

```bash
sudo apt-get update
```

- 安装 nginx

```bash
sudo apt-get install nginx
```

- 停掉 nginx

```bash
# 快速停止 nginx
nginx -s stop

# 完整有序停止 nginx
nginx -s quit

# 查看当前 nginx 进程 pid, 强制杀死进程
ps -ef | grep nginx
pkill -9 nginx
```

- 启动 nginx

```bash
# nginx 安装目录 -c nginx 配置文件
sudo nginx -c /etc/nginx/nginx.conf
```

- 重启 nginx (热启动，修改配置重启不影响线上)

```bash
# 进入 nginx 安装目录
sudo nginx -s reload
```

- 测试 nginx 配置

```bash
# 进入 nginx 安装目录
sudo nginx -t
```

<!-- more -->

## Nginx 配置

```bash
# 全局块
...
# events 块
events {
	...
}
# http 块
http {
	# http 全局块
	...
	# 虚拟主机 server 块
	server {
		# server 全局块
		...
		# location 全局块
		location [PATTERN]{
			...
		}
		location [PATTERN]{
			...
		}
	}
	server {
		...
	}
	# http 全局块
	...
}
```

Nginx 配置文件由如下几部分组成：

- **全局块** 配置影响 nginx 全局的指令。一般有运行 nginx 服务器的用户组，nginx 进程 pid 存放路径、日志存放路径，配置文件引入，允许生成 `work process` 数等

- **events 块** 配置影响 nginx 服务器或用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时连接多个网络请求，开启多个网络连接序列化等

- **http 块** 可嵌套多个 `server`，配置代理，缓存，日志定义等大多数功能和第三方模块的配置。如文件引入、mime-type 定义、日志自定义、连接超时时间、单链接请求数等

- **server 块** 配置虚拟主机相关参数，一个 http 中可有多个 server

- **location 块** 配置请求路由以及各种页面的处理情况

```bash
################### 每个指令必须有分号结束 ####################
# user administrator administrators; # 配置用户或组，默认 nobody nobody
# worker_processes 2; #允许生成的进程数，默认为 1
# pid /nginx/pid/nginx.pid; # 指定 nginx 进程运行文件存放地址
error_log log/error.log debug; # 指定日志路径，级别。该设置可放入全局块、http 块、server 块， 级别依次为 debug|info|notice|warn|error|crit|alert|emerg
events {
	accept_mutex on; # 设置网络连接序列化，防止惊群现象发生，默认为 on
	multi_accept on; # 设置一个进程是否同时接受多个网络连接，默认为 off
	# use epoll; # 事件驱动模型，select|poll|kqueue|epoll|resig|/dev/poll|eventport
	worker_connections 1024; # 最大连接数，默认为 512
}
http {
	include mime.types; # 文件扩展名和文件类型映射表
	default_type application/octet-stream; # 默认文件类型，默认为 text/plain
	# access_log off; # 取消服务日志
	log_format myFormat '$remote_addr–$remote_user [$time_local] $request $status $body_bytes_sent $http_referer $http_user_agent $http_x_forwarded_for'; # 自定义格式
	access_log log/access.log myFormat;  # combined 为日志格式的默认值
	sendfile on;   # 允许 sendfile 方式传输文件，默认为 off，可以在 http 块，server 块，location 块
	sendfile_max_chunk 100k;  # 每个进程每次调用传输数量不能大于设定的值，默认为 0，即不设上限
	keepalive_timeout 65;  # 连接超时时间，默认为 75s，可以在 http，server，location块

	# 定义常量
	upstream mysvr {
		server 127.0.0.1:8081;
		server 192.168.10.121:3333 backup; # 热备
	}
	error_page 404 https://www.baidu.com; # 错误页

	# 定义某个负载均衡服务器
	server {
		keepalive_requests 120; # 单连接请求上线次数
		listen 4545; # 监听端口
		server_name 127.0.0.1; # 监听地址
		location ~*^.+$ { # 请求的 url 过滤，正则匹配，~ 为区分大小写，~* 为不区分大小写
			# root path; 根目录
			# index index.html; # 设置默认页
			proxy_pass http://mysvr; # 请求转到 mysvr 定义的服务器列表
			deny 127.0.0.1; # 拒绝的 ip
			allow 127.18.5.54; # 允许的 ip

		}
	}
 }
```

## 静态站点

```bash
http {
	server {
		listen 80;
		server_name www.domain1.com;
		location / {
			index index.html
			root /var/www/domain1.com/htdocs;
		}
	}
	server {
		listen 80;
		server_name www.domain2.com
		location / {
			index index.html;
			root /var/www/domain2.com/htdocs;
		}
	}
}
```

### 主机和端口配置

```bash
listen 127.0.0.1:8080;
listen *:8080;
listen localhost:8080;
# IPV6
listen [::]:8080;
# other params
listen 443 default_server ssl;
listen 127.0.0.1 default_server accept_filter=dataready backlog=1024
```

### 服务器域名配置

```bash
# 支持多域名配置
server_name www.domain.com domain.com
# 支持泛域名解析
server_name *.domain.com
# 支持对于域名的正则匹配
server_name ~^\.domain\.com$;
```

### URL 匹配

```bash
location = / {
	# 完全匹配 =
	# 大小写敏感 ~
	# 忽略大小写 ~*
}
location ^~ /images/ {
	# 前半部分匹配 ^~
	# 可使用正则 如 location ~* \.(gif|jpg|png)$ {}
}
location / {
	# 如果以上都未匹配，进入这里
}
```

### 文件路径配置

```bash
location / {
	root /home/domain/test/;
}
```

### 首页配置

```bash
index /html/index.html /php/index.php;
```

### 别名配置

```bash
location /blog {
	alias /home/domain/blog/;
}
location ~ ^/blog/(\d+)/([\w-]+)$ {
	# /blog/20141202/article-name  
  # -> /blog/20141202-article-name.md
  alias /home/domain/blog/$1-$2.md;
}
```

### 重定向配置

```bash
error_page 404 /404.html;
error_page 502 503 /50x.html;
error_page 404 =200 /1x1.gif;
location / {
	error_page 404 @fallback;
}
location @fallback {
	# 将请求反向代理到上游服务器处理
	proxy_pass http://localhost:9000;
}
```