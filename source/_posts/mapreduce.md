---
title: mapreduce
date: 2022-07-17 10:10:23
categories: 
  - [database]
  - [mapreduce]
tags: [paper_read,mapreduce]
category_bar: true
---

懒得写了

[论文链接](https://static.googleusercontent.com/media/research.google.com/zh-CN//archive/mapreduce-osdi04.pdf)

[中翻链接](https://www.cnblogs.com/fuzhe1989/p/3413457.html)

就说一下mapreduce的思路吧

![MapReduce Overview](mapreduce/image-20221015150844432.png)

首先MapReduce大体来看就是一个分治思想，将任务拆分成小任务交给Worker，Worker经过处理后得到各自的结果，然后通过再经过Reduce将结果输出。

详细过程如上图。用户程序将拆分好的文件作为inputFiles，每个inputFile可以看成一个MapTask，由Worker执行。但是这个分配任务的过程是由Mapreduce集群中的Master节点来控制的，可以将每个Worker看成一个机器节点，由Worker向Master请求任务，Master向空闲的Worker节点发送Map任务或者Reduce任务。Worker读取对应的输入文件的内容，将其根据自定义的规则解析出Key/Value值，然后将这些值作为中间文件存储在内存中。缓存的数据对会被周期性的由划分函数分成Reduce节点的数量，并写入本地磁盘中。这些缓存对在本地磁盘中的位置会被传回给主节点，主节点负责将这些位置再传给reduce工作节点。

当一个reduce工作节点得到了主节点的这些位置通知后，它使用RPC调用去读map工作节点的本地磁盘中的缓存数据。当reduce工作节点读取完了所有的中间数据，它会将这些数据按中间key排序，这样相同key的数据就被排列在一起了。同一个reduce任务经常会分到有着不同key的数据，因此这个排序很有必要。如果中间数据数量过多，不能全部载入内存，则会使用外部排序。reduce工作节点遍历排序好的中间数据，并将遇到的每个中间key和与它关联的一组中间value传递给用户的reduce函数。reduce函数的输出会写到由reduce划分过程划分出来的最终输出文件的末尾。

当所有的map和reduce任务都完成后，主节点唤醒用户程序。此时，用户程序中的MapReduce调用返回到用户代码中。成功完成后，MapReduce执行的输出都在R个输出文件中（每个reduce任务产生一个，文件名由用户指定）。通常用户不需要合并这R个输出文件——他们经常会把这些文件当作另一个MapReduce调用的输入，或是用于另一个可以处理分成多个文件输入的分布式应用。

看看论文中的例子就大概懂了。
