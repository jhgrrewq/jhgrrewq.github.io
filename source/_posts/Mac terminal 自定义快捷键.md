---
title: Mac terminal 自定义快捷键
tag: 
	- terminal
---

比较长的终端命令可以通过自定义快捷键

- 编辑 `~/.bashrc`, 每行输入一个 alias 命令

```bash
ssh -p 20000 test@118.24.149.79
```

- 编辑 `~/.bash_profile`，输入如下命令（该文件是每次打开终端都会自动执行的文件）

```bash
source ~/.bashrc
```