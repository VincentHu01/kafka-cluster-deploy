# 一、虚拟机设置见kafka-zookeeper-deploy.md
3 hosts: master, slave1, slave2

# 二、 hadoop配置
cd /data/hdp/hadoop/hadoop-2.7.7/
cd etc/hadoop
## 2.1  vim core-site.xml
```xml
<configuration>
    <property>
　　　　<name>fs.default.name</name>
　　　　<value>hdfs://master:9000</value>
　　</property>
　　<property>
　　　　<name>hadoop.tmp.dir</name>
　　　　<value>file:/data/hdp/hadoop/hadoop-2.7.7/dfs/tmp</value>
　　</property>
</configuration>
```

## 2.2 vim hdfs-site.xml
```xml
<configuration>
    <property>
　　　　<name>dfs.replication</name>
　　　　<value>2</value>
　　</property>
　　<property>
　　　　<name>dfs.namenode.name.dir</name>
　　　　<value>file:/data/hdp/hadoop/hadoop-2.7.7/dfs/name</value>
　　</property>
　　<property>
　　　　<name>dfs.datanode.data.dir</name>
　　　　<value>file:/data/hdp/hadoop/hadoop-2.7.7/dfs/data</value>
　　</property>
　　<property>
　　　　<name>dfs.namenode.secondary.http-address</name>
　　　　<value>master:9001</value>
　　</property>
</configuration>

```

## 2.3 vim mapred-site.xml　　
```xml
<configuration>
　　<property>
　　　　<name>mapreduce.framework.name</name>
　　　　<value>yarn</value>
　　</property>
</configuration>
```

## 2.4 vim yarn-site.xml
```xml
<configuration>

    <!-- Site specific YARN configuration properties -->
    <property>
　　　　<name>yarn.resourcemanager.hostname</name>
　　　　<value>master</value>
　　</property>
　　<property>
　　　　<name>yarn.nodemanager.aux-services</name>
　　　　<value>mapreduce_shuffle</value>
　　</property>
　　<property>
　　　　<name>yarn.log-aggregation-enable</name>
　　　　<value>true</value>
　　</property>
　　<property>
　　　　<name>yarn.log-aggregation.retain-seconds</name>
　　　　<value>604800</value>
　　</property>
</configuration>
```
## 2.5 vim slaves
slave1  <br>
slave2 <br>

## 2.6 设置环境变量
vim /etc/profile <br>
export HADOOP_HOME=/data/hdp/hadoop/hadoop-2.7.7 <br>
export HADOOP_INSTALL=$HADOOP_HOME <br>
export HADOOP_MAPRED_HOME=$HADOOP_HOME <br>
export HADOOP_COMMON_HOME=$HADOOP_HOME <br>
export HADOOP_HDFS_HOME=$HADOOP_HOME <br>
export YARN_HOME=$HADOOP_HOME <br>
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native <br>
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin <br>


## 2.7 完成1-6配置,将hadoop-2.7.7文件夹和/etc/profile文件分发至slave1和slave2，source /etc/profile

# 三. 格式化namenode
hadoop namenode -format

# 四. 启动hadoop集群
sbin/start-all.sh
如果提示找不到JAVA_HOME，则需要在hadoop-env.sh等文件下添加设置JAVA_HOME

