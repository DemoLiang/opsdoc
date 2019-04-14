[TOC]

## 配置
```
server.1=172.16.250.220:2888:3888
server.2=172.16.250.221:2888:3888
server.3=172.16.250.222:2888:3888

```
## 启动集群
./bin/zkServer.sh restart

## 启动zkui
nohup java -jar zkui-2.0-SNAPSHOT-jar-with-dependencies.jar &
