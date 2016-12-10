---
layout: post
title:  "Hadoop Cluster 2.7.3 Installation on CentOS 7 in basic version"
date:   2016-12-10 09:00:00
author: Unskilled Coder
categories: hadoop
---

## Introduction

This posts will give all related detail in how to setup a Hadoop cluster on CentOS linux system. Before you read this article, I will assume that you already have all basic conceptions about Hadoop and Linux operating system.
<br /><br />

## Architecture

| IP Address      | Hostname    | Role                                     |
|:----------------|:------------|:-----------------------------------------|
| 192.168.171.132 | master      | NameNode, ResourceManager                |
| 192.168.171.133 | slave1      | SecondaryNameNode, DataNode, NodeManager |
| 192.168.171.134 | slave2      | DataNode, NodeManager                    |

<br />

## 1. CentOS setup

### 1.1. install necessary packages for OS

We pick up CentOS minimal IOS as our installation prototype, once the system installation completed, we need to install 2 things: <br />

```bash
sudo install -y net-tools
sudo install -y openssh-server
```
*The first line is to install ifconfig, while the second one is to be able to be ssh login by remote peer.*<br />

### 1.2. setup hostname for all nodes

This step is optional, but important for better self-identify while you use same username login to different nodes. 

```bash
sudo hostnamectl set-hostname master
```
*ex: at <b>master</b> node*
re-login to check the effect

### 1.3. setup jdk for all nodes
*install jdk from oracle official website*
```bash
sudo wget --header "Cookie: oraclelicense=accept-securebackup-cookie" http://download.oracle.com/otn-pub/java/jdk/8u112-b14/jdk-8u112-linux-x64.rpm
sudo yum localinstall -y jdk-8u112-linux-x64.rpm
sudo rm jdk-8u112-linux-x64.rpm
```
*add java.sh under /etc/profile.d/* <br />
java.sh content:
```bash
export JAVA_HOME=/usr/java/jdk1.8.0_112
export JRE_HOME=/usr/java/jdk1.8.0_112/jre
export CLASSPATH=$JAVA_HOME/lib:.
export PATH=$PATH:$JAVA_HOME/bin
```
re-login, and you'll find all environment variables, and java is well installed.
*Approach to verification:*
```bash
java -version
ls $JAVA_HOME
ls $PATH
```
if the java version goes wrong, you can 
```bash
sudo alternatives --config java
```
then choose a correct version.

### 1.4. setup user and user group on all nodes
```bash
sudo groupadd hadoop
sudo useradd -d /home/hadoop -g hadoop hadoop
passwd hadoop
```

### 1.5. modify hosts file for network inter-recognition on all nodes
```bash
echo '192.168.171.132 master' >> /etc/hosts
echo '192.168.171.133 slave1' >> /etc/hosts
echo '192.168.171.134 slave2' >> /etc/hosts
```
check the recognition works:
```bash
ping master
ping slave1
ping slave2
```

### 1.6. setup ssh no password login on all nodes
```bash
su - hadoop
ssh-keygen -t rsa
ssh-copy-id master
ssh-copy-id slave1
ssh-copy-id slave2
```
now you can ssh login to all 3 nodes without passwd, please have a try to check it out.
<br /> <br />

## 2. Hadoop Setup

P.S. the following operations happens on a single node, let's say, master. In addition, we'll login as user *hadoop* to finish all operations.
```bash
su - hadoop
```

### 2.1. Download and untar on the file system.
```bash
wget http://mirrors.sonic.net/apache/hadoop/common/hadoop-2.7.3/hadoop-2.7.3.tar.gz
untar -zxvf hadoop-2.7.3.tar.gz
rm hadoop-2.7.3.tar.gz
chmod 775 hadoop-2.7.3
```

### 2.2. Add environment variables for hadoop
append following content onto ~/.bashrc
```bash
export HADOOP_HOME=/home/hadoop/hadoop-2.7.3
export HADOOP_INSTALL=$HADOOP_HOME
export HADOOP_MAPRED_HOME=$HADOOP_HOME
export HADOOP_COMMON_HOME=$HADOOP_HOME
export HADOOP_HDFS_HOME=$HADOOP_HOME
export YARN_HOME=$HADOOP_HOME
export HADOOP_COMMON_LIB_NATIVE_DIR=$HADOOP_HOME/lib/native
export PATH=$PATH:$HADOOP_HOME/sbin:$HADOOP_HOME/bin
```
then make these variables effect:
```bash
source ~/.bashrc
```

### 2.3. Add slave node hostnames into *slaves* file
```bash
echo slave1 > $HADOOP_HOME/etc/hadoop/slaves
echo slave2 >> $HADOOP_HOME/etc/hadoop/slaves
```

### 2.4. Add secondary node hostname into *masters* file
```bash
echo slave1 > $HADOOP_HOME/etc/hadoop/masters
```

### 2.5. Modify `$HADOOP_HOME/etc/hadoop/core-site.xml` as following
```xml
<configuration>
<property>
    <name>fs.defaultFS</name>
    <value>hdfs://master:9000/</value>
    <description>namenode settings</description>
</property>
<property>
    <name>hadoop.tmp.dir</name>
    <value>/home/hadoop/hadoop-2.7.3/tmp/hadoop-${user.name}</value>
    <description> temp folder </description>
</property>  
<property>
    <name>hadoop.proxyuser.hadoop.hosts</name>
    <value>*</value>
</property>
<property>
    <name>hadoop.proxyuser.hadoop.groups</name>
    <value>*</value>
</property>
</configuration>
```

### 2.6. Modify `$HADOOP_HOME/etc/hadoop/hdfs-site.xml` as following
```xml
<configuration>  
    <property>  
        <name>dfs.namenode.http-address</name>  
        <value>master:50070</value>  
        <description> fetch NameNode images and edits </description>  
    </property>
    <property>  
        <name>dfs.namenode.secondary.http-address</name>  
        <value>slave1:50090</value>  
        <description> fetch SecondNameNode fsimage </description>  
    </property> 
    <property>
        <name>dfs.replication</name>
        <value>2</value>
        <description> replica count </description>
    </property>
    <property>  
        <name>dfs.namenode.name.dir</name>  
        <value>file:///home/hadoop/hadoop-2.7.3/hdfs/name</value>  
        <description> namenode </description>  
    </property>  
    <property>  
        <name>dfs.datanode.data.dir</name>
        <value>file:///home/hadoop/hadoop-2.7.3/hdfs/data</value>  
        <description> DataNode </description>  
    </property>  
    <property>  
        <name>dfs.namenode.checkpoint.dir</name>  
        <value>file:///home/hadoop/hadoop-2.7.3/hdfs/namesecondary</value>  
        <description>  check point </description>  
    </property> 
    <property>
        <name>dfs.webhdfs.enabled</name>
        <value>true</value>
    </property>
    <property>
        <name>dfs.stream-buffer-size</name>
        <value>131072</value>
        <description> buffer </description>
    </property> 
    <property>  
        <name>dfs.namenode.checkpoint.period</name>  
        <value>3600</value>  
        <description> duration </description>  
    </property> 
</configuration>
```

### 2.7. Modify `$HADOOP_HOME/etc/hadoop/mapred-site.xml` as following
```xml
<configuration>  
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
        </property>
    <property>
        <name>mapreduce.jobtracker.address</name>
        <value>hdfs://trucy:9001</value>
    </property>
    <property>
        <name>mapreduce.jobhistory.address</name>
        <value>master:10020</value>
        <description>MapReduce JobHistory Server host:port, default port is 10020.</description>
    </property>
    <property>
        <name>mapreduce.jobhistory.webapp.address</name>
        <value>master:19888</value>
        <description>MapReduce JobHistory Server Web UI host:port, default port is 19888.</description>
    </property>
</configuration>
```

### 2.8. Modify `$HADOOP_HOME/etc/hadoop/yarn-site.xml` as following
```xml
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>master</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
        <value>org.apache.hadoop.mapred.ShuffleHandler</value>
    </property>
    <property>
        <name>yarn.resourcemanager.address</name>
        <value>master:8032</value>
    </property>
    <property>
        <name>yarn.resourcemanager.scheduler.address</name>
        <value>master:8030</value>
    </property>
    <property>
        <name>yarn.resourcemanager.resource-tracker.address</name>
        <value>master:8031</value>
    </property>
    <property>
        <name>yarn.resourcemanager.admin.address</name>
        <value>master:8033</value>
    </property>
    <property>
        <name>yarn.resourcemanager.webapp.address</name>
        <value>master:8088</value>
    </property>
</configuration>
```

### 2.9. Create necessary folders
```bash
mkdir -p $HADOOP_HOME/tmp
mkdir -p $HADOOP_HOME/hdfs/name
mkdir -p $HADOOP_HOME/hdfs/data
```

### 2.10. Copy hadoop folders and environment settings to slaves
```bash
scp ~/.bashrc slave1:~/
scp ~/.bashrc slave2:~/

scp -r ~/hadoop-2.7.3 slave1:~/
scp -r ~/hadoop-2.7.3 slave2:~/
```
<br />


## 3. Launch hadoop cluster service
Format namenode for the first time launch
```bash
hdfs namenode -format
```
Launch dfs distributed file system
```bash
start-dfs.sh
```
Launch yarn distributed computing system
```bash
start-yarn.sh
```
<br />

## 4. Shutdown hadoop cluster
```bash
stop-yarn.sh
stop-dfs.sh
```
<br />

## 5. Verify the hadoop cluster is up and healthy

### 5.1. Verify by jps processus

Check *jps* on each node, and view results. <br />
jps normal results on *master* node: 
```bash
# jps
32967 NameNode
33225 Jps
32687 ResourceManager 
```
On *slave1* node:
```bash
# jps
28227 SecondaryNameNode
28496 Jps
28179 DataNode
28374 NodeManager 
```

On *slave2* node:
```bash
# jps
27680 DataNode
27904 Jps
27784 NodeManager
```

### 5.2. Verify on Web interface
`192.168.171.132:50070` to view hdfs storage status. <br />
`192.168.171.132:8088` to view yarn computing system resources and application status.<br />
