---
title: zeromq
date: 2022-11-12 20:34:30
categories: tech
tags: 日常
category_bar: true
---

zmq安装

我安装的是cppzmq 4.7.1 libzmq 4.3.3

```shell
sudo apt-get install libtool pkg-config build-essential autoconf automake
#install libsodium
git clone https://gitee.com/Codebells/libsodium.git
cd libsodium
./autogen.sh -s
./configure && make check
cd ..
make install
sudo ldconfig
#install libzmq
unzip libzmq-4.3.3.zip
./autogen.sh
./configure && make check
sudo make install
sudo ldconfig
cd ..
#install cppzmq
unzip cppzmq-4.7.1.zip
cd cppzmq.4.7.1/
mkdir build
cd build
cmake ..
make -j4 install
```

