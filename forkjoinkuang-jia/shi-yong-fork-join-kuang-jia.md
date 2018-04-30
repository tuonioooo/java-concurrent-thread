# 使用Fork/Join框架

让我们通过一个简单的需求来使用Fork/Join框架，需求是：计算1+2+3+4的结果。

使用Fork/Join框架首先要考虑到的是如何分割任务，如果希望每个子任务最多执行两个

数的相加，那么我们设置分割的阈值是2，由于是4个数字相加，所以Fork/Join框架会把这个任

务fork成两个子任务，子任务一负责计算1+2，子任务二负责计算3+4，然后再join两个子任务

的结果。因为是有结果的任务，所以必须继承RecursiveTask，实现代码如下。

```
package fj;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.Future;
import java.util.concurrent.RecursiveTask;
public class CountTask extends RecursiveTask<Integer> {
private static final int THRESHOLD = 2;　　// 阈值
private int start;
private int end;
public CountTask(int start, int end) {
this.start = start;
this.end = end;
}
@Override
protected Integer compute() {
int sum = 0;
// 如果任务足够小就计算任务
boolean canCompute = (end - start) <= THRESHOLD;
if (canCompute) {
for (int i = start; i <= end; i++) {
sum += i;
}
} else {
// 如果任务大于阈值，就分裂成两个子任务计算
int middle = (start + end) / 2;
CountTask leftTask = new CountTask(start, middle);
CountTask rightTask = new CountTask(middle + 1, end);
// 执行子任务
leftTask.fork();
rightTask.fork();
// 等待子任务执行完，并得到其结果
int leftResult=leftTask.join();
int rightResult=rightTask.join();
// 合并子任务
sum = leftResult + rightResult;
}
return sum;
}
public static void main(String[] args) {
ForkJoinPool forkJoinPool = new ForkJoinPool();
// 生成一个计算任务，负责计算1+2+3+4
CountTask task = new CountTask(1, 4);
// 执行一个任务
Future<Integer> result = forkJoinPool.submit(task);
try {
System.out.println(result.get());
} catch (InterruptedException e) {
} catch (ExecutionException e) {
}
}
}
```

通过这个例子，我们进一步了解ForkJoinTask，ForkJoinTask与一般任务的主要区别在于它

需要实现compute方法，在这个方法里，首先需要判断任务是否足够小，如果足够小就直接执

行任务。如果不足够小，就必须分割成两个子任务，每个子任务在调用fork方法时，又会进入

compute方法，看看当前子任务是否需要继续分割成子任务，如果不需要继续分割，则执行当

前子任务并返回结果。使用join方法会等待子任务执行完并得到其结果。

