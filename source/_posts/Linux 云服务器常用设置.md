---
title: Linux 云服务器常用设置
tag: 
	- Linux
---

<!-- markdownlint-disable MD010 -->

> 参考: [linux云服务器常用设置](http://www.cnblogs.com/xiaohuochai/p/7749727.html)

## 重装系统后重新登录

```bash
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ECDSA key sent by the remote host is
SHA256:fpZat/wEOrHUkMv81FrwdoGEBI6Xs+OlEHNElmyUu90.
Please contact your system administrator.
Add correct host key in /Users/jackzhang/.ssh/known_hosts to get rid of this message.
Offending ECDSA key in /Users/jackzhang/.ssh/known_hosts:41
ECDSA host key for 118.24.149.79 has changed and you have requested strict checking.
Host key verification failed.
```

服务器重装后连接远程服务器，本地和远程服务器 ssh 连接不上导致错误，因此需要删除本地 ssh 缓存信息

```
ssh-keygen -R <host>
```

## 登录 登出

- ssh 用户名密码登录

```bash
ssh <username>@<host>
```

回车输入密码

- 通过特定端口 登录

```bash
ssh -p <port> <username>@<host>
```

- 登出

```bash
exit

# 或者使用快捷键 ctrl + d
```

<!-- more -->

## 更改 shell

Ubuntu 系统默认使用的 shell 是 `dash`, 可通过下面命令切换成更常用的 `bash`

```bash
sudo dpkg-reconfigure dash
```

然后选择 <No>

![](http://ony85apla.bkt.clouddn.com/18-8-15/17396776.jpg)

执行 `ls -l /bin/sh`，查看 shell 类型已经修改为 bash

![](http://ony85apla.bkt.clouddn.com/18-8-15/37334854.jpg)

## 账号权限

为保证服务器安全性，需要设置一个高权限的账号来替代操作

- 新建一个新用户

ubuntu 系统默认不创建目录，如果需要添加 `-m` 参数

```bash
sudo useradd test -m
```

- 设置用户密码

```bash
sudo passwd test
```

- 将用户添加到 sudo 组中，操作命令前添加 sudo，可实现 root 权限

```bash
sudo gpasswd -a test sudo
```

- 配置 visudo (相当于编辑 vim /etc/sudoers 文件)

```bash
sudo visudo
```

`ALL=(ALL:ALL) ALL` 四个 all 分别指：任意用户、任意用户身份、任意用户组、任意命令；也就是**让该用户可以以 root 权限执行的命令**

![](http://ony85apla.bkt.clouddn.com/18-8-16/55830565.jpg)

- 切换到 test 账号下

```bash
su test
```

### 删除用户

- 强制删除用户

```bash
# -r 用于彻底删除，/home 下的目录会被移除，在其他位置的也将找出并删除
sudo userdel -r xxx
```

当前删除用户可能正被进程使用，因此需要先将使用的进程杀死

- 查看使用进程

```bash
ps -ef
```

```bash
UID        PID  PPID  C STIME TTY          TIME CMD
...
ubuntu   28817 28816  0 09:18 pts/0    00:00:00 -bash
root     29478     2  0 09:24 ?        00:00:00 [kworker/0:1]
root     29491  1318  0 09:24 ?        00:00:00 sshd: test [priv]
test    29543 29491  0 09:24 ?        00:00:00 sshd: test
root     29660  1318  0 09:26 ?        00:00:00 sshd: root [priv]
sshd     29661 29660  0 09:26 ?        00:00:00 sshd: root [net]
ubuntu   29664 28817  0 09:26 pts/0    00:00:00 ps -ef
test    32694     1  0 Aug15 ?        00:00:00 /lib/systemd/systemd --user
test    32695 32694  0 Aug15 ?        00:00:00 (sd-pam)
```

- 利用管道符查找进程

将 `ps` 的查询结果通过管道给 `grep` 查找包含特定字符串的进程

管道符 `|` 用于隔开两个命令，管道符左边命令的输出会作为右边命令的输入

```bash
ps -ef | grep test
```

- 杀死进程

```bash
sudo kill -s 9 29543
```

`-s 9` **指定传递给进程的信号是９，即强制、尽快终止进程**。29543 则是上面 `ps` 查到 test 的 PID

## 无密码登录

ssh 是网络上两台机器互联的一套协议，**默认使用 22 端口**

使用 `ssh test@xxx` 可以 test 用户名来登录服务器

可以通过添加 `ssh key` 实现无密码登录(**原理就是通过匹配公钥和私钥**)

- 本地机器上生成 `ssh key` (`id_rsa` 私钥文件，`id_rsa.pub` 为公钥文件)

```bash
ssh-keygen
```

- 使用 `ssh-copy-id` 命令将公钥文件传输到远程 `/home/test/.ssh/authorized_keys` 文件中

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub test@118.24.149.79
```

![](http://ony85apla.bkt.clouddn.com/18-8-15/9004089.jpg)

之后就可免密码登录 `ssh test@118.24.149.79`

注意遇到 `ssh-copy-id -i command not found`

- 本地终端执行以下命令

`curl -L https://raw.githubusercontent.com/beautifulcode/ssh-copy-id-for-OSX/master/install.sh | sh`

输入本地软件安装密码，看到 `Installed ssh-copy-id into /usr/local/bin`，说明 ssh-copy-id 安装成功

## 常见 linux 操作

- 查看系统盘

`/dev/vda1` 挂载系统盘

```bash
sudo fdisk -l
```

![](http://ony85apla.bkt.clouddn.com/18-8-16/91258893.jpg)

- 查看硬盘使用情况

```bash
df -h
```

![](http://ony85apla.bkt.clouddn.com/18-8-16/6567381.jpg)

## 增强服务器安全性

### 修改 ssh 端口

ssh 默认端口是 22。为提高服务器安全性，缩小被扫描和猜测几率，可将其修改为其他端口

- 修改 `/etc/ssh/sshd_conf` 文件端口为 1024 - 65536 之间端口(0-1024 端口一般系统占用)

```bash
sudo vim /etc/ssh/sshd_config
```

![](http://ony85apla.bkt.clouddn.com/18-8-16/93374088.jpg)

`/etc/ssh/sshd_conf` 文件末尾添加 `AllowUsers test`

![](http://ony85apla.bkt.clouddn.com/18-8-16/46325545.jpg)

- 然后重启 ssh 服务

```bash
sudo service ssh restart
```

### 取消密码登录

使用 ssh key 免密登录后，可以设置取消密码登录，同样是修改 `/etc/ssh/sshd_config`，将 `PasswordAuthentication yes` 改为 `PasswordAuthentication no`, 在重启 ssh 服务

## linux 包依赖安装

- 更新包列表

```bash
sudo apt-get update
```

- 卸载包但不删除配置

```bash
sudo apt-get remove <package>
```

- 卸载包并且删除配置

```bash
sudo apt-get purge <package>
```

- 安装常用依赖

```bash
sudo apt-get install vim openssl build-essential libssl-dev wget curl git
```

## 安装 Node.js

linux 安装 Node.js 一般有三种方式：

- 下载源码并手动编译二进制文件
- 直接下载二进制文件并解压
- 使用 `apt-get install nodejs` `sudo apt-get install npm`下载(不推荐)

### 直接下载二进制文件并解压

- 去官网下载和系统匹配的二进制文件

通过命令 `uname -a` 查看系统位数为 64 位

```
Linux VM-0-9-ubuntu 4.4.0-130-generic #156-Ubuntu SMP Thu Jun 14 08:53:28 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

- 下载的 tar 包文件上传到服务器并且解压，然后通过建立软连接变为全局

1）上传服务器可是任意路径，笔者这里为 `/home/test/software/`
2) 上传解压

```bash
# 上传远程服务器
scp -P <port> -r node-v10.15.3-linux-x64.tar.xz test@<host>:/home/test/software
# 解压
tar -xvf node-v10.15.3-linux-x64.tar.xz
# 重命名
mv node-v10.15.3-linux-x64.tar.xz nodejs-v10.15.3
```

3) 解压目录 `nodejs-v10.15.3` 下 `bin` 目录下有 node npm npx 三个执行文件。 **因为 `/home/test/software/node-v10.15.3-linux-x64/bin` 不在环境变量中，因此只能在当前目录下才能执行 node 的程序，如果在其他目录下想要执行 node 命令，必须通过绝对路径。如果想要在任意目录访问，需要将 node 所在目录添加到环境变量；或者通过软连接的形式将 node 和 npm 链接到系统默认的 path 目录下**

```bash
sudo ln -s /home/test/software/nodejs-v10.15.3/bin/node /usr/local/bin/node
sudo ln -s /home/test/software/nodejs-v10.15.3/bin/npm /usr/local/bin/npm
```

- 删除软链接

```bash
# 如删除上述软连接
rm -rf /usr/local/bin/node
```

### 执行 `npm i -g <package>` 报错没有找到命令

npm 全局安装找不到命令，**本质还是环境变量问题**

linux 环境变量有三种：

- 当前用户当前 shell 有效（临时环境变量关闭则失效）
- 当前用户有效
- 所有用户都有效

#### 当前用户当前 shell 有效

在 shell 中执行以下命令，`$PATH:` 后跟想要加入环境变量的目录

```bash
export PATH=$PATH:/home/test/software/nodejs-v10.15.3/bin
```

#### 当前用户有效

修改当前用户目录下 `./bashrc` 文件, 执行 `source ~/.bashrc`

```bash
export PATH=$PATH:/home/test/software/nodejs-v10.15.3/bin
```

#### 所有用户

修改 	`/etc/profile` 文件, 执行 `source /etc/profile`

```bash
export PATH=$PATH:/home/test/software/nodejs-v10.15.3/bin
```

## 安装 Node.js 常用依赖

- 安装 n 模块管理 node 版本

```bash
# 常用指令
# 列出当前安装的 node 版本
n
# 列出 node 全部现有版本
n ls
# 列出 node 发行的最新版本
n --latest
# 列出 node 长期支持的最新版本
n --lts
# 安装最新发行的 node 版本
n latest
# 安装最新的长期支持版本
n lts

# 安装特定版本 node
n <version>

# 使用特定版本 node
n use <version>

# 删除特定 node(可删除多个)
n rm <version ...>

# 删除除了当前版本的所有版本
n prune
```

- 安装 nrm 管理 registry

```bash
# 全局安装
npm i nrm -g

# 列出所有 registry
nrm ls

# 使用特定 registry
nrm use npm

# 添加特定 registry
nrm add <registry-name> <registry>

# 删除特定 registry
nrm del <registry-name>
```

- 安装常用 Node.js 模块

``` bash
npm i pm2 webpack gulp grunt-cli -g
```

## pm2 管理 node 进程

pm2 是一个带有负载均衡功能的 Node 应用的进程管理器, 括守护进程，监控，日志的一整套完整的功能

- 进程守护，系统崩溃自动重启
- 启动多进程，充分利用 CPU 和内存
- 自带日志记录功能

### 命令行

```bash
# 启动
pm2 start app.js
pm2 start app.js --name my-api   # 启动进程并命名为 my-api
pm2 start app.js -i 4           # 根据 CPU 核数启动进程个数, 这个 4 个应用会自动负载均衡
pm2 start app.js --watch   # 实时监控 app.js 的方式启动，当app.js文件有变动时，pm2 会自动 reload；更复杂的监听最好通过配置文件

# 停止
pm2 stop all  # 停止 PM2 列表中所有的进程
pm2 stop 0    # 停止 PM2 列表中进程为 0 的进程
pm2 stop app

# 删除 PM2 进程
pm2 delete 0     #删除 PM2 列表中进程为 0 的进程
pm2 delete all   #删除 PM2 列表中所有的进程
pm2 delete app

# 重启
pm2 restart all     # 重启 PM2 列表中所有的进程
pm2 restart 0      # 重启 PM2 列表中进程为 0 的进程
pm2 restart app

# 重载
pm2 reload all    # 重载 PM2 列表中所有的进程
pm2 reload 0     # 重载 PM2 列表中进程为 0 的进程
pm2 reload app

# 查看进程
pm2 list
pm2 show 0 # pm2 info 0  #查看进程详细信息，0 为 PM2 进程 id
pm2 show app  #查看进程详细信息，app 为 PM2 进程 name, 下同

# 进程信息
pm2 info app

# 监控
pm2 monit

# 日志操作
pm2 log app
pm2 logs [--raw]   # Display all processes logs in streaming
pm2 flush          # Empty all log file
pm2 reloadLogs    # Reload all logs

# 升级 PM2
npm install pm2@lastest -g   # 安装最新的 PM2 版本
pm2 updatePM2                # 升级pm2

# 更多命令参数请查看帮助
pm2 --help
```

### 配置文件

```js
# 执行pm2 start pm2.config.js
module.exports = {
  "apps": {
        "name": "wuwu",                             // 进程名          
        "script": "./bin/www",                      // 执行文件，如果是 node 项目就是入口文件
        "cwd": "./",                                // 根目录
        "args": "",                                 // 传递给脚本的参数，可以数组
        "watch": true,                              // 是否监听文件变动然后重启
        "ignore_watch": [                           // 不用监听的文件
            "node_modules",
            "logs"
        ],
        "exec_mode": "cluster_mode",                // 应用启动模式，支持fork和cluster模式
        "instances": 4,                             // 应用启动实例个数，仅在cluster模式有效 默认为fork；或者 max
        "max_memory_restart": 8,                    // 最大内存限制数，超出自动重启
        "error_file": "./logs/app-err.log",         // 错误日志文件
        "out_file": "./logs/app-out.log",           // 正常日志文件
        "merge_logs": true,                         // 设置追加日志而不是新建日志
        "log_date_format": "YYYY-MM-DD HH:mm:ss",   // 指定日志文件的时间格式
        "min_uptime": "60s",                        // 应用运行少于时间被认为是异常启动
        "max_restarts": 30,                         // 最大异常重启次数，即小于min_uptime运行时间重启次数；
        "restart_delay": "60s"                      // 异常重启情况下，延时重启时间
        "env": {
           "NODE_ENV": "development",                // 环境参数，当前指定为开发环境 process.env.NODEPORT
           "PORT": 9091                              // process.env.PORT
        },
        "env_development": {
            "NODE_ENV": "development",              // 环境参数，当前指定为开发环境 pm2 start app.js --env development
            "PORT": 9091                             
        },
        "env_test": {                               // 环境参数，当前指定为测试环境 pm2 start app.js --env test
            "NODE_ENV": "test",
            "PORT": 9091                              
        },
  			"env_production": {                         // 环境参数，当前指定为生产环境 pm2 start app.js --env production
            "NODE_ENV": "production",
            "PORT": 9091                              
        }
    }
}
```

## 配置 nginx

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

### 反向代理

笔者将域名代理到 coding pages 博客

```bash
server {
	listen 80;
	server_name blog.jhgrrewq.com;

	location / {
		proxy_pass https://qwerrghj.coding.me/;
	}
}
```

### 配置 ssl 证书

- 生成 ssl 证书

- 基本 https 配置

```bash
# 基本配置
server {
	listen 443;
	server_name jhgrrewq.com;
	ssl on;
	ssl_certificate /etc/nginx/certs/1_jhgrrewq.com_bundle.crt;
	ssl_certificate_key /etc/nginx/certs/2_jhgrrewq.com.key;
}

# 负载
upstream jack {
  server 127.0.0.1:<port>;
}

server {
	listen 443;
	server_name jhgrrewq.com;
	ssl on;
	ssl_certificate /etc/nginx/certs/1_jhgrrewq.com_bundle.crt;
	ssl_certificate_key /etc/nginx/certs/2_jhgrrewq.com.key;

	location / {
		proxy_pass http://jack;
	}
}
```