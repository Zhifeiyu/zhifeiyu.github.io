---
title: Hadoop + Hbase 单机伪分布式安装配置
date: 2016-10-29 11:25:31
tags: [hadoop,hbase,大数据]
categories: [大数据]
---

## **环境**
centos7 + hadoop2.7.2 + jdk1.8 + hbase1.2.3
## **准备工作**
### 创建hadoop用户
``` shell
# su hadoop
# useradd -m hadoop -s /bin/bash   #创建新用户hadoop
# passwd hadoop    #创建密码
# vim /etc/sudoers  #赋予管理员权限
```
 找到 root ALL=(ALL) ALL 这行
 然后在这行下面增加一行内容：hadoop ALL=(ALL) ALL
![](http://oflrm5g9z.bkt.clouddn.com/Image%201.png)
### JAVA环境配置
### 安装SSH、配置SSH无密码登陆
``` shell
$ rpm -qa | grep ssh
```
![](http://oflrm5g9z.bkt.clouddn.com/Image%202.png)
返回结果如上图，则说明已经安装了ssh,不需要安装，否则需要通过yum进行安装。
```  shell
$ sudo yum install openssh-clients
$ sudo yum install openssh-server
```
 然后配置SSH无密码登陆
``` shell
$ cd ~/.ssh/                     # 若没有该目录，请先执行一次ssh localhost
$ ssh-keygen -t rsa              # 会有提示，都按回车就可以
$ cat id_rsa.pub >> authorized_keys  # 加入授权
$ chmod 600 ./authorized_keys    # 修改文件权限
```
执行如下命令测试一下 SSH 是否可用：
``` shell
$ ssh localhost
```
无需输入密码就可以直接登陆了。

## **安装Haoop**
### 下载hadoop-2.7.2.tar.gz包
### 解压并修改文件权限
``` shell
$ sudo tar -zxf ~/下载/hadoop-2.7.2.tar.gz -C /usr/local    # 解压到/usr/local中
$ cd /usr/local/
$ sudo ln -s hadoop-2.7.2 hadoop            # 创建hadoop软连接
$ sudo chown -R hadoop:hadoop ./hadoop        # 修改文件权限
```
## **配置环境变量**
``` shell
$ sudo vim /etc/profile
```
添加如下内容
``` vim 
# hadoop Env
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export HADOOP_OPTS="-Djava.library.path=$HADOOP_HOME/lib"
export HADOOP_CLASSPATH=${JAVA_HOME}/lib/tools.jar
```
### 测试hadoop是否可用
``` shell
[hadoop@osd01 root]$ hadoop version
Hadoop 2.7.2
Subversion https://git-wip-us.apache.org/repos/asf/hadoop.git -r e3496499ecb8d220fba99dc5ed4c99c8f9e33bb1
Compiled by jenkins on 2014-11-13T21:10Z
Compiled with protoc 2.5.0
From source with checksum 18e43357c8f927c0695f1e9522859d6a
This command was run using /usr/local/hadoop-2.7.2/share/hadoop/common/hadoop-common-2.7.2.jar
```
成功则会显示 如上的Hadoop 版本信息
## **Hadoop伪分布式配置**
### 修改配置文件
Hadoop 的配置文件位于 /usr/local/hadoop/etc/hadoop/ 中，伪分布式需要修改2个配置文件 core-site.xml 和 hdfs-site.xml 。（/*Hadoop的配置文件是 xml 格式，每个配置以声明 property 的 name 和 value 的方式来实现。*/）
- 修改配置文件 core-site.xml 
``` xml
<configuration>
    <property>
        <name>hadoop.tmp.dir</name>
        <value>file:/usr/local/hadoop/tmp</value>
        <description>Abase for other temporary directories.</description>
    </property>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>
```
- 修改配置文件 hdfs-site.xml
``` xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/name</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>file:/usr/local/hadoop/tmp/dfs/data</value>
    </property>
</configuration>
```
- 修改配置hadoop-env.sh的JAVA_HOME
```  sh
# The java implementation to use.
#export JAVA_HOME=${JAVA_HOME}
export JAVA_HOME=/usr/local/jdk-1.8.0_45
```
- 修改配置yarn-env.sh的JAVA_HOME
``` sh
# some Java parameters
export JAVA_HOME=/usr/local/jdk-1.8.0_45
if [ "$JAVA_HOME" != "" ]; then
  #echo "run java in $JAVA_HOME"
  JAVA_HOME=$JAVA_HOME
fi
```
### NameNode 的格式化
``` shell 
$ hadoop hdfs -format
```
成功的话，会看到 “successfully formatted” 和 “Exitting with status 0” 的提示，若为 “Exitting with status 1” 则是出错。
### 开启 NameNode 、DataNode 守护进程 和YARN
``` shell
$ start-dfs.sh
$ start-yarn.sh
```
通过jps 来判断是否成功启动
``` shell
[hadoop@osd01 root]$ jps
8195 NodeManager
8106 ResourceManager
7707 NameNode
7821 DataNode
7965 SecondaryNameNode
10382 Jps
```
若成功启动则会列出如下进程:  NameNode 、DataNode、SecondaryNameNode、NodeManager、ResourceManager
## **安装Hbase**
### 下载hbase-1.2.3-bin.tar.gz包
下载的hbase版本需要与安装的hadoop匹配，具体参考下面的链接
[hbase-config](https://hbase.apache.org/book.html#configuration)
### 解压并修改文件权限
``` shell
$ tar -zxvf hbase-1.2.3-bin.tar.gz -C /usr/local/    # 解压到/usr/local中
$ cd /usr/local/
$ sudo ln -s hbase-1.2.3 hbase
$ sudo chown -R hadoop:hadoop ./hbase        # 修改文件权限
```
### 配置环境变量
``` shell
$ sudo vim /etc/profile
```
添加如下内容
``` vim 
#hbase Env
export HBASE_HOME=/usr/local/hbase
export PATH=$PATH:$HBASE_HOME/bin
```
### 测试hadoop是否可用
``` shell
[hadoop@osd01 root]$ hbase version
HBase 1.2.3
Source code repository git://kalashnikov.att.net/Users/stack/checkouts/hbase.git.commit revision=bd63744624a26dc3350137b564fe746df7a721a4
Compiled by stack on Mon Aug 29 15:13:42 PDT 2016
From source with checksum 0ca49367ef6c3a680888bbc4f1485d18
```
成功则会显示如上的Hbase版本信息
### HBase伪分布式模式
#### 修改配置文件
- 修改hbase-env.sh
添加变量HBASE_CLASSPATH，并将路径设置为本机Hadoop安装目录下的conf目录（即{HADOOP_HOME}/conf）
``` vim
export JAVA_HOME=/usr/local/jdk-1.8.0_45
export HBASE_CLASSPATH=/usr/hadoop/conf 
export HBASE_MANAGES_ZK=true
```
- 修改hbase-site.xml 
修改hbase.rootdir，将其指向localhost(与hdfs的端口保持一致)，并指定HBase在HDFS上的存储路径。将属性hbase.cluter.distributed设置为true。假设当前Hadoop集群运行在伪分布式模式下，且NameNode运行在9000端口；
``` vim
<configuration>
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://localhost:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
</configuration>
```
### 启动HBase
完成以上操作后启动HBase，启动顺序：先启动Hadoop–>再启动HBase，关闭顺序：先关闭HBase–>再关闭Hadoop。
假设hadoop已经启动。
``` shell
[hadoop@osd01 hbase]$ start-hbase.sh
localhost: starting zookeeper, logging to /usr/local/hbase/bin/../logs/hbase-hadoop-zookeeper-osd01.out
starting master, logging to /usr/local/hbase/logs/hbase-hadoop-master-osd01.out
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option PermSize=128m; support was removed in 8.0
Java HotSpot(TM) 64-Bit Server VM warning: ignoring option MaxPermSize=128m; support was removed in 8.0
starting regionserver, logging to /usr/local/hbase/logs/hbase-hadoop-1-regionserver-osd01.out
[hadoop@osd01 hbase]$ jps
23189 SecondaryNameNode
28139 HQuorumPeer
22908 NameNode
23005 DataNode
23341 ResourceManager
28205 HMaster
28333 HRegionServer
23438 NodeManager
28815 Jps
```
### 进入shell模式
``` shell
[hadoop@osd01 hbase]$ hbase shell
2016-10-31 11:07:31,779 WARN  [main] util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
SLF4J: Class path contains multiple SLF4J bindings.
SLF4J: Found binding in [jar:file:/usr/local/hbase-1.2.3/lib/slf4j-log4j12-1.7.5.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: Found binding in [jar:file:/usr/local/hadoop-2.7.2/share/hadoop/common/lib/slf4j-log4j12-1.7.10.jar!/org/slf4j/impl/StaticLoggerBinder.class]
SLF4J: See http://www.slf4j.org/codes.html#multiple_bindings for an explanation.
SLF4J: Actual binding is of type [org.slf4j.impl.Log4jLoggerFactory]
HBase Shell; enter 'help<RETURN>' for list of supported commands.
Type "exit<RETURN>" to leave the HBase Shell
Version 1.2.3, rbd63744624a26dc3350137b564fe746df7a721a4, Mon Aug 29 15:13:42 PDT 2016

hbase(main):001:0> 
```
### 查看HDFS的HBase数据库文件
``` shell 
[hadoop@osd01 hbase]$ hadoop fs -ls /hbase
16/10/31 11:08:41 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
Found 7 items
drwxr-xr-x   - hadoop supergroup          0 2016-10-31 11:06 /hbase/.tmp
drwxr-xr-x   - hadoop supergroup          0 2016-10-31 11:06 /hbase/MasterProcWALs
drwxr-xr-x   - hadoop supergroup          0 2016-10-31 11:06 /hbase/WALs
drwxr-xr-x   - hadoop supergroup          0 2016-10-31 10:36 /hbase/data
-rw-r--r--   1 hadoop supergroup         42 2016-10-31 10:35 /hbase/hbase.id
-rw-r--r--   1 hadoop supergroup          7 2016-10-31 10:35 /hbase/hbase.version
drwxr-xr-x   - hadoop supergroup          0 2016-10-31 11:06 /hbase/oldWALs
```
## **hive 安装**
- - - 
[参考链接1](http://www.powerxing.com/install-hadoop-in-centos/)
[参考链接2](http://blog.csdn.net/andie_guo/article/details/44086389)

