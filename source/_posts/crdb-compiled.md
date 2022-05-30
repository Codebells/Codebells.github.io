---
title: CRDB源码编译
date: 2022-03-26 16:42:06
categories: 
  - [database]
  - [system]
tags: [system,database]
category_bar: true
excerpt: 在上篇记录了如何初步使用CRDB，但是想要更多了解CRDB以及为了完成CRDB客户端没实现（可能我没找到）的某些功能，就需要自行阅读源码进行更改并进行编译
---

 在上篇记录了如何初步使用CRDB，但是想要更多了解CRDB以及为了完成CRDB客户端没实现（可能我没找到）的某些功能，就需要自行阅读源码进行更改并进行编译，在更改源码过程中，因为怕影响CRDB性能，基本不会删除某些关键性代码，因为我需要的是查看CRDB测试时的abort率，以及事务的生命周期，以及每个阶段所消耗的时间，所以只对其做出了增加时间戳以及记录的代码，并不会影响数据库性能。下面，是我在编译过程中记录的语句。该说不说，CRDB的官方编译指南是真简洁，官方的[指南1](https://wiki.crdb.io/wiki/spaces/CRDB/pages/181338446/Getting+and+building+CockroachDB+from+source) 还有[指南2](https://www.cockroachlabs.com/docs/v21.1/install-cockroachdb-linux#build-from-source) 仅作参考

首先还是说初始环境配置，为阿里云ubuntu 20.04,配置编译CRDB 21.2.4

按照官方给的，需要配置

> - The tools needed to build CockroachDB from scratch:
>   - A C++ compiler that supports C++11. Note that GCC prior to 6.0 doesn't work due to [48891 – std functions conflicts with C functions when building with c++0x support (and using namespace std)](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=48891)
>   - The standard C/C++ development headers on your system.
>   - On GNU/Linux, the terminfo development libraries, which may be part of a ncurses development package (e.g. `libncurses-dev` on Debian/Ubuntu, but `ncurses-devel` on CentOS).
>   - A Go environment with a recent 64-bit version of the toolchain. Note that the Makefile enforces the specific version required, as it is updated frequently.
>   - Git 1.9+, we recommend v2.20+
>   - Bash (4+ is preferred)
>   - GNU Make (4.2+ is known to work)
>   - CMake 3.20.* (on Macs 3.21+ are also known to work)
>   - Autoconf 2.68+
>   - NodeJS 12.x and Yarn 1.7+
>   - Yacc or Bison
>   - [Bazelisk](https://github.com/bazelbuild/bazelisk) (you can use Bazel directly instead of Bazelisk, although Bazelisk makes sure you’re using the same version that we use in CI)
>   - If you're running macOS, a full installation of XCode is necessary.

有些东西在aliyun的ubuntu中是配置好的，所以在下面的指令中没有这些环境配置语句，各位见谅

# 安装编译所需依赖

首先sudo apt update一下

**安装必要的一些软件**，如c++,zip,python

```
sudo apt-get update
sudo apt-get install pkg-config zip g++ zlib1g-dev unzip python -y
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**安装bazel以及bazelisk**，这个是管理编译的，需要安装，因为apt get太慢了，我直接下载好之后用ftp传输到服务器，我下载的为bazel 4.0各位可以根据需要自行更换(最好不要，我换过其他版本，不好使)

```
chmod +x bazel-4.0.0-installer-linux-x86_64.sh
./bazel-4.0.0-installer-linux-x86_64.sh --user
npm install -g @bazel/bazelisk
export PATH="$PATH:$HOME/bin"
source /etc/profile
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**安装go环境**

```
wget https://studygolang.com/dl/golang/go1.17.7.linux-amd64.tar.gz
tar -C /usr/local -zxvf go1.17.7.linux-amd64.tar.gz
vim /etc/profile
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin
export GOPATH=$HOME/works
//添加上述三行到文件末尾，wq后source一下
source /etc/profile
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**安装git，bison以及\**ncurses\****

我记得是ubuntu和centos这里不一样，大家自行寻找centos的ncurses的替代品

```
sudo apt-get install libncurses5-dev -y
sudo apt-get install libssl-dev
sudo apt install git -y
sudo apt install bison -y
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

**安装cmake**

```
wget https://cmake.org/files/v3.20/cmake-3.20.2.tar.gz 
tar xzvf cmake-3.20.2.tar.gz
cd cmake-3.20.2
rm -f CMakeCache.txt
./bootstrap
make
make install
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

这个CMakecache.txt我记得好像是不删除会报错，各位根据情况删除

**安装nodejs和yarn**

```
curl -sL https://deb.nodesource.com/setup_12.x | sudo -E bash -
sudo apt-get install -y nodejs
curl -sS https://dl.yarnpkg.com/debian/pubkey.gpg | sudo apt-key add -
echo "deb https://dl.yarnpkg.com/debian/ stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
apt install -y yarn
apt install --no-install-recommends yarn -y
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

到这已经安装完成所有依赖，各位可以在安装过程中输入 --version或者-v验证是否安装成功

可以下载CRDB源码进行编译了

# CRDB源码获取及编译

我习惯创建一个works目录进行存放代码，各位可以根据习惯自行调整

```
cd ~
mkdir works
cd works
wget -qO- https://binaries.cockroachdb.com/cockroach-v21.2.4.src.tgz | tar xvz
mv cockroach-v21.2.4/* .
rm cockroach-v21.2.4 -rf
cd src/github.com/cockroachdb/cockroach
make clean 
make build
make install
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

到这已经编译完成，各位可以输入cockroach --version验证是否编译成功

祝大家一切顺利！
