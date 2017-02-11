---
layout: post
title:  "Zookeeper 3.4.9 Cluster Setup for Hadoop"
date:   2016-12-10 09:00:00
author: Hao WU
categories: hadoop zookeeper
---

## Summary

### [1. Introduction](#1)

### [2. Architecture](#2)

### [3. Zookeeper Setup](#3)

### [4. Launch and Shutdown Zookeeper Cluster Service](#4)

### [5. Verify the Zookeeper cluster is up and healthy](#5)

<br />

## <a name="1">1. Introduction</a>

This post is to basically guide you to setup a Zookeeper cluster based on the Hadoop cluster I previously built, please refer to my another post -- [Hadoop Cluster 2.6.5 Installation on CentOS 7 in basic version](/hadoop/2016/12/10/hadoop-cluster-installation-basic-version.html).
<br />

## <a name="2">2. Architecture</a>

| IP Address      | Hostname    | Role                                                     |
|:----------------|:------------|:---------------------------------------------------------|
| 192.168.171.132 | master      | NameNode, ResourceManager, QuorumPeerMain                |
| 192.168.171.133 | slave1      | SecondaryNameNode, DataNode, NodeManager, QuorumPeerMain |
| 192.168.171.134 | slave2      | DataNode, NodeManager, QuorumPeerMain                    |

<br />

## <a name="3">3. Zookeeper Setup</a>

P.S. Please login as *hadoop* to build this cluster.

### 3.1. Download and untar

```bash
wget http://apache.mirrors.pair.com/zookeeper/stable/zookeeper-3.4.9.tar.gz
tar -zxvf zookeeper-3.4.9.tar.gz
rm zookeeper-3.4.9.tar.gz
```

### 3.2. Configuration files

* Append following content in `~/.bashrc`

```bash
export ZOOKEEPER_HOME=/home/hadoop/zookeeper-3.4.9
export PATH=$PATH:$ZOOKEEPER_HOME/bin
```

* Make environment variable valid

```bash
source ~/.bashrc
```

* Create `zoo.cfg` file

```bash
cp $ZOOKEEPER_HOME/conf/zoo_sample.cfg $ZOOKEEPER_HOME/conf/zoo.cfg
```

* Append the following to `zoo.cfg`

```bash
dataLogDir=/home/hadoop/zookeeper-3.4.9/logs

server.1=master:2888:3888
server.2=slave1:2888:3888
server.3=slave2:2888:3888
```

### 3.3. Copy the zookeeper folder and configuration to all nodes

```bash
cd ~
scp -r .bashrc slave1:~/
scp -r .bashrc slave2:~/

scp -r zookeeper-3.4.9 slave1:~/
scp -r zookeeper-3.4.9 slave2:~/
```

### 3.4. Specify the server id according to the `zoo.cfg` configuration, around all nodes

```bash
ssh master echo 1 > $ZOOKEEPER_HOME/data/myid
ssh slave1 echo 2 > $ZOOKEEPER_HOME/data/myid
ssh slave2 echo 3 > $ZOOKEEPER_HOME/data/myid
```

P.S. **1** for master, 2 for slave1, 3 for slave2

### <a name="4">4. Launch and Shutdown Zookeeper Cluster Service</a>

```bash
zkServer.sh start
```

```bash
zkServer.sh stop
```

### <a name="5">5. Verify the Zookeeper cluster is up and healthy</a>

* use `jps` to check through all nodes

```bash
# jps
29765 QuorumPeerMain
```

* `zkCli.sh` to verify all nodes are well being synchronized.

* Done.
