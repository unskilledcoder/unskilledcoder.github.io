---
layout: post
title:  "HBase 1.2.4 Cluster setup with Zookeeper"
date:   2016-12-11 09:00:00
author: Hao WU
categories: hadoop hbase
---

## Summary

### [1. Introduction](#1)

### [2. Architecture](#2)

### [3. HBase Setup](#3)

### [4. Launch and Shutdown HBase Cluster Service](#4)

### [5. Verify the HBase cluster is up and healthy](#5)

<br />

## <a name="1">1. Introduction</a>

In this post, I'm going to go through the HBase Cluster setup process (version 1.2.4) onto the environment we just built in these posts:

- [Hadoop Cluster 2.6.5 Installation on CentOS 7 in basic version](/hadoop/2016/12/10/hadoop-cluster-installation-basic-version.html)
- [Zookeeper 3.4.9 Cluster Setup for Hadoop](/hadoop/zookeeper/2016/12/10/zookeeper-cluster-setup-for-hadoop.html)

<br />

## <a name="2">2. Architecture</a>

| IP Address      | Hostname    | Role                                                                    |
|:----------------|:------------|:------------------------------------------------------------------------|
| 192.168.171.132 | master      | NameNode, ResourceManager, QuorumPeerMain, HMaster                      |
| 192.168.171.133 | slave1      | SecondaryNameNode, DataNode, NodeManager, QuorumPeerMain, HRegionServer |
| 192.168.171.134 | slave2      | DataNode, NodeManager, QuorumPeerMain, HRegionServer                    |

<br />

## <a name="3">3. HBase Setup</a>

P.S. Also, we use *hadoop* as our login user.

### 3.1. Download and untar HBase bin package

```bash
wget http://apache.claz.org/hbase/stable/hbase-1.2.4-bin.tar.gz
tar -zxvf hbase-1.2.4-bin.tar.gz
rm hbase-1.2.4-bin.tar.gz
```

### 3.2. Add environment variables, append the following to `~/.bashrc`

```bash
export HBASE_HOME=/home/hadoop/hbase-1.2.4
export HBASE_MANAGES_ZK=false
export PATH=$PATH:$HBASE_HOME/bin
```

### 3.3. Make the variables effect

```bash
source ~/.bashrc
```

### 3.4. Modify the HBase config file `$HBASE_HOME/conf/hbase-site.xml`

```xml
<configuration>
    <!-- the port and hostname should be identity to the fs.default.name in $HADOOP_HOME/conf/core-site.xml -->
    <property>
        <name>hbase.rootdir</name>
        <value>hdfs://master:9000/hbase</value>
    </property>
    <property>
        <name>hbase.cluster.distributed</name>
        <value>true</value>
    </property>
    <property>
        <name>hbase.master</name>
        <value>master:60000</value>
    </property>
    <property>
        <name>hbase.master.port</name>
        <value>60000</value>
        <description>The port master should bind to.</description>
    </property>
    <!-- zookeeper cluster we setup in previous post -->
    <property>
        <name>hbase.zookeeper.quorum</name>
        <value>master,slave1,slave2</value>
    </property>
    <!-- 2 since we have 2 slaves for data -->
    <property>
        <name>dfs.replication</name>
        <value>2</value>
    </property>
</configuration>
```

### 3.5. Write correct region server hostnames into `$HBASE_HOME/conf/regionservers`

```bash
echo slave1 > $HBASE_HOME/conf/regionservers
echo slave2 >> $HBASE_HOME/conf/regionservers
```

### 3.6. Copy the environment variables and HBase config to other nodes

```bash
scp -r ~/.bashrc slave1:~/
scp -r ~/.bashrc slave2:~/

scp -r ~/hbase-1.2.4 slave1:~/
scp -r ~/hbase-1.2.4 slave2:~/
```

<br />

## <a name="4">4. Launch and Shutdown HBase Cluster Service</a> 

First of all, we should start Hadoop cluster and make sure it's up and healthy.

After that, start the hbase cluster in *master* node:

```bash
start-hbase.sh
```

And the way to shutdown the whole cluster:

```bash
stop-hbase.sh
```

## <a name="5">5. Verify the HBase cluster is up and healthy</a>

* First we should check `jps` in all nodes, to make sure all nodes take the right responsibility. 

Normally, the `jps` returns as it's described in [Architecture](#2), if not, please clear all log files, and restart the cluster for try.

* Enter hbase shell to do some shell operations for test.

```bash
hbase shell
```

* Open `http://192.168.171.132:16010/` to view the cluster status on Web Interface.

* Done.
