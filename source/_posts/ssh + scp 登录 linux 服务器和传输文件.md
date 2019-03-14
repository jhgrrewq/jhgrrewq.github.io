---
title: ssh + scp 登录 linux 服务器和传输文件
tag: 
	- linux
	- ssh
	- scp
---

## ssh 登录 linux 服务器

ssh 是用于计算机之间加密登录的协议

```javascript
ssh user@host
```

可以连接远程主机的指定端口

```javascript
ssh -p port user@host
```

使用 logout 或 exit 命令退出登录

```javascript
exit
```

<!-- more -->

使用 root 身份登录 linux 服务器后会进入到下图红色箭头位置， __cd ../__ 进入上一层，再 __ls -a__ 可以看到类似下图 linux 系统常见的目录结构

![](http://ony85apla.bkt.clouddn.com/18-1-25/64988547.jpg)

## scp 传输文件

scp 是 secure copy 简写，用于在 linux 下进行远程拷贝文件的命令。类似有 cp 命令，只限于在本机拷贝文件

- 指定端口

```bash
scp -P <port>
```

- 上传一个文件

如 scp 本地文件路径/文件名 远程服务器用户名@ip域名:文件路径

```javascript
scp path/filename user@host:path/folderName
```

- 下载一个文件

如 scp 远程服务器用户名@ip域名:文件路径/文件名 本地文件路径

```javascript
scp user@host:path/filename path/folderName
```

- 上传一个文件夹

如 scp -r 本地文件路径/文件夹 远程服务器用户名@ip域名:文件路径

```javascript
scp -r path/folderName user@host:path/folderName
```

- 下载一个文件夹

如 scp -r 远程服务器用户名@ip域名:文件路径/文件夹 本地文件路径

```javascript
scp -r user@host:path/folderName path/folderName
```