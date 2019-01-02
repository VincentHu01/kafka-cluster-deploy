# 一、虚拟机设置
## 1. 准备三台centos7.4虚拟机
内存1g以上
## 2. 更改主机名
vim /etc/hosts <br>
修改内容如下： <br>
127.0.0.1  master.localdomain master <br>
::1  master.localdomain master <br>
192.168.65.130 master <br>
192.168.65.131 slave1 <br>
192.168.65.132 slave2 <br>
三台机器对应修改名称

## 3. 设置静态ip

vim /etc/sysconfig/network-scripts/ifcfg-eno16777736  <br>
修改如下： <br>
TYPE=Ethernet <br>
BOOTPROTO=static  --此处修改 <br>
DEFROUTE=yes <br>
IPV4_FAILURE_FATAL=no <br>
IPV6INIT=no <br>
IPV6_AUTOCONF=no <br>
IPV6_DEFROUTE=no <br>
IPV6_FAILURE_FATAL=no <br>
NAME=eno16777736 <br>
UUID=88dd1319-9391-4fe9-a80a-aa6d18e5d96d <br>
DEVICE=eno16777736 <br>
ONBOOT=yes <br>
PEERDNS=yes <br>
PEERROUTES=yes <br>
IPV6_PEERDNS=yes <br>
IPV6_PEERROUTES=yes <br>
--以下增加 <br>
IPADDR=192.168.65.130 <br>
NETMASKS=225.225.225.0 <br>
GATEWAY=192.168.65.2 <br>
DNS1=8.8.8.8 <br>
其余机器对应修改，然后重启所有机器

## 4. 增加用户和用户组
useradd -g hdp hdp

## 5. 设置ssh免密互通
### 5.1
在三台机器上执行： <br>
su hdp <br>
cd <br>
ssh-keygen -t rsa <br>
提示输入密码，一直回车 <br>
### 5.2
在slave1和slave2执行以下操作：    <br>
scp .ssh/id_rsa.pub hdp@master:～/id_rsa.slave1.pub   <br>
scp .ssh/id_rsa.pub hdp@master:～/id_rsa.slave2.pub   <br>
### 5.3
在master根目录执行：   <br>
cat id_rsa.*.pub >> .ssh/authorized_keys   <br>
scp .ssh/authorized_keys hdp@slave1:～/.ssh   <br>
scp .ssh/authorized_keys hdp@slave2:～/.ssh   <br>

# 二、zookeeper集群搭建
## 1. 软件下载
mkdir -p /data/software  <br>
官网下载zookeeper-3.4.9.tar.gz至/data/software  <br>
解压：  <br>
tar xzvf zookeeper-3.4.9.tar.gz  <br>
## 2. 修改conf目录下的配置文件
vim /data/hdp/zookeeper/zookeeper-3.4.9/conf/zoo.cfg  <br>
修改内容如下：  <br>
tickTime=2000  <br>
initLimit=10  <br>
syncLimit=5  <br>
dataDir=/data/hdp/zookeeper/data/zk  <br>
clientPort=2181  <br>

quorumListenOnAllIPs=true  <br>

server.1=master:2887:3887  <br>
server.2=slave1:2888:3888  <br>
server.3=slave2:2889:3889  <br>
如果不共用网卡，则端口统一设为:2888:3888  <br>
如果共用网卡，则要设置quorumListenOnAllIPs=true  <br>

## 3. scp zookeeper文件夹到其他两台机器
scp -r /data/hdp/zookeeper/zookeeper-3.4.9 hdp@slave1:/data/hdp/zookeeper  <br>
scp -r /data/hdp/zookeeper/zookeeper-3.4.9 hdp@slave2:/data/hdp/zookeeper  <br>

## 4. myid设置
在三台机器分别执行以下命令  <br>
mkdir -p /data/hdp/zookeeper/data/zk  <br>
然后在zk文件夹下新建文件myid，往三台机器的myid文件中分别写入1,2,3  <br>
## 5. 启动zookeeper集群
三台机器分别执行：  <br>
/data/hdp/zookeeper/zookeeper-3.4.9/bin/zkServer.sh start  <br>
然后通过以下命令查看节点状态  <br>
/data/hdp/zookeeper/zookeeper-3.4.9/bin/zkServer.sh status  <br>
比如：某一台为leader节点：  <br>
[hdp@slave1 ~]$ /data/hdp/zookeeper/zookeeper-3.4.9/bin/zkServer.sh status  <br>
ZooKeeper JMX enabled by default  <br>
Using config: /data/hdp/zookeeper/zookeeper-3.4.9/bin/../conf/zoo.cfg  <br>
Mode: leader  <br>
另外两台为follower节点  <br>
说明zk集群启动正常  <br>

# 三、kafka集群搭建
## 1. 软件下载

官网下载kafka_2.11-1.0.0.tgz至/data/software  <br>
解压至/data/hdp/kafka/  <br>

scp至其余两台机器  <br>
scp -r /data/hdp/kafka/kafka_2.11-1.0.0 hdp@slave1:/data/hdp/kafka  <br>
scp -r /data/hdp/kafka/kafka_2.11-1.0.0 hdp@slave2:/data/hdp/kafka  <br>

## 2. 修改配置文件
mkdir -p /data/hdp/kafka/logs  <br>

在三台机器分别修改  <br>
vim /data/hdp/kafka/kafka_2.11-1.0.0/config/server.properties  <br>
修改内容如下，前后有空行为修改处  <br>
broker.id三台机器分别为0,1,2   <br>

broker.id=0  <br>

listeners=PLAINTEXT://192.168.65.130:9092  <br>

host.name=master  <br>

num.network.threads=3  <br>
num.io.threads=8  <br>
socket.send.buffer.bytes=102400  <br>
socket.receive.buffer.bytes=102400  <br>
socket.request.max.bytes=104857600  <br>

log.dirs=/data/hdp/kafka/logs  <br>

num.partitions=1  <br>
num.recovery.threads.per.data.dir=1  <br>
offsets.topic.replication.factor=1  <br>
transaction.state.log.replication.factor=1  <br>
transaction.state.log.min.isr=1  <br>
log.retention.hours=168  <br>
log.segment.bytes=1073741824  <br>
log.retention.check.interval.ms=300000  <br>

zookeeper.connect=192.168.65.130:2181,192.168.65.131:2181,192.168.65.132:2181  <br>

zookeeper.connection.timeout.ms=18000  <br>
group.initial.rebalance.delay.ms=0  <br>

## 3. 启动kafka集群
在三台机器分别执行  <br>
/data/hdp/kafka/kafka_2.11-1.0.0/bin/kafka-server-start.sh /data/hdp/kafka/kafka_2.11-1.0.0/config/server.properties &   <br>

