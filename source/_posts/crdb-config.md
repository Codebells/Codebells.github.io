---
title: CockRoachDB配置跨域集群
date: 2022-03-26 16:42:06
categories: 
  - [database]
  - [system]
tags: [system,database]
category_bar: true
excerpt: CRDB做简单的了解，可以进行简单使用及测试编译，配置三节点跨域集群
---

 这段时间老师需要我测试CRDB的性能，因而对CRDB做了些简单的了解，可以进行简单使用及测试编译，首先说明我使用的是阿里云服务器Ubuntu20.04，各位可以根据需要自行选择操作系统，centos和ubuntu，CRDB都支持,下面是我在配置三节点跨域集群过程中记录的执行语句

官方给了个同步时钟的过程，而我的服务器全部由阿里云提供，时钟没啥影响

# 安装CRDB

下面的版本可以根据需要自行更改

```
curl https://binaries.cockroachdb.com/cockroach-v21.2.4.linux-amd64.tgz | tar -xz && sudo cp -i cockroach-v21.2.4.linux-amd64/cockroach /usr/local/bin/
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

CRDB存在两种启动模式，secure和insecure，两者区别在于是否加密，secure模式只能赋予安全证书的节点进行连接集群，而insecure模式任何节点都可以直接加入集群。secure模式多了一个创建证书以及赋予证书的过程，我由于在使用insecure模式时，测试吞吐量很低，以及在大数据量的情况下节点容易宕机，选择使用了secure模式

例如我使用的三个区域的节点ip分别为

node1公网ip：112.74.74.190 私网ip:172.25.136.116

node2公网ip：8.142.166.129 私网ip:172.27.155.153

node3公网ip：47.108.183.202 私网ip:172.29.20.174









# 生成安全证书

需要在每个节点都生成certs目录

```
mkdir certs my-safe-directory
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

现在首先在node1上生成CA证书

```
cockroach cert create-ca --certs-dir=certs --ca-key=my-safe-directory/ca.key
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

使用该证书生成客户端证书

```
cockroach cert create-client root --certs-dir=certs --ca-key=my-safe-directory/ca.key
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

使用该证书为所有节点生成证书，注意，**先生成其他节点的密钥，再生成CA证书所在节点的密钥**，否则会使CA证书所在节点的密钥被覆盖，导致无法连接数据库

生成**node2**的密钥

```
cockroach cert create-node 8.142.166.129 172.27.155.153 --certs-dir=certs --ca-key=my-safe-directory/ca.key --overwrite
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

将node2的密钥及客户端证书发送给node2，此时生成的证书在node1的certs目录下

```
scp certs/* root@8.142.166.129:~/certs
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

node3同理

生成完node2，3后，生成node1的密钥即可进行数据库的启动阶段

# 启动集群

运行cockroach start命令启动数据库节点

node1：

```
cockroach start --certs-dir=certs --locality=cloud=aliyun,region=shenzhen,zone=sz --locality-advertise-addr=region=shenzhen@172.25.136.116 --advertise-addr=47.112.148.208 --join=172.25.136.116,8.142.166.129,47.108.183.202 --cache=.25 --max-sql-memory=.35 --background 
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

node2:

```
cockroach start --certs-dir=certs --locality=cloud=aliyun,region=zhangjiakou,zone=zjk --locality-advertise-addr=region=zhangjiakou@172.27.155.153 --advertise-addr=8.142.166.129 --join=112.74.74.190,172.27.155.153,47.108.183.202 --cache=.25 --max-sql-memory=.35 --background
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

node3:

```
cockroach start --certs-dir=certs --locality=cloud=aliyun,region=chengdu,zone=cd --locality-advertise-addr=region=chengdu@172.29.20.174 --advertise-addr=47.108.183.202 --join=112.74.74.190,8.142.166.129,47.108.183.202 --cache=.25 --max-sql-memory=.35  --background
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

其中各参数的含义见https://www.cockroachlabs.com/docs/v21.2/cockroach-start.html

此时屏幕会显示，正在等待init，或者加入某个集群的信息

我们还没有初始化集群，在任意一台节点进行cockroach init即可，例如在node3运行

```
cockroach init --certs-dir=certs --host=47.108.183.202
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

即初始化集群成功可以使用下列命令查看，当前集群的节点数量

```
cockroach node ls --certs-dir=certs --host=47.108.183.202
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

insecure模式就不需要这么麻烦了，直接把--cert-dir参数换成--insecure即可

现在就配置好了三节点跨域的CRDB，接下来就可以进入sql了

```
cockroach sql --certs-dir=certs --host=47.108.183.202
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

以上就是如何配置CRDB三节点跨域集群

参考https://www.cockroachlabs.com/docs/v21.2/deploy-cockroachdb-on-aws.html

下一篇介绍如何编译cockroach源码
