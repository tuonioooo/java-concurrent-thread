# HashMap讲解一 {#hashmap详讲}

## hashing\(散列法或哈希法\)的概念 {#hashing散列法或哈希法的概念}

可以自行百度

## 什么是HashMap以及HashMap的构成 {#什么是hashmap以及hashmap的构成}

HashMap是基于哈希表的Map接口的非同步实现。此实现提供所有可选的映射操作，并允许使用null值和null键。HashMap储存的是键值对，HashMap很快。此类不保证映射的顺序，特别是它不保证该顺序恒久不变。

HashMap实际上是一个“链表散列”的数据结构，即数组和链表的结合体。

数组：存储区间连续，占用内存严重，寻址容易，插入删除困难；  
链表：存储区间离散，占用内存比较宽松，寻址困难，插入删除容易；  
Hashmap综合应用了这两种数据结构，实现了寻址容易，插入删除也容易。

hashMap的结构示意图如下：  
![](https://img-blog.csdn.net/20170208131626900?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbHl0XzdjczFkbjk=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast "这里写图片描述")

## HashMap的基本存储原理以及存储内容的组成 {#hashmap的基本存储原理以及存储内容的组成}

基本原理：先声明一个下标范围比较大的数组来存储元素。另外设计一个哈希函数（也叫做散列函数）来获得每一个元素的Key（关键字）的函数值（即数组下标，hash值）相对应，数组存储的元素是一个Entry类，这个类有三个数据域，key、value（键值对），next\(指向下一个Entry\)。  
例如， 第一个键值对A进来。通过计算其key的hash得到的index=0。记做:Entry\[0\] = A。  
第二个键值对B，通过计算其index也等于0， HashMap会将B.next =A,Entry\[0\] =B,  
第三个键值对 C,index也等于0,那么C.next = B,Entry\[0\] = C；这样我们发现index=0的地方事实上存取了A,B,C三个键值对,它们通过next这个属性链接在一起。我们可以将这个地方称为桶。 对于不同的元素，可能计算出了相同的函数值，这样就产生了“冲突”，这就需要解决冲突，“直接定址”与“解决冲突”是哈希表的两大特点。

## HashMap的工作原理以及存取方法过程 {#hashmap的工作原理以及存取方法过程}

HashMap的工作原理 ：HashMap是基于散列法（又称哈希法hashing）的原理，使用put\(key, value\)存储对象到HashMap中，使用get\(key\)从HashMap中获取对象。当我们给put\(\)方法传递键和值时，我们先对键调用hashCode\(\)方法，返回的hashCode用于找到bucket（桶）位置来储存Entry对象。”HashMap是在bucket中储存键对象和值对象，作为Map.Entry。并不是仅仅只在bucket中存储值。

HashMap具体的存取过程如下：  
put键值对的方法的过程是：  
1、获取key ；  
2、通过hash函数得到hash值；  
int hash=key.hashCode\(\); //获取key的hashCode，这个值是一个固定的int值

3、得到桶号\(一般都为hash值对桶数求模\) ，也即数组下标int index=hash%Entry\[\].length。//获取数组下标：key的hash值对Entry数组长度进行取余

4、 存放key和value在桶内。  
table\[index\]=Entry对象；

get值方法的过程是:  
1、获取key  
2、通过hash函数得到hash值  
int hash=key.hashCode\(\);

3、得到桶号\(一般都为hash值对桶数求模\)  
int index =hash%Entry\[\].length;

4、比较桶的内部元素是否与key相等，若都不相等，则没有找到。

5、取出相等的记录的value。

HashMap中直接地址用hash函数生成；解决冲突，用比较函数解决。如果每个桶内部只有一个元素，那么查找的时候只有一次比较。当许多桶内没有值时，许多查询就会更快了\(指查不到的时候\)。

## HashMap中的碰撞探测\(collision detection\)以及碰撞的解决方法 {#hashmap中的碰撞探测collision-detection以及碰撞的解决方法}

当两个对象的hashcode相同时，它们的bucket位置相同，‘碰撞’会发生。因为HashMap使用LinkedList存储对象，这个Entry\(包含有键值对的Map.Entry对象\)会存储在LinkedList中。这两个对象就算hashcode相同，但是它们可能并不相等。 那如何获取这两个对象的值呢？当我们调用get\(\)方法，HashMap会使用键对象的hashcode找到bucket位置，遍历LinkedList直到找到值对象。找到bucket位置之后，会调用keys.equals\(\)方法去找到LinkedList中正确的节点，最终找到要找的值对象使用不可变的、声明作final的对象，并且采用合适的equals\(\)和hashCode\(\)方法的话，将会减少碰撞的发生，提高效率。不可变性使得能够缓存不同键的hashcode，这将提高整个获取对象的速度，使用String，Interger这样的wrapper类作为键是非常好的选择。

## 如何重新调整HashMap的大小 {#如何重新调整hashmap的大小}

“如果HashMap的大小超过了负载因子\(load factor\)定义的容量，怎么办？”  
默认的负载因子大小为0.75，也就是说，当一个map填满了75%的bucket时候，和其它集合类\(如ArrayList等\)一样，将会创建原来HashMap大小的两倍的bucket数组，来重新调整map的大小，并将原来的对象放入新的bucket数组中。这个过程叫作rehashing，因为它调用hash方法找到新的bucket位置。

## 不可变对象的好处 {#不可变对象的好处}

上面说到使用包装类时刻作为键的原因是 String, Interger这样的wrapper类作为HashMap的键是很合适的，而且String最为常用。因为String是不可变的，也是final的，而且已经重写了equals\(\)和hashCode\(\)方法了。其他的wrapper类也有这个特点。不可变性是必要的，因为为了要计算hashCode\(\)，就要防止键值改变，如果键值在放入时和获取时返回不同的hashcode的话，那么就不能从HashMap中找到你想要的对象。不可变性还有其他的优点如线程安全。如果你可以仅仅通过将某个field声明成final就能保证hashCode是不变的，那么请这么做吧。因为获取对象的时候要用到equals\(\)和hashCode\(\)方法，那么键对象正确的重写这两个方法是非常重要的。如果两个不相等的对象返回不同的hashcode的话，那么碰撞的几率就会小些，这样就能提高HashMap的性能。

## HashMap多线程的条件竞争 {#hashmap多线程的条件竞争}

重新调整HashMap大小存在什么问题吗？”在多线程的情况下，可能产生条件竞争\(race condition\)。因为如果两个线程都发现HashMap需要重新调整大小了，它们会同时试着调整大小。在调整大小的过程中，存储在LinkedList中的元素的次序会反过来，因为移动到新的bucket位置的时候，HashMap并不会将元素放在LinkedList的尾部，而是放在头部，这是为了避免尾部遍历\(tail traversing\)。如果条件竞争发生了，那么就死循环了。\(在多线程的情况下，为什么还要使用HashMap呢？不懂\)

我们也可以使用自定义的对象作为键，只要它遵守了equals\(\)和hashCode\(\)方法的定义规则，并且当对象插入到Map中之后将不会再改变了。如果这个自定义对象时不可变的，那么它已经满足了作为键的条件，因为当它创建之后就已经不能改变了。

我们可以使用CocurrentHashMap来代替HashTable吗？这是另外一个很热门的面试题，因为ConcurrentHashMap越来越多人用了。我们知道HashTable是synchronized的，但是ConcurrentHashMap同步性能更好，因为它仅仅根据同步级别对map的一部分进行上锁。ConcurrentHashMap当然可以代替HashTable，但是HashTable提供更强的线程安全性。

