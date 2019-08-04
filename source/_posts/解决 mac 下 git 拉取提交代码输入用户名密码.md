---
title: 解决 mac 下 git 拉取提交代码输入用户名密码
tag: 
	- git
---

> 需要使用 credential-osxkeychain helper 在 Mac OSX keychain 中存储 git 密码

<!--more-->

需要执行以下步骤：

```shell
$ git credential-osxkeychain

// 测试是否已安装 heilper， 没有会报错 git: 'credential-osxkeychain' is not a git command. See 'git --help'.

$ curl -s -O \ https://github-media-downloads.s3.amazonaws.com/osx/git-credential-osxkeychain
 
// 下载安装 helper

$ chmod u+x git-credential-osxkeychain

// 修改文件权限

$ sudo mv git-credential-osxkeychain \ "$(dirname $(which git))/git-credential-osxkeychain"

// 将 helper 搬移到 git 安装的目录

$ git config --global credential.helper osxkeychain

// 让 git 使用该helper

```