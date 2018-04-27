定位Segment

既然ConcurrentHashMap使用分段锁Segment来保护不同段的数据，那么在插入和获取元素

的时候，必须先通过散列算法定位到Segment。可以看到ConcurrentHashMap会首先使用

Wang/Jenkins hash的变种算法对元素的hashCode进行一次再散列。

private static int hash\(int h\) {

h += \(h &lt;&lt; 15\) ^ 0xffffcd7d;

h ^= \(h &gt;&gt;&gt; 10\);

h += \(h &lt;&lt; 3\);

h ^= \(h &gt;&gt;&gt; 6\);

h += \(h &lt;&lt; 2\) + \(h &lt;&lt; 14\);

return h ^ \(h &gt;&gt;&gt; 16\);

}

之所以进行再散列，目的是减少散列冲突，使元素能够均匀地分布在不同的Segment上，

从而提高容器的存取效率。假如散列的质量差到极点，那么所有的元素都在一个Segment中，

不仅存取元素缓慢，分段锁也会失去意义。笔者做了一个测试，不通过再散列而直接执行散列

计算。

System.out.println\(Integer.parseInt\("0001111", 2\) & 15\);

System.out.println\(Integer.parseInt\("0011111", 2\) & 15\);

System.out.println\(Integer.parseInt\("0111111", 2\) & 15\);

System.out.println\(Integer.parseInt\("1111111", 2\) & 15\);

计算后输出的散列值全是15，通过这个例子可以发现，如果不进行再散列，散列冲突会非

常严重，因为只要低位一样，无论高位是什么数，其散列值总是一样。我们再把上面的二进制

数据进行再散列后结果如下（为了方便阅读，不足32位的高位补了0，每隔4位用竖线分割下）。

0100｜0111｜0110｜0111｜1101｜1010｜0100｜1110

1111｜0111｜0100｜0011｜0000｜0001｜1011｜1000

0111｜0111｜0110｜1001｜0100｜0110｜0011｜1110

1000｜0011｜0000｜0000｜1100｜1000｜0001｜1010

可以发现，每一位的数据都散列开了，通过这种再散列能让数字的每一位都参加到散列

运算当中，从而减少散列冲突。ConcurrentHashMap通过以下散列算法定位segment。

final Segment&lt;K,V&gt; segmentFor\(int hash\) {

return segments\[\(hash &gt;&gt;&gt; segmentShift\) & segmentMask\];

}

默认情况下segmentShift为28，segmentMask为15，再散列后的数最大是32位二进制数据，

向右无符号移动28位，意思是让高4位参与到散列运算中，（hash&gt;&gt;&gt;segmentShift）

&segmentMask的运算结果分别是4、15、7和8，可以看到散列值没有发生冲突。

