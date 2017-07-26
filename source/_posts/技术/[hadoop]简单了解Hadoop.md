---
title: "[hadoop]简单了解Hadoop"
date: 2016-03-31 13:55 
tags:
- hadoop
- google 
- mapreduce
categories: 
- 技术
---



# What is Hadoop

> Hadoop is built to process large amounts of data from terabytes to petabytes and beyond. With this much data, it’s unlikely that it would fit on a single computer's hard drive, much less in memory. The beauty of Hadoop is that it is designed to efficiently process huge amounts of data by connecting many commodity computers together to work in parallel. Using the MapReduce model, Hadoop can take a query over a dataset, divide it, and run it in parallel over multiple nodes. Distributing the computation solves the problem of having data that’s too large to fit onto a single machine.

# Hapood 跟过去的分布式工具有什么不同?

1. Hapood可以以处理最简单的流式数据，不用特定的格式、结构.
2. Hapood有非常好用的API来方便分布式编程。 
3. 可以方便的管理，也有节点宕机自动切换功能。 
4. 更加灵活，因为不受数据结构限制。


# HDFS
 Hapood Ditribute File System

 