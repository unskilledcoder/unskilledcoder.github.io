---
layout: post
title:  "Difference of Java maps (HashMap, HashTable, ConcurrentHashMap)"
date:   2016-10-26 09:00:00
author: Unskilled Coder
categories: Java
---

Based on different senarios, we are often struggling choosing among different map implementations, like HashMap, HashTable & ConcurrentHashMap. Unlike other articles, I will not show up detail code here, but explain the principle and conceptions. And I highly recommend here you should at least look into the source code yourself for concret notion and deeper impression.

#### HashTable

Let me introduce **HashTable** first, which is the earliest hashmap implementation in Java library above all 3. To make this table fit all senarios, the syncronized keyword is added onto most of its methods to provide thread-safe feature. And it does not accept the null key and value.

So in javadoc there is a comment for this hashmap:

If a thread-safe implementation is not needed, it is recommended to use HashMap in place of Hashtable. If a thread-safe highly-concurrent implementation is desired, then it is recommended to use java.util.concurrent.ConcurrentHashMap in place of Hashtable.

In another word, we should not use this class any longer.

#### HashMap
The HashMap class is nothing but the new unsynchronized version of Hashtable (while code between them is not relevant), and it permits nulls.

#### ConcurrentHashMap
So it's straight forward that, this class is the thread-safe version of HashTable, but in a more reasonable way of implementation. It does not permit nulls neither just like HashTable.

Its thread-safe mechanism is simply as below: 
1. Volatile level read
2. Lock level write

** **volatile** keyword makes sure that write always happens before read operation.
