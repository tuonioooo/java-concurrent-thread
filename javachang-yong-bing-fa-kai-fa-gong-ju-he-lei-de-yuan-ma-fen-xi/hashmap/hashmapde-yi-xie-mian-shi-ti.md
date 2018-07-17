1**、HashMap的工作原理**

  


HashMap，都知道哪里要用HashMap，知道Hashtable和HashMap之间的区别，那么为何这道面试题如此特殊呢？是因为这道题考察的深度很深。

这题经常出现在高级或中高级面试中。投资银行更喜欢问这个问题，甚至会要求你实现HashMap来考察你的编程能力。

ConcurrentHashMap和其它同步集合的引入让这道题变得更加复杂。让我们开始探索的旅程吧！

  


先来些简单的问题

  


“你用过HashMap吗？”

  


“什么是HashMap？你为什么用到它？”

  


几乎每个人都会回答“是的”，然后回答HashMap的一些特性，譬如HashMap可以接受null键值和值，而Hashtable则不能；HashMap是非

  


synchronized;HashMap很快；以及HashMap储存的是键值对等等。这显示出你已经用过HashMap，而且对它相当的熟悉。但是面试官来个急转直下，

  


从此刻开始问出一些刁钻的问题，关于HashMap的更多基础的细节。面试官可能会问出下面的问题：

  


“你知道HashMap的工作原理吗？”

  


“你知道HashMap的get\(\)方法的工作原理吗？”

  


你也许会回答“我没有详查标准的Java API，你可以看看Java源代码或者Open JDK。”“我可以用Google找到答案。”

  


但一些面试者可能可以给出答案，“HashMap是基于hashing的原理，我们使用put\(key, value\)存储对象到HashMap中，使用get\(key\)从HashMap中

  


获取对象。当我们给put\(\)方法传递键和值时，我们先对键调用hashCode\(\)方法，返回的hashCode用于找到bucket位置来储存Entry对象。”这里关

  


键点在于指出，HashMap是在bucket中储存键对象和值对象，作为Map.Entry。这一点有助于理解获取对象的逻辑。如果你没有意识到这一点，或者

  


错误的认为仅仅只在bucket中存储值的话，你将不会回答如何从HashMap中获取对象的逻辑。这个答案相当的正确，也显示出面试者确实知道

  


hashing以及HashMap的工作原理。但是这仅仅是故事的开始，当面试官加入一些Java程序员每天要碰到的实际场景的时候，错误的答案频现。下个

  


问题可能是关于HashMap中的碰撞探测\(collision detection\)以及碰撞的解决方法：

  


“当两个对象的hashcode相同会发生什么？”

  


从这里开始，真正的困惑开始了，一些面试者会回答因为hashcode相同，所以两个对象是相等的，HashMap将会抛出异常，或者不会存储它们。然

  


后面试官可能会提醒他们有equals\(\)和hashCode\(\)两个方法，并告诉他们两个对象就算hashcode相同，但是它们可能并不相等。一些面试者可能就

  


此放弃，而另外一些还能继续挺进，他们回答“因为hashcode相同，所以它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用LinkedList

  


存储对象，这个Entry\(包含有键值对的Map.Entry对象\)会存储在LinkedList中。”这个答案非常的合理，虽然有很多种处理碰撞的方法，这种方法

  


是最简单的，也正是HashMap的处理方法。但故事还没有完结，面试官会继续问：

  


“如果两个键的hashcode相同，你如何获取值对象？”

  


面试者会回答：当我们调用get\(\)方法，HashMap会使用键对象的hashcode找到bucket位置，然后获取值对象。面试官提醒他如果有两个值对象储存

  


在同一个bucket，他给出答案:将会遍历LinkedList直到找到值对象。面试官会问因为你并没有值对象去比较，你是如何确定确定找到值对象的？

  


除非面试者直到HashMap在LinkedList中存储的是键值对，否则他们不可能回答出这一题。

  


其中一些记得这个重要知识点的面试者会说，找到bucket位置之后，会调用keys.equals\(\)方法去找到LinkedList中正确的节点，最终找到要找的值对象。

完美的答案！

