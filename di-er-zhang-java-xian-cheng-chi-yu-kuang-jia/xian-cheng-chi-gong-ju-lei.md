# 线程池工具类

```
package com.ise.api.pool.threadpoolexecutor;

import java.util.concurrent.*;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-4-20
 * Time: 10:06
 * info: 自定义线程池
 */
public class ThreadPoolConfig {

    private static ThreadPoolExecutor executor;

    /**返回可用处理器的Java虚拟机的数量*/
    private static int PROCESSORS = Runtime.getRuntime().availableProcessors();
    /**线程池中所保存的核心线程数，包括空闲线程*/
    private static int corePoolSize = 2;
    /**池中允许的最大线程数*/
    private static int maximumPoolSize = 6;
    /**线程池中的空闲线程所能持续的最长时间*/
    private static int keepAliveTime = 1;
    /**持续时间的单位*/
    private static TimeUnit timeUnit = TimeUnit.DAYS;
    /**队列容量*/
    private static int capacity = 50000;

    private static LinkedBlockingQueue<Runnable> queue;

    private static class SingletonClassInstance {
        private static final ThreadPoolConfig instance = new ThreadPoolConfig();
    }

    private ThreadPoolConfig(){}


    public static ThreadPoolConfig getInstance(){
        return SingletonClassInstance.instance;
    }

    public ThreadPoolExecutor init(){
        if(executor == null){
            queue = new LinkedBlockingQueue<Runnable>(capacity);
            executor = new ThreadPoolExecutor(corePoolSize, maximumPoolSize, keepAliveTime, timeUnit, queue);
        }
        return executor;
    }

    public int getQueueSize() {
        return queue.size();
    }

    public boolean isQueueFull() {
        return queue.size() == capacity;
    }

    public boolean isBusy() {
        return executor.getPoolSize() == maximumPoolSize && isQueueFull();
    }

    public int getPoolSize() {
        return executor.getPoolSize();
    }

    public Future<?> submit(Callable t){
        return executor.submit(t);
    }

    public void shutdown(){
        executor.shutdown();
    }
}
```

测试类

```
package com.ise.api.pool.threadpoolexecutor;

import java.util.concurrent.ThreadPoolExecutor;

/**
 * Created by daizhao.
 * User: tony
 * Date: 2018-4-20
 * Time: 10:25
 * info:
 */
public class RoomTask implements Runnable {

    @Override
    public void run() {
        System.out.println("roomTask测试");
    }


    public static void main(String[] args) {
        ThreadPoolConfig threadPoolConfig = ThreadPoolConfig.getInstance();
        ThreadPoolExecutor executor = threadPoolConfig.init();
        for (int i = 0; i < 10; i++) {
            executor.execute(new Thread(new RoomTask(), "TestThread".concat(""+i)));

        }
        System.out.println("线程队列大小为-->" + executor.getCorePoolSize());
        System.out.println("队列是否繁忙-->" + threadPoolConfig.isBusy());
        System.out.println("队列是否满员-->" + threadPoolConfig.isQueueFull());
        System.out.println("线程池数量-->" + threadPoolConfig.getPoolSize());
        System.out.println("线程池队列数量-->" + threadPoolConfig.getQueueSize());
        executor.shutdown();
    }
}

```



