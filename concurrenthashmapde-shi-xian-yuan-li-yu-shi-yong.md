# ConcurrentHashMap的实现原理与使用

ConcurrentHashMap是线程安全且高效的HashMap。本节让我们一起研究一下该容器是如何在保证线程安全的同时又能保证高效的操作。

## 目录

* [为什么要使用ConcurrentHashMap](/concurrenthashmapde-shi-xian-yuan-li-yu-shi-yong/wei-shi-yao-yao-shi-yong-concurrenthashmap.md) 
* [ConcurrentHashMap的结构](/concurrenthashmapde-shi-xian-yuan-li-yu-shi-yong/concurrenthashmapde-jie-gou.md) 
* [ConcurrentHashMap的初始化](/concurrenthashmapde-shi-xian-yuan-li-yu-shi-yong/concurrenthashmapde-chu-shi-hua.md) 
* [定位Segment](/concurrenthashmapde-shi-xian-yuan-li-yu-shi-yong/ding-wei-segment.md) 
* [ConcurrentHashMap的操作](/concurrenthashmapde-shi-xian-yuan-li-yu-shi-yong/concurrenthashmapde-cao-zuo.md) 
* [ConcurrentHashMap讲解\(一\)](/concurrenthashmapde-shi-xian-yuan-li-yu-shi-yong/concurrenthashmapjiang-89e328-4e0029.md)



