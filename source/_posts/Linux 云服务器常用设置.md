---
title: Linux 云服务器常用设置
tag: 
	- Linux
---

> 参考: [linux云服务器常用设置](http://www.cnblogs.com/xiaohuochai/p/7749727.html)

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
sudo adduser test -m
```

- 设置用户密码

```bash
sudo passwd test
```

- 将用户添加到 sudo 组中，操作命令前添加 sudo，可实现 root 权限

```bash
sudo gpasswd -a test sudo
```

- 配置 visudo

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

- 修改 `/etc/ssh/sshd_conf` 文件端口为 1024 - 65536 之前端口(0-1024 端口一般系统占用)

```bash
sudo vim /etc/ssh/sshd_config/
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

## 配置 Node.js 生产环境

- 更新包列表

```bash
sudo apt-get update
```

- 安装常用依赖

```bash
sudo apt-get install vim openssl build-essential libssl-dev wget curl git
```

- 安装 nvm 模块管理 node 版本

```bash
# 常用指令
# 安装特定版本 node
nvm install 8.0.0

# 使用特定版本 node
nvm use 8.0.0

# 设置默认 node
nvm alias default 8.0.0
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

```bash
npm i pm2 webpack gulp grunt-cli -g
```

## pm2 管理 node 进程

pm2 是一个带有负载均衡功能的 Node 应用的进程管理器, 括守护进程，监控，日志的一整套完整的功能

```bash
# 启动
pm2 start app.js
pm2 start app.js --name my-api   # 启动进程并命名为 my-api
pm2 start app.js -i 4           # 根据 CPU 核数启动进程个数, 这个 4 个应用会自动负载均衡
pm2 start app.js --watch   # 实时监控 app.js 的方式启动，当app.js文件有变动时，pm2 会自动 reload；更复杂的监听最好通过配置文件

# 查看进程
pm2 list
pm2 show 0 # pm2 info 0  #查看进程详细信息，0 为 PM2 进程 id
pm2 show app  #查看进程详细信息，app 为 PM2 进程 name, 下同

# 监控
pm2 monit

# 停止
pm2 stop all  # 停止 PM2 列表中所有的进程
pm2 stop 0    # 停止 PM2 列表中进程为 0 的进程
pm2 stop app

# 重载
pm2 reload all    # 重载 PM2 列表中所有的进程
pm2 reload 0     # 重载 PM2 列表中进程为 0 的进程
pm2 reload app

# 重启
pm2 restart all     # 重启 PM2 列表中所有的进程
pm2 restart 0      # 重启 PM2 列表中进程为 0 的进程
pm2 restart app

# 删除 PM2 进程
pm2 delete 0     #删除 PM2 列表中进程为 0 的进程
pm2 delete all   #删除 PM2 列表中所有的进程
pm2 delete app

# 日志操作
pm2 logs [--raw]   # Display all processes logs in streaming
pm2 flush          # Empty all log file
pm2 reloadLogs    # Reload all logs

# 升级 PM2
npm install pm2@lastest -g   # 安装最新的 PM2 版本
pm2 updatePM2                # 升级pm2

# 更多命令参数请查看帮助
pm2 --help
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

- 重启 nginx

```bash
# 进入 nginx 安装目录
sudo nginx -s reload
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

### 


## 配置 ssl 证书

- 生成 ssl 证书