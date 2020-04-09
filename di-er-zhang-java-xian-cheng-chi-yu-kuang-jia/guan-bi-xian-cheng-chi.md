# 关闭线程池

可以通过调用线程池的shutdown或shutdownNow方法来关闭线程池。它们的原理是遍历线程池中的工作线程，然后逐个调用线程的interrupt方法来中断线程，所以无法响应中断的任务可能永远无法终止。但是它们存在一定的区别，shutdownNow首先将线程池的状态设置成STOP，然后尝试停止所有的正在执行或暂停任务的线程，并返回等待执行任务的列表，而shutdown只是将线程池的状态设置成SHUTDOWN状态，然后中断所有没有正在执行任务的线程。

只要调用了这两个关闭方法中的任意一个，isShutdown方法就会返回true。当所有的任务都已关闭后，才表示线程池关闭成功，这时调用isTerminaed方法会返回true。

通常如果配置线程池的话，是不需要关闭的。

先了解如下方法：

**shutdown\(\)方法**：

* 停止接收外部submit的任务
* 内部正在跑的任务和队列里等待的任务，会执行完
* 等到第二步完成后，才真正停止

**shutdownNow\(\)方法**：**企图**立即停止，事实上不一定\(比如死循环的任务\)。

* 跟shutdown\(\)一样，先停止接收外部提交的任务
* 对于尚未执行的任务，全部取消掉
* 对于正在执行的任务，发出interrupt\(\)

> 它试图终止线程的方法是通过调用`Thread.interrupt()`方法来实现的，但是大家知道，这种方法的作用有限，如果线程中**没有sleep 、wait、Condition、定时锁等应用**, interrupt\(\)方法是**无法中断当前的线程**的。所以，ShutdownNow\(\)并不代表线程池就一定立即就能退出，它也可能必须要等待所有正在执行的任务都执行完成了才能退出。但是大多数时候是能立即退出的。

**awaitTermination**\(long timeOut, TimeUnit unit\)  
方法：当前线程阻塞，直到

* 等所有已提交的任务（包括正在跑的和队列中等待的）执行完
* 或者等超时时间到
* 或者线程被中断，抛出`InterruptedException`

然后返回true（shutdown请求后所有任务执行完毕）或false（已超时）

> 实验发现，shuntdown\(\)和awaitTermination\(\)效果差不多，方法执行之后，都要等到提交的任务全部执行完才停。

**shutdown\(\)和shutdownNow\(\)的区别**

> shutdownNow\(\)能立即停止线程池，正在跑的和正在等待的任务都停下了。这样做立即生效，但是风险也比较大；
>
> shutdown\(\)只是关闭了提交通道，用submit\(\)是无效的；而内部该怎么跑还是怎么跑，跑完再停。

**shutdown\(\)和awaitTermination\(\)的区别**

> shutdown\(\)后，不能再提交新的任务进去；但是awaitTermination\(\)后，可以继续提交。 awaitTermination\(\)是阻塞的，返回结果是线程池是否已停止（true/false）；shutdown\(\)不阻塞。

## 总结 <a id="&#x603B;&#x7ED3;"></a>

* 优雅的关闭，用shutdown\(\)
* 想立马关闭，并得到未执行任务列表，用shutdownNow\(\)
* 优雅的关闭，并允许关闭声明后新任务能提交，用awaitTermination\(\)

常用的线程池关闭示例如下：

1.直接使用代码shutdown\(\)/shutdownNow\(\)

```text
executor.shutdown();//可以使用，但不优雅
```

```text
executor.shutdownNow();//风险太大不建议直接使用
```

2.优雅的方式

```text
/**
 * @author Benjamin Winterberg
 */
public class ConcurrentUtils {

    public static void stop(ExecutorService executor) {
        try {
            executor.shutdown();
            executor.awaitTermination(60, TimeUnit.SECONDS);//时间可以根据业务场景，设置大小
            /*
            阻塞的时候，可以做一些其他操作，这段注释代码可以忽略
            while(!executor.isTerminated()){
                System.out.println("running...");
            }
            */

            /*
            阻塞的时候，可以做一些其他操作，这段注释代码可以忽略
            while (true){
                if(executor.isTerminated()){
                    System.out.println("线程任务都已经完成");
                    break;
                }
            }
            */
        }
        catch (InterruptedException e) {
            System.err.println("termination interrupted");
        }
        finally {
            if (!executor.isTerminated()) {//isTerminated() 一定在shutdown()后面使用，否则永远是false
                System.err.println("killing non-finished tasks");
            }
            executor.shutdownNow();
        }
    }

    public static void sleep(int seconds) {
        try {
            TimeUnit.SECONDS.sleep(seconds);
        } catch (InterruptedException e) {
            throw new IllegalStateException(e);
        }
    }

}
```

