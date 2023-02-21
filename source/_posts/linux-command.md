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

# 查看linux信息

## 查看内核版本

- `cat /proc/version`
- `uname -a`

## 查看linux版本信息

- `lsb_release -a`

- `cat /etc/issue`

## 查看系统的架构

- `dpkg --print-architecture`
- `arch`
- `file /lib/systemd/systemd`

> x86_64，x64，AMD64基本上是同一个东西
>
> - x86是intel开发的一种32位指令集
> - x84_64是CPU迈向64位的时候
> - x86_64是一种64位的指令集，x86_64是x86指令的超集，在`x86`上可以运行的程序，在`x86_64`上也可以运行，`x86_64是AMD发明的，也叫AMD64`
>
> 现在用的`intel/amd的桌面级CPU`基本上都是`x86_64`，与之相对的arm、pcc等都不是`x86_64`

