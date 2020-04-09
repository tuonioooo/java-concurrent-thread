# Java内存模型

## 目录

* [Java 内存模型的基础](java-nei-cun-mo-xing-de-ji-chu/) 
* [重排序](zhong-pai-xu/) 
* [顺序一致性](shun-xu-yi-zhi-xing/) 
* [volatile内存语义](volatilenei-cun-yu-yi/) 
* [锁内存定义](suo-nei-cun-ding-yi/) 
* [final域内存语义](finalyu-nei-cun-yu-yi/)

Java线程之间的通信对程序员完全透明，内存可见性问题很容易困扰Java程序员，本章将  
揭开Java内存模型神秘的面纱。

本章大致分4部分：

Java内存模型的基础，主要介绍内存模型相  
关的基本概念；

Java内存模型中的顺序一致性，主要介绍重排序与顺序一致性内存模型；

同步  
原语，主要介绍3个同步原语（synchronized、volatile和final）的内存语义及重排序规则在处理器  
中的实现；

Java内存模型的设计，主要介绍Java内存模型的设计原理，及其与处理器内存模型  
和顺序一致性内存模型的关系。

> **注意阅读本章读者用了解的心态去看**

