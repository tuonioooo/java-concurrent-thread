# 程序顺序规则

根据happens-before的程序顺序规则，上面计算圆的面积的示例代码存在3个happens-

before关系。

1）A　happens-before B。

2）B　happens-before C。

3）A　happens-before C。

这里的第3个happens-before关系，是根据happens-before的传递性推导出来的。

这里A happens-before B，但实际执行时B却可以排在A之前执行（看上面的重排序后的执

行顺序）。如果A happens-before B，JMM并不要求A一定要在B之前执行。JMM仅仅要求前一个

操作（执行的结果）对后一个操作可见，且前一个操作按顺序排在第二个操作之前。这里操作A

的执行结果不需要对操作B可见；而且重排序操作A和操作B后的执行结果，与操作A和操作B

按happens-before顺序执行的结果一致。在这种情况下，JMM会认为这种重排序并不非法（not

illegal），JMM允许这种重排序。

在计算机中，软件技术和硬件技术有一个共同的目标：在不改变程序执行结果的前提下，

尽可能提高并行度。编译器和处理器遵从这一目标，从happens-before的定义我们可以看出，

JMM同样遵从这一目标。

