# RejectedExecutionException

引发java.util.concurrent.RejectedExecutionException**主要有两种原因**：

1. 线程池显示的调用了shutdown\(\)之后，再向线程池提交任务的时候，如果你配置的拒绝策略是ThreadPoolExecutor.AbortPolicy的话，这个异常就被会抛出来。

2. 当你的排队策略为有界队列，并且配置的拒绝策略是ThreadPoolExecutor.AbortPolicy，当线程池的线程数量已经达到了maximumPoolSize的时候，你再向它提交任务，就会抛出ThreadPoolExecutor.AbortPolicy异常。

### 显示关闭掉线程池

这一点很好理解。比如说，你向一个仓库去存放货物，一开始，仓库管理员把门给你打开了，你放了第一件商品到仓库里，但是当你放好出去后，有人把仓库门关了，那你下次再来存放物品时，你就会被拒绝。示例代码如下：

```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  


public class TextExecutor {  
    public ExecutorService fixedExecutorService = Executors.newFixedThreadPool(5);  
    public ExecutorService cachedExecutorService = Executors.newCachedThreadPool();  
    public ExecutorService singleExecutorService = Executors.newSingleThreadExecutor();  

    public void testExecutorException() {  
        for (int i = 0; i < 10; i ++) {  
            fixedExecutorService.execute(new SayHelloRunnable());  
            fixedExecutorService.shutdown();  
        }  
    }  

    private class SayHelloRunnable implements Runnable {  

        @Override  
        public void run() {  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            } finally {  
                System.out.println("hello world!");  
            }  

        }  
    }  

    public static void main(String[] args) {  
        TextExecutor testExecutor = new TextExecutor();  
        testExecutor.testExecutorException();  
    }  
}
```

### 解决方案

1. 不要显示的调用shutdown方法，例如Android里，只有你在Destory方法里cancel掉AsyncTask，则线程池里没有活跃线程会自己回收自己。

2. 调用线程池时，判断是否已经shutdown，通过API方法isShutDown方法判断，示例代码：

```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  


public class TextExecutor {  
    public ExecutorService fixedExecutorService = Executors.newFixedThreadPool(5);  
    public ExecutorService cachedExecutorService = Executors.newCachedThreadPool();  
    public ExecutorService singleExecutorService = Executors.newSingleThreadExecutor();  

    public void testExecutorException() {  
        for (int i = 0; i < 10; i ++) {  
            // 增加isShutdown()判断  
            if (!fixedExecutorService.isShutdown()) {  
                fixedExecutorService.execute(new SayHelloRunnable());  
            }  
            fixedExecutorService.shutdown();  
        }  
    }  

    private class SayHelloRunnable implements Runnable {  

        @Override  
        public void run() {  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            } finally {  
                System.out.println("hello world!");  
            }  

        }  
    }  

    public static void main(String[] args) {  
        TextExecutor testExecutor = new TextExecutor();  
        testExecutor.testExecutorException();  
    }  
}
```

### 线程数量超过maximumPoolSize

示例代码里使用了自定义的ExecutorService，可以复现这种问题：

```
import java.util.concurrent.ExecutorService;  
import java.util.concurrent.Executors;  
import java.util.concurrent.SynchronousQueue;  
import java.util.concurrent.ThreadPoolExecutor;  
import java.util.concurrent.TimeUnit;  


public class TextExecutor {  
    public ExecutorService fixedExecutorService = Executors.newFixedThreadPool(5);  
    public ExecutorService cachedExecutorService = Executors.newCachedThreadPool();  
    public ExecutorService singleExecutorService = Executors.newSingleThreadExecutor();  
    public ExecutorService customerExecutorService = new ThreadPoolExecutor(3, 5, 0, TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>());  

    public void testExecutorException() {  
        for (int i = 0; i < 10; i ++) {  
            // 增加isShutdown()判断  
            if (!fixedExecutorService.isShutdown()) {  
                fixedExecutorService.execute(new SayHelloRunnable());  
            }  
            fixedExecutorService.shutdown();  
        }  
    }  

    public void testCustomerExecutorException() {  
        for (int i = 0; i < 100; i ++) {  
            customerExecutorService.execute(new SayHelloRunnable());  
        }  
    }  

    private class SayHelloRunnable implements Runnable {  

        @Override  
        public void run() {  
            try {  
                Thread.sleep(1000);  
            } catch (InterruptedException e) {  
                // TODO Auto-generated catch block  
                e.printStackTrace();  
            } finally {  
                System.out.println("hello world!");  
            }  

        }  
    }  

    public static void main(String[] args) {  
        TextExecutor testExecutor = new TextExecutor();  
        testExecutor.testCustomerExecutorException();;  
    }  
}
```

### 解决方案

1. 尽量调大maximumPoolSize，例如设置为Integer.MAX\_VALUE

```
public ExecutorService customerExecutorService = new ThreadPoolExecutor(3, Integer.MAX_VALUE, 0, TimeUnit.MILLISECONDS, new SynchronousQueue<Runnable>());
```

   2.使用其他排队策略，例如LinkedBlockingQueue

```
public ExecutorService customerExecutorService = new ThreadPoolExecutor(3, 5, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<Runnable>());
```



