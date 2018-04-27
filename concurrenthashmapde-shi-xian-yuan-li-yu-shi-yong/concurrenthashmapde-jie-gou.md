# ConcurrentHashMap的结构

通过ConcurrentHashMap的类图来分析ConcurrentHashMap的结构，如图6-1所示。ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。Segment是一种可重入锁（ReentrantLock），在ConcurrentHashMap里扮演锁的角色；HashEntry则用于存储键值对数据。一个ConcurrentHashMap里包含一个Segment数组。Segment的结构和HashMap类似，是一种数组和链表结构。一个Segment里包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得与它对应的Segment锁，如图6-2所示。

图6-1的类图

![](/assets/import-6-1.png)



图6-2 ConcurrentHashMap的结构图![](/assets/import-6-2.png)

