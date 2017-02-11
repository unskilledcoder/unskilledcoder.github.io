---
layout: post
title:  "Spark 2.1.0 setup on YARN environment, along with Zeppelin notebook"
date:   2017-02-10 09:00:00
author: Hao WU
categories: hadoop spark
---

## Summary

### [1. Introduction](#1)

### [2. Architecture](#2)

### [3. Spark Setup](#3)

### [4. Zeppelin Setup](#4)

<br />

## <a name="1">1. Introduction</a>

This posts will give the detail about how to setup Spark environment onto YARN computing cluster, and also along with apache [Zeppelin](http://zeppelin.apache.org) notebook.

<br />

## <a name="2">2. Architecture</a>

The architecture is specified in my previous [post](/hadoop/2016/12/10/hadoop-cluster-installation-basic-version.html), please refer to it for YARN to be setup.

<br />

## <a name="3">3. Spark Setup</a>

P.S. We use *hadoop* as our login user.

### 3.1. Download and untar Spark bin package

```bash
wget http://d3kbcqa49mib13.cloudfront.net/spark-2.1.0-bin-without-hadoop.tgz
tar -xvf spark-2.1.0-bin-without-hadoop.tgz
rm spark-2.1.0-bin-without-hadoop.tgz
mv spark-2.1.0-bin-without-hadoop ~/spark-2.1.0
```

### 3.2. Add environment variables, append the following to `~/.bashrc`

```bash
export SPARK_MASTER_HOST=master
export SPARK_HOME=/home/hadoop/spark-2.1.0
export SPARK_LOCAL_DIRS=$SPARK_HOME/storage
export SPARK_DRIVER_MEMORY=1G
export PATH=$PATH:$SPARK_HOME/bin
export SPARK_DIST_CLASSPATH=$(hadoop classpath)
```

Run `source ~/.bashrc` to make environment variables effect

### 3.3. Specify slave node info to conf/slaves

```bash
echo slave1 > $SPARK_HOME/conf/slaves
echo slave2 >> $SPARK_HOME/conf/slaves
```

### 3.4. Copy to slave nodes

```bash
scp ~/.bashrc slave1:~/
scp ~/.bashrc slave2:~/

scp -r ~/spark-2.1.0 slave1:~/
scp -r ~/spark-2.1.0 slave2:~/
```

### 3.5. Check if Spark works well in standalone cluster mode

```bash
~/spark-2.1.0/sbin/start-all.sh
```

check web ui master:8080 is up and displays ok, then turn the cluster down because we will use YARN as our mode. `~/spark-2.1.0/sbin/stop-all.sh`

### 3.6. The Spark commands below now can be now running on YARN

```bash
spark-submit
spark-shell
pyspark
```

<br />

## <a name="4"> 4. Zeppelin Setup</a>

P.S. We'll continue use *hadoop* as our login user.

### 4.1. Download and untar Zeppelin bin package

```bash
wget http://apache.mirrors.ionfish.org/zeppelin/zeppelin-0.7.0/zeppelin-0.7.0-bin-all.tgz
tar -xvf zeppelin-0.7.0-bin-all.tgz
rm zeppelin-0.7.0-bin-all.tgz
mv zeppelin-0.7.0-bin-all ~/zeppelin-0.7.0
```

### 4.2. Add environment variables, append the following to `~/.bashrc`

```bash
export ZEPPELIN_HOME=/home/hadoop/zeppelin-0.7.0
export PATH=$PATH:$ZEPPELIN_HOME/bin
```

Run `source ~/.bashrc` to make environment variables effect

### 4.3. Check if Zeppelin works well

Launch the Zeppelin daemon use `zeppelin-daemon.sh start`

check web ui master:8080 is up and works ok.

use commands below to operate the zeppelin notebook.

`zeppelin-daemon.sh start/restart/stop`

<br />
