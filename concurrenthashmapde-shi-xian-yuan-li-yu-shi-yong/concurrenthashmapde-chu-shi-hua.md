# ConcurrentHashMap的初始化

ConcurrentHashMap初始化方法是通过initialCapacity、loadFactor和concurrencyLevel等几个参数来初始化segment数组、段偏移量segmentShift、段掩码segmentMask和每个segment里的HashEntry数组来实现的。

**1.初始化segments数组**

让我们来看一下初始化segments数组的源代码。

```text
if(concurrencyLevel>MAX_SEGMENTS)

            concurrencyLevel=MAX_SEGMENTS;

            int sshift=0;

            int ssize=1;

            while(ssize<concurrencyLevel){

        ++sshift;

        ssize<<=1;

        }

        segmentShift=32-sshift;

        segmentMask=ssize-1;

        this.segments=Segment.newArray(ssize);
```

由上面的代码可知，segments数组的长度ssize是通过concurrencyLevel计算得出的。为了能通过按位与的散列算法来定位segments数组的索引，必须保证segments数组的长度是2的N次方（power-of-two size），所以必须计算出一个大于或等于concurrencyLevel的最小的2的N次方值来作为segments数组的长度。假如concurrencyLevel等于14、15或16，ssize都会等于16，即容器里

锁的个数也是16。

**注意** concurrencyLevel的最大值是65535，这意味着segments数组的长度最大为65536，对应的二进制是16位。

**2.初始化segmentShift和segmentMask**

这两个全局变量需要在定位segment时的散列算法里使用，sshift等于ssize从1向左移位的次数，在默认情况下concurrencyLevel等于16，1需要向左移位移动4次，所以sshift等于4。

segmentShift用于定位参与散列运算的位数，segmentShift等于32减sshift，所以等于28，这里之所以用32是因为ConcurrentHashMap里的hash\(\)方法输出的最大数是32位的，后面的测试中我们可以看到这点。segmentMask是散列运算的掩码，等于ssize减1，即15，掩码的二进制各个位的值都是1。因为ssize的最大长度是65536，所以segmentShift最大值是16，segmentMask最大值是65535，对应的二进制是16位，每个位都是1。

**3.初始化每个segment**

输入参数initialCapacity是ConcurrentHashMap的初始化容量，loadfactor是每个segment的负载因子，在构造方法里需要通过这两个参数来初始化数组中的每个segment。

```text
if (initialCapacity > MAXIMUM_CAPACITY)

        initialCapacity = MAXIMUM_CAPACITY;

        int c = initialCapacity / ssize;

        if (c * ssize < initialCapacity)

        ++c;

        int cap = 1;

        while (cap < c)

        cap <<= 1;

        for (int i = 0; i < this.segments.length; ++i)

        this.segments[i] = new Segment<K,V>(cap, loadFactor);
```

上面代码中的变量cap就是segment里HashEntry数组的长度，它等于initialCapacity除以ssize的倍数c，如果c大于1，就会取大于等于c的2的N次方值，所以cap不是1，就是2的N次方。

segment的容量threshold＝（int）cap\*loadFactor，默认情况下initialCapacity等于16，loadfactor等于0.75，通过运算cap等于1，threshold等于零。

