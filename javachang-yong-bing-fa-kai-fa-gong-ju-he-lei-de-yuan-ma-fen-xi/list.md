# List

[原文链接](http://ifeve.com/java-collection-list/)

`java.util.List`接口是`java.util.Collection`接口的一个子接口。它表示对象的一个有序列表，意味你可以特定的顺序访问元素，也可以通过索引访问。也可以向一个`List`中多次添加重复的元素。

**Java List 视频教程**

如果你更喜欢看视频而不是文本，下面是一个版本的Java List视频教程。  
[https://youtu.be/d3QbptJRln4](https://youtu.be/d3QbptJRln4)

**List实现**

作为一个Collection的子类型，Collection接口的所有方法在List接口里也适用。

因为`List`是一个接口，为了使用它，你必须实例化一个具体的实现，你可以在下列List的实现中选择：

1. java.util.ArrayList
2. java.util.LinkedList
3. java.util.Vector
4. java.util.Stack

在`java.util.concurrent`包中，同样也有`List`的实现，但是在本教程中，我将不考虑并发程序。

