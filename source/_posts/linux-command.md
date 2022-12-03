---
title: linux-command
date: 2022-11-12 20:38:03
categories: tech
tags: tools
category_bar: true
---

记录我常用的linux命令

# scp

```shell
scp local_file remote_username@remote_ip:remote_folder 
scp local_file remote_username@remote_ip:remote_file 
scp local_file remote_ip:remote_folder
scp local_file remote_ip:remote_file
```

没有指定用户名，命令执行后需要输入用户名和密码，仅指定了远程的目录，文件名字不变。
