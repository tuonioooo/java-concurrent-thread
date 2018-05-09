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

    public CompletableFuture<Void> thenRun(Runnable action) {
        return uniRunStage(null, action);
    }

    public CompletableFuture<Void> thenRunAsync(Runnable action) {
        return uniRunStage(asyncPool, action);
    }

    public CompletableFuture<Void> thenRunAsync(Runnable action, Executor executor) {
        return uniRunStage(screenExecutor(executor), action);
    }

    public <U,V> CompletableFuture<V> thenCombine(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(null, other, fn);
    }

    public <U,V> CompletableFuture<V> thenCombineAsync(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn) {
        return biApplyStage(asyncPool, other, fn);
    }

    public <U,V> CompletableFuture<V> thenCombineAsync(
        CompletionStage<? extends U> other,
        BiFunction<? super T,? super U,? extends V> fn, Executor executor) {
        return biApplyStage(screenExecutor(executor), other, fn);
    }

    public <U> CompletableFuture<Void> thenAcceptBoth(
        CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action) {
        return biAcceptStage(null, other, action);
    }

    public <U> CompletableFuture<Void> thenAcceptBothAsync(
        CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action) {
        return biAcceptStage(asyncPool, other, action);
    }

    public <U> CompletableFuture<Void> thenAcceptBothAsync(
        CompletionStage<? extends U> other,
        BiConsumer<? super T, ? super U> action, Executor executor) {
        return biAcceptStage(screenExecutor(executor), other, action);
    }
    
    public CompletableFuture<Void> runAfterBoth(CompletionStage<?> other,
                                                Runnable action) {
        return biRunStage(null, other, action);
    }

    public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action) {
        return biRunStage(asyncPool, other, action);
    }

    public CompletableFuture<Void> runAfterBothAsync(CompletionStage<?> other,Runnable action, Executor executor) {
        return biRunStage(screenExecutor(executor), other, action);
    }
    public <U> CompletableFuture<U> applyToEither(
        CompletionStage<? extends T> other, Function<? super T, U> fn) {
        return orApplyStage(null, other, fn);
    }

    public <U> CompletableFuture<U> applyToEitherAsync(
        CompletionStage<? extends T> other, Function<? super T, U> fn) {
        return orApplyStage(asyncPool, other, fn);
    }

    public <U> CompletableFuture<U> applyToEitherAsync(
        CompletionStage<? extends T> other, Function<? super T, U> fn,
        Executor executor) {
        return orApplyStage(screenExecutor(executor), other, fn);
    }

    public CompletableFuture<Void> acceptEither(
        CompletionStage<? extends T> other, Consumer<? super T> action) {
        return orAcceptStage(null, other, action);
    }

    public CompletableFuture<Void> acceptEitherAsync(
        CompletionStage<? extends T> other, Consumer<? super T> action) {
        return orAcceptStage(asyncPool, other, action);
    }

    public CompletableFuture<Void> acceptEitherAsync(
        CompletionStage<? extends T> other, Consumer<? super T> action,
        Executor executor) {
        return orAcceptStage(screenExecutor(executor), other, action);
    }

    public CompletableFuture<Void> runAfterEither(CompletionStage<?> other,
                                                  Runnable action) {
        return orRunStage(null, other, action);
    }

    public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other,
                                                       Runnable action) {
        return orRunStage(asyncPool, other, action);
    }

    public CompletableFuture<Void> runAfterEitherAsync(CompletionStage<?> other,
                                                       Runnable action,
                                                       Executor executor) {
        return orRunStage(screenExecutor(executor), other, action);
    }

    public <U> CompletableFuture<U> thenCompose(
        Function<? super T, ? extends CompletionStage<U>> fn) {
        return uniComposeStage(null, fn);
    }

    public <U> CompletableFuture<U> thenComposeAsync(
        Function<? super T, ? extends CompletionStage<U>> fn) {
        return uniComposeStage(asyncPool, fn);
    }

    public <U> CompletableFuture<U> thenComposeAsync(
        Function<? super T, ? extends CompletionStage<U>> fn,
        Executor executor) {
        return uniComposeStage(screenExecutor(executor), fn);
    }

    public CompletableFuture<T> whenComplete(
        BiConsumer<? super T, ? super Throwable> action) {
        return uniWhenCompleteStage(null, action);
    }

    public CompletableFuture<T> whenCompleteAsync(
        BiConsumer<? super T, ? super Throwable> action) {
        return uniWhenCompleteStage(asyncPool, action);
    }

    public CompletableFuture<T> whenCompleteAsync(
        BiConsumer<? super T, ? super Throwable> action, Executor executor) {
        return uniWhenCompleteStage(screenExecutor(executor), action);
    }

    public <U> CompletableFuture<U> handle(
        BiFunction<? super T, Throwable, ? extends U> fn) {
        return uniHandleStage(null, fn);
    }

    public <U> CompletableFuture<U> handleAsync(
        BiFunction<? super T, Throwable, ? extends U> fn) {
        return uniHandleStage(asyncPool, fn);
    }

    public <U> CompletableFuture<U> handleAsync(
        BiFunction<? super T, Throwable, ? extends U> fn, Executor executor) {
        return uniHandleStage(screenExecutor(executor), fn);
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
```

* thenRun、thenRunAsync对上一步的计算结果不关心，执行下一个操作，他的入参是Runnable,无返回值，示例如下：

```
public static void thenRun(){
        IntStream stream = IntStream.of(10,9,8,7,6,5,4,3,2,1);
        Runnable runnable = ()->{
            stream.forEach(i->{
                try {
                    Thread.sleep(1000);
                    System.out.format("线程名称：%s, 倒计时：%d\n", Thread.currentThread().getName(), i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        };

        CompletableFuture.supplyAsync(()->{
            try {
                Thread.sleep(3000);
                System.out.format("线程名称：%s\n" , Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenRun(runnable);
        while (true){}

    }

    public static void thenRunAsync(){
        IntStream stream = IntStream.of(10,9,8,7,6,5,4,3,2,1);
        CompletableFuture.supplyAsync(()->{
            try {
                Thread.sleep(3000);
                System.out.format("线程名称：%s\n" , Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenRunAsync(()->{
            stream.forEach(i->{
                try {
                    Thread.sleep(1000);
                    System.out.format("线程名称：%s, 倒计时：%d\n", Thread.currentThread().getName(), i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        });
        while (true){}

    }

    public static void thenRunAsyncOfExecutor(){
        ExecutorService executor = Executors.newCachedThreadPool();
        IntStream stream = IntStream.of(10,9,8,7,6,5,4,3,2,1);
        CompletableFuture.supplyAsync(()->{
            try {
                Thread.sleep(3000);
                System.out.format("线程名称：%s\n" , Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenRunAsync(()->{
            stream.forEach(i->{
                try {
                    Thread.sleep(1000);
                    System.out.format("线程名称：%s, 倒计时：%d\n", Thread.currentThread().getName(), i);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });

            executor.shutdown();

        }, executor);

        while(true){
            if(executor.isTerminated()){
                System.out.println("线程任务都已经完成");
                break;
            }
        }

    }
```

* thenCombine、thenCombineAsync  它需要原来的处理返回值，利用这两个返回值，进行转换后返回指定类型的值。

```
public static void thenCombine() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
                System.out.println("线程名称：" + Thread.currentThread().getName());
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenCombine(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程名称：" + Thread.currentThread().getName());
            return "world";
        }), (s1, s2) -> s1 + " " + s2).join();
        System.out.println(result);
    }


    public static void thenCombineAsync() {
        ExecutorService executor = Executors.newFixedThreadPool(1);
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程名称：" + Thread.currentThread().getName());
            return "hello";
        }).thenCombineAsync(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println("线程名称：" + Thread.currentThread().getName());
            return "world";
        }), (s1, s2) -> s1 + " " + s2, executor).join();
        executor.shutdown();
        System.out.println(result);
    }
```

* thenAcceptBoth、thenAcceptBothAsync 它需要原来的处理返回值，并且other代表的CompletionStage也要返回值之后，利用这两个返回值，进行消耗

```
public static void thenAcceptBoth() {
        ExecutorService executor = Executors.newFixedThreadPool(1);
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello";
        }).thenAcceptBothAsync(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }

            return "world";
        }), (s1, s2) ->{
            System.out.println(s1 + " " + s2);
            executor.shutdown();
        }, executor);

        while(true){
            if(executor.isTerminated()){
                System.out.println("线程任务都已经完成");
                break;
            }
        }
    }
```

* runAfterBoth、runAfterBothAsync 不关心这两个CompletionStage的结果，只关心这两个CompletionStage执行完毕，之后在进行操作（Runnable）。

```
public static void runAfterBoth(){
        ExecutorService executor = Executors.newFixedThreadPool(1);
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).runAfterBothAsync(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s2";
        }), () -> {
            System.out.println("hello world");
            executor.shutdown();
        }, executor);

        while(true){
            if(executor.isTerminated()){
                System.out.println("线程任务都已经完成");
                break;
            }
        }
    }
    
    //hello world
    //线程任务都已经完成
```

* applyToEither、applyToEitherAsync 两个CompletionStage，谁计算的快，我就用那个CompletionStage的结果进行下一步的转化操作。我们现实开发场景中，总会碰到有两种渠道完成同一个事情，所以就可以调用这个方法，找一个最快的结果进行处理。
* 示例如下：

```
public static void acceptEither() {
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).acceptEither(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "hello world";
        }), System.out::println);
        while (true){}
    }
```

* runAfterEither、runAfterEitherAsync 两个CompletionStage，任何一个完成了都会执行下一步的操作（Runnable）示例如下：

```
public static void runAfterEither() {
        CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s1";
        }).runAfterEither(CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            return "s2";
        }), () -> System.out.println("hello world"));
        while (true) {
        }
    }
```

* 当运行时出现了异常，可以通过exceptionally进行补偿

```
public CompletableFuture<T> exceptionally(
        Function<Throwable, ? extends T> fn) {
        return uniExceptionallyStage(fn);
    }
```

```
public static void exceptionally() {
        String result = CompletableFuture.supplyAsync(() -> {
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            if (1 == 1) {
                throw new RuntimeException("测试一下异常情况");
            }
            return "s1";
        }).exceptionally(e -> {
            System.out.println(e.getMessage());
            return "hello world";
        }).join();
        System.out.println(result);
    }
    
    //java.lang.RuntimeException: 测试一下异常情况
    //hello world
```

* whenComplete、whenCompleteAsync  当运行完成时，对结果的记录。这里的完成时有两种情况，一种是正常执行，返回值。另外一种是遇到异常抛出造成程序的中断。这里为什么要说成记录，因为这几个方法都会返回CompletableFuture，当Action执行完毕后它的结果返回原始的CompletableFuture的计算结果或者返回异常。所以不会对结果产生任何的作用。  示例如下：

```

```



