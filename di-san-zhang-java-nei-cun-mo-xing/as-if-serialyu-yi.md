as-if-serial语义

as-if-serial语义的意思是：不管怎么重排序（编译器和处理器为了提高并行度），（单线程）

程序的执行结果不能被改变。编译器、runtime和处理器都必须遵守as-if-serial语义。

为了遵守as-if-serial语义，编译器和处理器不会对存在数据依赖关系的操作做重排序，因

为这种重排序会改变执行结果。但是，如果操作之间不存在数据依赖关系，这些操作就可能被

编译器和处理器重排序。为了具体说明，请看下面计算圆面积的代码示例。

---

double pi = 3.14; // A

double r = 1.0; // B

double area = pi \* r \* r; // C

---

上面3个操作的数据依赖关系如图1所示。

图1

![](/assets/import-serial-1.png)

如图1所示，A和C之间存在数据依赖关系，同时B和C之间也存在数据依赖关系。因此在

最终执行的指令序列中，C不能被重排序到A和B的前面（C排到A和B的前面，程序的结果将会

被改变）。但A和B之间没有数据依赖关系，编译器和处理器可以重排序A和B之间的执行顺序。

图2是该程序的两种执行顺序。

图2

![](/assets/import-serial-2.png)
