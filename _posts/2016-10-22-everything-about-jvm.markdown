---
layout: post
title:  "Everything about JVM"
date:   2016-10-22 08:43:59
author: Unskilled Coder
categories: jvm
---

This post generally is to explain what JVM is & how it works. I'm here to record my understandings of this mistery black box and share them with you.

### 1. Introduction
As we all know JVM is the abbreviation of Java Virtual Machine, which is the cornerstone of the Java Platform. It mainly owns the following responsibility:
* Manage resources between living OS and its inside running code, like all virtual machine do.
* Optimize the space & time efficiency of the running code.
* Manage and automate garbage collection of the objects in memory.
* Manage code loading & initialization. 

### 2. Focused points
#### 2.1. JVM memory model & garbage collection
#### 2.1.1. JVM memory model

Let's directly get into our topic, the JVM memory model.
The JVM memory can be simply splitted into 3 parts
1. Heap Memory
2. Non-heap Memory
3. Other

Heap memory stores all java objects. Non-heap memory, on another hand, keeps all loaded classes and meta-data. For last "other" memory, stores JVM itself, codes and runtimes.

#### 2.1.2. Heap memory & Garbage collection

The garbage collection is always a popular issue in the c++ based languages, which may cause stop-the-world pause, so that have a big impact on the system response speed. 

Heap memory can be divided into 3 parts, which are: Young generation, Old generation & Permanent generation.

**Young Generation** is responsible for containing newly created objects. Once gc is called, JVM will move all survived objects in **Eden** section to blank-half **Survivor** section (either **from** or **to**). And another half **Survivor** section moves contained survivor objects to this blank-half section as well. If this **Survivor** section is full, the rest objects will be moved to the **Old Generation**. Except this blank-half section, others will be cleaned up. This gc algorithm also called Copying algorithm, which is designed for better performance, but may waste half of memory.

**Old Generation** is responsible for keeping long last objects. Once gc happens, the Mark-Sweep-Compact gc algorithm will be applied onto this section, which may have worse performance, but take advantage of memory usage.

So based on the above description, we should not create and release big objects with high frequency, it's better to keep objects small, or even create less objects.

#### 2.1.3. Non-Heap Memory

**Permanent Generation** is used for store class related meta-data, which will not participate in gc events.

In addition, the **Permanent Generation** is replaced by **Meta-data space** after Java 8, however they are very similar, except **Meta-data space** can be dynamically re-size in runtime.

<br/>
#### More points to add...
Recommended book for JVM : Java Performance (2012) - Charlie Hunt