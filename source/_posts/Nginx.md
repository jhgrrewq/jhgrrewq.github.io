---
title: Nginx
tag: 
	- Nginx
---

<!-- markdownlint-disable MD010 -->

> 参考:[Nginx基本配置备忘
](https://zhuanlan.zhihu.com/p/24524057?refer=wxyyxc1992)、[nginx中location详解](https://blog.csdn.net/chlinwei/article/details/67631830)

## 常用命令

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
nginx -c /etc/nginx/nginx.conf
```

- 重启 nginx (热启动，修改配置重启不影响线上)

```bash
# 进入 nginx 安装目录
nginx -s reload
```

- 测试 nginx 配置

```bash
# 进入 nginx 安装目录
nginx -t
```

<!-- more -->

## Nginx 配置 (每个指令必须有分号结束)

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

### try_files

```bash
try_files $uri $uri.html $uri/index.html @other;
location @ther {
	# 尝试寻找匹配 uri 文件，失败会转到上游处理
	proxy_pass http://localhost:9000;
}
location / {
	# 尝试寻找匹配 uri 文件，没找到直接返回 502
	try_files $uri $uri.html =502
}
```

## 反向代理

> 反向代理方式是指用代理服务器来接受网络上的连接请求，然后将请求转发给内部网络的上游服务器，并将上游服务器得到的结果返回给网络上请求连接的客户端，此时代理服务器对外表现就是一个 web 服务器

### 负载均衡配置

`upstream` 定义一个上游服务器集群

`proxy_pass` 将请求转发到有处理能力的端上，默认不会转发请求中的 Host 头部

```bash
events {
	...
}
http {
	upstream jack {
		server 127.0.0.1:8080;
	}
	# 80 端口，可配置多个虚拟主机
	server {
		listen 80;
		server_name www.jhgrrewq.com

		location / {
			# 将请求转发到有处理能力的端上
			proxy_pass http://jack;

			### 下面都是次要项
			proxy_set_header Host $host;
			proxy_method POST;
			# 指定不转发的头部字段
			proxy_hide_header Cache-Control;
			# 指定转发的头部字段
			proxy_pass_header Server-IP;
			# 是否转发包体
			proxy_pass_request_body on|off;
			# 是否转发头部
			proxy_pass_request_headers on|off;
			# 显形/隐形 URI，上游发生重定向时，nginx 是否同步更改 uri
			proxy_redirect on|off;
		}
	}
}
```

### 实例

处理请求，如果是静态文件， Nginx 直接返回，否则交给 Node 服务器处理

```js
// app.js
const http = require('http');
http.createServer((req, res) => {
  res.end('hello world');
}).listen(9000)
```

任何请求归来都会返回 'hello world'

```bash
# nginx
events {
	# ...
}
http {
	server {
		listen 127.0.0.1:8080;
		# 如果请求路径和文件路径按照如下方式匹配找到，直接返回
		try_file $url $url/index.html;
		location ~* ^/(js|css|image|font)/$ {
			# 静态资源都在 static 文件夹下
			root /home/jhgrrewq/www/static/;
		}
		location /app {
			# Node.js 在 9000 开了一个监听端口
			proxy_pass http://127.0.0.1:9000;
		}
		# 上面处理出错或者找不到，返回对应状态码文件
		error_page 404 /404/html;
		error_page 502 503 504 /50x.html;
	}
}
```

首先 `try_files`, 尝试直接匹配文件；没找到就匹配静态资源；还没找到就交给 Node 处理，否则就返回 4xx/5xx 状态码

## 基本 https 配置

```bash
server {
	listen 192.168.1.11:443; # ssl 端口
	server_name test.com;
	# 为一个 server 开启 ssl 支持
	ssl on;
	# 指定 pem 格式证书文件
	ssl_certificate /etc/nginx/test.pem;
	# 指定 pem 格式私钥文件
	ssl_certificate_key /etc/nginx/test.key;
}
```

## Nginx location 匹配

`location block` 的基本语法

```bash
location [=|~|~*|^~|@] patten {...}
```

`[=|~|~*|^~|@]` 为 `location modifier`, 定义 Nginx 如何匹配之后的 `pattern`, 以及 `pattern` 的最基本属性（简单字符串或者正则表达式） 

### location = 

```bash
server {
	server_name www.domain.com;
	location = /abcd {...}
}
```

匹配情况：

- http://www.domain.com/abcd # 完全匹配
- http://www.domain.com/ABCD # 如果运行 Nginx 的系统本身忽略大小写也匹配，如 windows
- http://www.domain.com/abcd?params1 # 忽略查询字符串参数，也就是 ？后面的内容
- http://www.domain.com/abcd/ # 不匹配，末尾存在反斜杠，Nginx 不认为完全匹配
- http://www.domain.com/abcde # 不匹配

### location (none)

可以不写 `location modifier`, Nginx 仍然能去匹配 `pattern`。这种情况下，**匹配指定 `pattern` 开头的 url（只能是普通字符串，不能是正则）**

```bash
server {
	server_name www.domain.com;
	location /abcd {...}
}
```

匹配情况：

- http://www.domain.com/abcd # 完全匹配
- http://www.domain.com/ABCD # 如果运行 Nginx 的系统本身忽略大小写也匹配，如 windows
- http://www.domain.com/abcd?params1 # 忽略查询字符串参数，也就是 ？后面的内容
- http://www.domain.com/abcd/ # 末尾存在反斜杠也属于匹配
- http://www.domain.com/abcde # 不匹配

### location ~

**对大小写敏感（不能忽略大小写） `pattern` 只能为正则**

```bash
server {
	server_name www.domain.com;
	location ~ ^/abcd$ {...}
}
```

匹配情况：

- http://www.domain.com/abcd # 完全匹配
- http://www.domain.com/ABCD # 不匹配，不能忽略大小写（但是如 window 等大小写不敏感系统，~ 和 ~* 不起作用）
- http://www.domain.com/abcd?params1 # 忽略查询字符串参数，也就是 ？后面的内容
- http://www.domain.com/abcd/ # 不匹配，末尾存在反斜杠，不匹配正则表达式 ^/abcd$
- http://www.domain.com/abcde # 不匹配

### location ~*

**对大小写不敏感（忽略大小写） `pattern` 只能为正则**

```bash
server {
	server_name www.domain.com;
	location ~* ^/abcd$ {...}
}
```

匹配情况：

- http://www.domain.com/abcd # 完全匹配
- http://www.domain.com/ABCD # 匹配，忽略大小写（但是如 window 等大小写不敏感系统，~ 和 ~* 不起作用）
- http://www.domain.com/abcd?params1 # 忽略查询字符串参数，也就是 ？后面的内容
- http://www.domain.com/abcd/ # 不匹配，末尾存在反斜杠，不匹配正则表达式 ^/abcd$
- http://www.domain.com/abcde # 不匹配

### location ^~

匹配情况类似 `location (none)` 情况，以指定匹配模式开头的 uri 被匹配，不同的是，**一旦匹配成功， Nginx 就停止寻找其他的 location 块进行匹配**

### location @

定义一个 location 块，不能被外部 client 访问，只能被 Nginx 内部配置指令访问，如 `try_file` `error_page`

### 搜索顺序和生效优先级

**`Server` 块中的 `location` 块的次序是不重要的，Nginx 会根据 `location modifier` 优先级依次用 uri 匹配 `pattern`:**

```bash
1. =
2. (none) # 如果 pattern 完全匹配 uri（不是只匹配 uri 头部）
3. ^~
4. ~ 或 ~*
5. （none）# pattern 匹配 uri 头部
```

```bash
location = / {
	# 精确匹配 /，主机名后面不能带任何字符串
	[ config a ]
}
location / {
	# 所有地址都以为 / 开头，因此该规则会匹配到全部请求
	# 正则和最长字符串会优先匹配
	[ config b ]
}
location /documents/ {
	# 匹配到任何 /documents/ 开头的地址，匹配符合后还要继续往下搜索
	# 只有后面的正则表达式没有匹配，才会采用这一条
	[ config c ]
}
location ^~ /images/ {
	# 匹配到任何 /images/ 开头的地址，匹配符合，停止往下搜索正则，采用这一条
	[ config d ]
}
location ~* \.(gif|jpg|jpeg)$ {
	# 匹配全部 gif、jpg 或 jpeg 结尾的请求
	# 但是请求 /images/ 下的图片会被 config d 处理，因为 ^~ 到达不了这一条正则
}
```

### 实际使用建议

一般至少有三个匹配规则：

- **直接匹配网站根，通过域名访问网站比较频繁**

```bash
# 可以转发给后端服务器，也可以是一个静态站点首页
location = / {
	proxy_pass http://www.domain.com/index;
}
```

- **处理静态文件请求**

```bash
# 一般两种模式，目录匹配 或 后缀匹配
location ^~ /static/ {
	root /htocs/static/;
}
location ~* \.(gif|jpg|jpeg|png|css|js|ico)$ {
	root /htocs/res/;
}
```

- **一般用来转发动态(可认为是非静态文件)请求到后端应用服务器，如代理等**

```bash
location / {
	proxy_pass http://www.domain.com/;
}
```