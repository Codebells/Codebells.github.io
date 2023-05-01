---
title: opengauss-odbc安装
date: 2023-04-23 20:09:27
categories: 
  - [database]
  - [system]
tags: [system,database]
category_bar: true
---

# Linux下配置数据源

[Linux下配置数据源 (opengauss.org)](https://docs.opengauss.org/zh/docs/2.0.0/docs/Developerguide/Linux下配置数据源.html)

1. 下载unixODBC

2. 安装unixODBC

   ```shell
   tar zxvf unixODBC-2.3.0.tar.gz
   cd unixODBC-2.3.0
   #修改configure文件(如果不存在，那么请修改configure.ac)，找到LIB_VERSION
   #将它的值修改为"1:0:0"，这样将编译出*.so.1的动态库，与psqlodbcw.so的依赖关系相同。
   vim configure
   
   ./configure --enable-gui=no #如果要在鲲鹏服务器上编译，请追加一个configure参数： --build=aarch64-unknown-linux-gnu 
   make
   #安装可能需要root权限
   make install
   cd ..
   tar zxvf openGauss-2.0.0-ODBC.tar.gz -C /usr/local/lib
   cd /usr/local/lib
   cp lib/* .
   cp odbc/lib/* .
   yum install libtool-ltdl
   ```

3. 在“/usr/local/etc/odbcinst.ini”文件中追加以下内容

   ```
   [GaussMPP]
   Driver64=/usr/local/lib/psqlodbcw.so
   setup=/usr/local/lib/psqlodbcw.so
   ```

   在“/usr/local/etc/odbc.ini ”文件中追加以下内容

   ```
   [MPPODBC]
   Driver=GaussMPP
   Servername=10.10.0.13（数据库Server IP）
   Database=postgres  （数据库名）
   Username=omm  （数据库用户名）
   Password=  （数据库用户密码）
   Port=8000 （数据库侦听端口）
   Sslmode=allow
   ```

4. 生成SSL证书

   ```
   --假设用户为omm已存在,搭建CA的路径为test
   --以root用户身份登录Linux环境,切换到用户omm
   mkdir test
   cd /etc/pki/tls
   cp openssl.cnf ~/test
   cd ~/test
   mkdir ./demoCA ./demoCA/newcerts ./demoCA/private
   chmod 777 ./demoCA/private
   echo '01'>./demoCA/serial
   touch ./demoCA/index.txt
   ```

5. 修改openssl.cnf配置文件中的参数

   ```
   dir  = ./demoCA
   default_md      = sha256
   ```

   CA环境搭建完成

6. 生成根私钥

   ```
   openssl genrsa -aes256 -out demoCA/private/cakey.pem 2048
   Test@123
   Test@123
   ```

7. 生成根证书请求文件

   ```
   openssl req -config openssl.cnf -new -key demoCA/private/cakey.pem -out demoCA/careq.pem
   Test@123
   
   Country Name (2 letter code) [XX]:CN
   State or Province Name (full name) []:liaoning
   Locality Name (eg, city) [Default City]:shenyang
   Organization Name (eg, company) [Default Company Ltd]:neu
   Organizational Unit Name (eg, section) []:neu      
   Common Name (eg, your name or your server's hostname) []:neu
   Email Address []:
   
   Please enter the following 'extra' attributes
   to be sent with your certificate request
   A challenge password []:
   An optional company name []:
   ```

8. 生成自签发根证书。

   ```
   --生成根证书时，需要修改openssl.cnf文件，设置basicConstraints=CA:TRUE
   vi openssl.cnf
   openssl ca -config openssl.cnf -out demoCA/cacert.pem -keyfile demoCA/private/cakey.pem -selfsign -infiles demoCA/careq.pem
   Test@123
   ```

9. 生成服务端证书私钥

   ```
   openssl genrsa -aes256 -out server.key 2048
   Test@123
   Test@123
   openssl req -config openssl.cnf -new -key server.key -out server.req
   ```

10. 生成服务端证书

    ```
    --生成服务端/客户端证书时，修改openssl.cnf文件，设置basicConstraints=CA:FALSE
    vi openssl.cnf
    --修改demoCA/index.txt.attr中属性为no。
    vi demoCA/index.txt.attr
    openssl ca  -config openssl.cnf -in server.req -out server.crt -days 3650 -md sha256
    gs_guc encrypt -M server -K Test@123 -D ./
    ```

11. 客户端证书私钥生成

    ```
    openssl genrsa -aes256 -out client.key 2048
    openssl req -config openssl.cnf -new -key client.key -out client.req 
    openssl ca -config openssl.cnf -in client.req -out client.crt -days 3650 -md sha256
    gs_guc encrypt -M server -K Test@123 -D ./
    
    
    ```

12. 去掉私钥密码保护

    **openssl** **rsa** **-in** **server**.key **-out** **server**.key

    openssl rsa -in client.key -out client.key

将客户端密钥转化为DER格式，方法如下：
openssl pkcs8 -topk8 -outform DER -in client.key -out client.key.pk8 -nocrypt





gs_guc reload -N NodeName -I all -c "listen_addresses='localhost,0.0.0.0'"

gs_guc reload -N all -I all -h "host all jack 39.98.81.111/32 sha256"

# BUG

启动失败

```shell
[omm@iZ8vbc1ycxkzrasostw66mZ ~]$ gs_om -t start
Starting cluster.
=========================================
=========================================
[GAUSS-53600]: Can not start the database, the cmd is source /home/omm/.bashrc; python3 '/opt/huawei/install/om/script/local/StartInstance.py' -U omm -R /opt/huawei/install/app -t 300 --security-mode=off,  Error:
[GAUSS-51400] : Failed to execute the command: source /home/omm/.bashrc; python3 '/opt/huawei/install/om/script/local/StartInstance.py' -U omm -R /opt/huawei/install/app -t 300 --security-mode=off. Error:
[FAILURE] node1_hostname:
..
```

更改主机名

hostnamectl set-hostname node1_hostname



sudo yum install libtool-ltdl

