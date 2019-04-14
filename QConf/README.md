[TOC]

# QConf
1. 主要是解决长链接在zookeeper上
2. QConf 可以获取配置，然后供微服务获取本地的数据
3. QConf 实际上作为zookeeper一个客户端的使用者

## 编译
  115  yum install cmake
  129  yum install gcc
  133  yum install g++
  135  yum install gdbm gdbm-devel
  139  yum install gdbm-devel.i686 
  205  yum install aclocal
  206  yum install aclocal-1.14
  209  yum install aclocal
  401  yum install mvn
  406  yum install maven
  587  yum install makeinfo
  588  yum install texinfo

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

