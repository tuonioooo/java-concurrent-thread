# ThreadLocal关键字

ThreadLocal，即线程变量，是一个以ThreadLocal对象为键、任意对象为值的存储结构。这 个结构被附带在线程上，也就是说一个线程可以根据一个ThreadLocal对象查询到绑定在这个 线程上的一个值。 可以通过set\(T\)方法来设置一个值，在当前线程下再通过get\(\)方法获取到原先设置的值。 在代码清单1所示的例子中，构建了一个常用的Profiler类，它具有begin\(\)和end\(\)两个 方法，而end\(\)方法返回从begin\(\)方法调用开始到end\(\)方法被调用时的时间差，单位是毫秒。

代码清单1 Profiler.java

```text
public class Profiler {
    // 第一次get()方法调用时会进行初始化（如果set方法没有调用），每个线程会调用一次
    private static final ThreadLocal<Long> TIME_THREADLOCAL = new ThreadLocal<Long>() {
        protected Long initialValue() {
            return System.currentTimeMillis();
        }
    };
    public static final void begin() {
        TIME_THREADLOCAL.set(System.currentTimeMillis());
    }
    public static final long end() {
        return System.currentTimeMillis() - TIME_THREADLOCAL.get();
    }
    public static void main(String[] args) throws Exception {
        Profiler.begin();
        TimeUnit.SECONDS.sleep(1);
        System.out.println("Cost: " + Profiler.end() + " mills");
    }
}
```

输出结果如下所示。

Cost: 1001 mills

Profiler可以被复用在方法调用耗时统计的功能上，在方法的入口前执行begin\(\)方法，在 方法调用后执行end\(\)方法，好处是两个方法的调用不用在一个方法或者类中，比如在AOP（面 向方面编程）中，可以在方法调用前的切入点执行begin\(\)方法，而在方法调用后的切入点执行 end\(\)方法，这样依旧可以获得方法的执行耗时。

先关文档参考：

百度百科：[https://baike.baidu.com/item/ThreadLocal/4915311?fr=aladdin](https://baike.baidu.com/item/ThreadLocal/4915311?fr=aladdin)

