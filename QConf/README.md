[TOC]

# QConf
1. 主要是解决长链接在zookeeper上
2. QConf 可以获取配置，然后供微服务获取本地的数据
3. QConf 实际上作为zookeeper一个客户端的使用者

## 编译
yum install cmake
yum install gcc
yum install g++
yum install gdbm gdbm-devel
yum install gdbm-devel.i686 
yum install aclocal
yum install aclocal-1.14
yum install aclocal
yum install mvn
yum install maven
yum install makeinfo
yum install texinfo

  cd /QCfonf
  cd deps/qdbm;./configure --prefix=/root/workspace/qconf/QConf/deps/gdbm/_install
  mkdir build
  cd build
  cmake
  make
  make install
  cd /usr/local/qconf/
  ./bin/agent_qconf.sh start

  注意，prefix必须配置到deps/gdbm下，否则make qconf 的时候，编译不过



# 配置
  zookeeper.test=172.16.250.220:2181,172.16.250.221:2181,172.16.250.222:2181

