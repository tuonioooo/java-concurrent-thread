final域为引用类型

上面我们看到的final域是基础数据类型，如果final域是引用类型，将会有什么效果？请看

下列示例代码。

public class FinalReferenceExample {

final int\[\] intArray; // final是引用类型

static FinalReferenceExample obj;

public FinalReferenceExample \(\) { // 构造函数

intArray = new int\[1\]; // 1

intArray\[0\] = 1; // 2

}

public static void writerOne \(\) { // 写线程A执行

obj = new FinalReferenceExample \(\); // 3

}

public static void writerTwo \(\) { // 写线程B执行

obj.intArray\[0\] = 2; // 4

}

public static void reader \(\) { // 读线程C执行

if \(obj != null\) { // 5

int temp1 = obj.intArray\[0\]; // 6

}

}

}

本例final域为一个引用类型，它引用一个int型的数组对象。对于引用类型，写final域的重

排序规则对编译器和处理器增加了如下约束：在构造函数内对一个final引用的对象的成员域

的写入，与随后在构造函数外把这个被构造对象的引用赋值给一个引用变量，这两个操作之

间不能重排序。

对上面的示例程序，假设首先线程A执行writerOne\(\)方法，执行完后线程B执行

writerTwo\(\)方法，执行完后线程C执行reader\(\)方法。图3-31是一种可能的线程执行时序。

在图3-31中，1是对final域的写入，2是对这个final域引用的对象的成员域的写入，3是把被

构造的对象的引用赋值给某个引用变量。这里除了前面提到的1不能和3重排序外，2和3也不

能重排序。

JMM可以确保读线程C至少能看到写线程A在构造函数中对final引用对象的成员域的写

入。即C至少能看到数组下标0的值为1。而写线程B对数组元素的写入，读线程C可能看得到，

也可能看不到。JMM不保证线程B的写入对读线程C可见，因为写线程B和读线程C之间存在数

据竞争，此时的执行结果不可预知。

如果想要确保读线程C看到写线程B对数组元素的写入，写线程B和读线程C之间需要使

用同步原语（lock或volatile）来确保内存可见性。

