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
   yum install libtool-ltdl -y
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
   Servername=47.92.142.44
   Database=postgres 
   Username=jack
   Password=Test@123
   Port=26000
   Sslmode=allow
   ```

4. gs_guc reload -N NodeName -I all -c "listen_addresses='localhost,0.0.0.0'"

   gs_guc reload -N all -I all -h "host all jack 39.98.81.111/32 sha256"

5. 在客户端配置环境变量。

   vim ~/.bashrc

   export LD_LIBRARY_PATH=/usr/local/lib/:$LD_LIBRARY_PATH
   export ODBCSYSINI=/usr/local/etc
   export ODBCINI=/usr/local/etc/odbc.ini

   source ~/.bashrc

   

pg_hba.conf

```
# "local" is for Unix domain socket connections only
local   all             all                                     trust
# IPv4 local connections:
host    all             all             127.0.0.1/32            trust
host    all    all    47.92.209.140/32    sha256
host    all    all    172.31.16.64/32    sha256
host    all    all    0.0.0.0/0    sha256
host    all    all    0.0.0.0/0    trust
# IPv6 local connections:
host    all             all             ::1/128                 trust
```

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

# 卸载Opengauss

hostnamectl set-hostname node1_hostname
su - omm

gs_uninstall --delete-data
#切换到root用户下
userdel -r omm

rm -rf /opt/huawei
rm -rf /opt/software
rm -rf /root/gauss_om

# 预安装

export LD_LIBRARY_PATH=/opt/software/openGauss/script/gspylib/clib:$LD_LIBRARY_PATH
cd /opt/software/openGauss/script
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8

./gs_preinstall -U omm -G dbgrp -X /tmp/clusterconfig.xml





# 安装

chmod 777 -R /tmp
chown -R omm:dbgrp /tmp
\cp -rf  /tmp/openGauss-server/simpleInstall/ /tmp/openGauss-server/mppdb_temp_install/
su - omm
rm -rf /tmp/openGauss-server/mppdb_temp_install/data/
cd /tmp/openGauss-server/mppdb_temp_install/simpleInstall/
sh install.sh -w "Pengqi1998" -p 26000



cp /tmp/postgresql.conf /tmp/openGauss-server/mppdb_temp_install/data/single_node/
cp /tmp/mot.conf /tmp/openGauss-server/mppdb_temp_install/data/single_node/
cp /tmp/pg_hba.conf /tmp/openGauss-server/mppdb_temp_install/data/single_node/
gs_ctl restart -D /tmp/openGauss-server/mppdb_temp_install/data/single_node -Z single_node



gsql -d postgres -p 26000
ALTER ROLE omm IDENTIFIED BY 'pengqi.1998' REPLACE 'Pengqi1998';
create user jack password 'Test@123';
grant usage on foreign server mot_server to jack;
GRANT ALL PRIVILEGES ON DATABASE postgres to jack;
ALTER ROLE jack CREATEDB;
GRANT ALL PRIVILEGES To jack;





### 安装

```
hostnamectl set-hostname node1_hostname

yum install -y libaio-devel flex bison ncurses-devel glibc-devel patch lsb_release redhat-lsb readline-devel bzip2
yum install -y python36
yum install -y bzip2 libaio-devel flex bison ncurses-devel glibc-devel patch
yum install -y net-tools tar lrzsz
systemctl disable firewalld.service
systemctl stop firewalld.service
systemctl stop firewalld.service
sed -i 's/^SELINUX=.*/SELINUX=disabled/' /etc/selinux/config
setenforce 0
if [ "$LANG" != "en_US.UTF-8" ];then
export LANG=en_US.UTF-8
echo export LANG=en_US.UTF-8 >> /etc/profile
fi
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
swapoff -a
ifconfig
ifconfig ens192 mtu 8192
sed -i 's/^Banner .*/Banner none/' /etc/selinux/config
sed -i 's/^#PermitRootLogin .*/PermitRootLogin yes/' /etc/selinux/config
sed -i 's/^PermitRootLogin no/PermitRootLogin yes/' /etc/selinux/config
systemctl restart sshd
echo 5 > /proc/sys/net/ipv4/tcp_retries1
echo 5 > /proc/sys/net/ipv4/tcp_syn_retries
echo 10 > /proc/sys/net/sctp/path_max_retrans
echo 10 > /proc/sys/net/sctp/max_init_retransmits
echo net.ipv4.tcp_retries1 = 5 >>/etc/sysctl.conf
echo net.ipv4.tcp_syn_retries = 5 >>/etc/sysctl.conf
echo net.sctp.path_max_retrans = 10 >>/etc/sysctl.conf
echo net.sctp.max_init_retransmits = 10 >>/etc/sysctl.conf
yum install -y ntp
systemctl start ntpd && systemctl enable ntpd


yum - y install openssl-devel
cd /tmp
vim cluster_conf.xml
chmod 777 cluster_conf.xml


#omm 卸载
gs_uninstall --delete-data


#root卸载
userdel -r omm
rm -rf /tmp/huawei
rm -rf /tmp/software
rm -rf /root/gauss_om






cd /opt
rm -rf *
mkdir /opt/huawei
chmod -R 777 /opt/huawei
mkdir -p /opt/software/openGauss
chmod -R 777  /opt/software
cd /opt/software/openGauss
cd /usr/bin/

rm -f python2
mv python python2.6.ori
ln -s python2.7 python2
ln -s /usr/bin/python3.6 /usr/bin/python

cd /opt
mkdir test
cd test
wget https://opengauss.obs.cn-south-1.myhuaweicloud.com/2.0.1/x86/openGauss-2.0.1-CentOS-64bit-all.tar.gz
tar -xzvf openGauss-2.0.1-CentOS-64bit-all.tar.gz 
cp openGauss-2.0.1-CentOS-64bit-om.tar.gz /opt/software/openGauss
cd /tmp/merge-200-Taas-mot
sh build.sh -m release -3rd /tmp/binarylibs/ -pkg


cd output
cp * /opt/software/openGauss/
tar -xzvf  openGauss-2.0.1-CentOS-64bit-om.tar.gz



# 一次性
export LD_LIBRARY_PATH=/opt/software/openGauss/script/gspylib/clib:$LD_LIBRARY_PATH
cd /opt/software/openGauss/script
export LANG=en_US.UTF-8
export LANGUAGE=en_US.UTF-8


./gs_preinstall -U omm -G dbgrp -X /tmp/cluster_conf.xml

su - omm
gs_install -X /tmp/cluster_conf.xml

gsql -d postgres -p 26000
ALTER ROLE omm IDENTIFIED BY 'pengqi.1998' REPLACE 'Pengqi1998';
create user jack password 'Test@123';
grant usage on foreign server mot_server to jack;
GRANT ALL PRIVILEGES ON DATABASE postgres to jack;
ALTER ROLE jack CREATEDB;
GRANT ALL PRIVILEGES To jack;




vi /tmp/huawei/install/data/db1/postgresql.conf
enable_incremental_checkpoint = on
checkpoint部分需要改为off
开启关闭服务：gs_om -t start/stop
进入用户：su - omm 
进入数据库：gsql -d postgres -p 26000
退出数据库：ctrl+d
退出用户：exit

CREATE Foreign TABLE IF NOT EXISTS taas_odbc(c_sk INTEGER , c_name VARCHAR(32), c_tid VARCHAR(32));
select count(*) from taas_odbc;
insert into taas_odbc values(1,'123','qwe');


```

/opt/huawei/install/data/dn/postgresql.conf

```
listen_addresses = '*'	
enable_incremental_checkpoint = off
```

/opt/huawei/install/data/dn/pg_hba.conf
