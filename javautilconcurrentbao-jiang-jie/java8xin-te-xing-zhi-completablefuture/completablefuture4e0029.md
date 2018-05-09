# CompletableFuture\(Ⅰ\)

completableFuture实现了CompletionStage接口，如下：

```
    public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
        return uniAcceptStage(null, action);
    }

    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
        return uniAcceptStage(asyncPool, action);
    }

    public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,Executor executor) {
        return uniAcceptStage(screenExecutor(executor), action);
    }

    public <U> CompletableFuture<U> thenApply(Function<? super T,? extends U> fn) {
        return uniApplyStage(null, fn);
    }

    public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn) {
        return uniApplyStage(asyncPool, fn);
    }

    public <U> CompletableFuture<U> thenApplyAsync(Function<? super T,? extends U> fn, Executor executor) {
        return uniApplyStage(screenExecutor(executor), fn);
    }
```

首先说明一下已Async结尾的方法都是可以异步执行的，如果指定了线程池，会在指定的线程池中执行，如果没有指定，默认会在ForkJoinPool.commonPool\(\)中执行，下文中将会有好多类似的，都不详细解释了。关键的入参只有一个Function，它是函数式接口，所以使用Lambda表示起来会更加优雅。它的入参是上一个阶段计算后的结果，返回值是经过转化后结果。

* 初始化CompletableFuture的几种方式:

```
public static void init(){
        //方式一
        CompletableFuture<String> completableFuture1 = new CompletableFuture();
        //方式二
        CompletableFuture<Integer> completableFuture2 = CompletableFuture.completedFuture(111);
        CompletableFuture<String>  completableFuture3 = CompletableFuture.completedFuture("123");
        //方式三
        CompletableFuture<String>  completableFuture4 = CompletableFuture.supplyAsync(()->"123");
        CompletableFuture<Integer>  completableFuture5 = CompletableFuture.supplyAsync(()->123);
        CompletableFuture  completableFuture6 = CompletableFuture.supplyAsync(()->123);
        CompletableFuture  completableFuture7 = CompletableFuture.supplyAsync(()->"123");

        //方式四  创建自定义的executor
        ExecutorService executor = Executors.newFixedThreadPool(1);
        CompletableFuture.supplyAsync(()->"123", executor);
        try {
            executor.shutdownNow();
            executor.awaitTermination(5, TimeUnit.SECONDS);//在指定时间内，终止任务
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

    }
```

* thenAccpet、thenAcceptAsync 是针对结果进行消耗，因为他的入参是Consumer，有入参无返回值 示例如下：

```
    public static void thenAccept(){
        CompletableFuture.supplyAsync(()->"hello")
                .thenAccept(s -> System.out.println(s + " world"))
                .thenAccept(v-> System.out.println("done"));

        //hello world
        //done
    }

    public static void thenAcceptAsync(){//默认使用ForkJoinPool.commonPool()
        CompletableFuture.supplyAsync(()->"hello").thenAcceptAsync(s -> System.out.println(s + " world "));
        System.out.println("异步任务执行");

        //异步任务执行
        //hello world
    }

    public static void thenAcceptAsyncOfExecutor(){//创建自定义的executor
        ExecutorService executor = Executors.newFixedThreadPool(1);
        CompletableFuture.supplyAsync(()->"hello").thenAcceptAsync(s -> System.out.println(s + " world "), executor);
        try {
            executor.shutdownNow();
            executor.awaitTermination(5, TimeUnit.SECONDS);//在指定时间内，终止任务
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("异步任务执行");
    }
```

* thenApply、thenApplyAsync 是针对结果进行转换，有返回值，他的入参是Function，可以使用lambda表达式，示例如下：

```
   public static void thenApply(){
        String result = CompletableFuture.supplyAsync(()->"hello")
                .thenApply(s -> s + " world")
                .thenApply(v-> v + " done").join();
        System.out.println("result = " + result);
        //hello world done
    }

    public static void thenApplyAsync(){
        ExecutorService executor = Executors.newSingleThreadExecutor();
        String result = CompletableFuture.supplyAsync(()->"hello")
                .thenApplyAsync(s -> s + " world")
                .thenApplyAsync(v-> v + ", 异步线程名：" + Thread.currentThread().getName()).join();
        System.out.println("result = " + result);
        System.out.println("主线程名：" + Thread.currentThread().getName());
        //result = hello world, 异步线程名：ForkJoinPool.commonPool-worker-1
        //主线程名：main
    }

    public static void thenApplyOfExecutor(){
        String result = CompletableFuture.supplyAsync(()->"hello")
                .thenApplyAsync(s -> s + " world")
                .thenApplyAsync(v-> v + ", 异步线程名：" + Thread.currentThread().getName()).join();
        System.out.println("result = " + result);
        System.out.println("主线程名：" + Thread.currentThread().getName());
        //result = hello world, 异步线程名：ForkJoinPool.commonPool-worker-1
        //主线程名：main
    }
```



















