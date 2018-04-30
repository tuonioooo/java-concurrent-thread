# ConcurrentLinkedQueue的结构

通过ConcurrentLinkedQueue的类图来分析一下它的结构，如图6-3所示。

图6-3　ConcurrentLinkedQueue的类图

![](/assets/import-6-3.png)

ConcurrentLinkedQueue由head节点和tail节点组成，每个节点（Node）由节点元素（item）和

指向下一个节点（next）的引用组成，节点与节点之间就是通过这个next关联起来，从而组成一

张链表结构的队列。默认情况下head节点存储的元素为空，tail节点等于head节点。

`private transient volatile Node<E> tail = head;`

