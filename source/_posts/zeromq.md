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
#或者直接拷贝zmq.cpp
#git clone https://github.com/zeromq/cppzmq.git
#cd cppzmq
#sudo cp zmq.hpp /usr/local/include/
#cd ..
```

测试demo server

```cpp
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <assert.h>
#include <zmq.h>
int main (void)                                                         {                                                                          //  Socket to talk to clients
    void *context = zmq_ctx_new ();                                         void *responder = zmq_socket (context, ZMQ_REP);                         int rc = zmq_bind (responder, "tcp://*:5555");                           assert (rc == 0);                                                       while (1) {                                                                 char buffer [10];                                                       zmq_recv (responder, buffer, 10, 0);                                     printf ("Received Hello\n");                                             sleep (1);          //  Do some 'work'
    	zmq_send (responder, "World", 5, 0);                                 }                                                                       return 0;                                                           }
```

测试 client.cpp

```cpp
#include <zmq.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
int main (void)
{
    printf ("Connecting to hello world server…\n");
    void *context = zmq_ctx_new ();
    void *requester = zmq_socket (context, ZMQ_REQ);
    zmq_connect (requester, "tcp://localhost:5555");
    int request_nbr;
    for (request_nbr = 0; request_nbr != 10; request_nbr++) {
        char buffer [10];
        printf ("Sending Hello %d…\n", request_nbr);
        zmq_send (requester, "Hello", 5, 0);
        zmq_recv (requester, buffer, 10, 0);
        printf ("Received World %d\n", request_nbr);
    }
    zmq_close (requester);
    zmq_ctx_destroy (context);
    return 0;
}
```

```
g++ zserver.cpp -o server -lzmq 
g++ zclient.cpp -o client -lzmq
```







