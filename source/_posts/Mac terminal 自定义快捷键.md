---
title: Mac terminal 自定义快捷键
tag: 
	- terminal
---

比较长的终端命令可以通过自定义快捷键

- 编辑 `~/.bash_profile`，输入如下命令

```bash
alias ssh_jhgrrewq='ssh -p 20000 test@118.24.149.79'
```

- 本地终端输入 `source ~/.bashrc`, 打开终端自动执行该文件