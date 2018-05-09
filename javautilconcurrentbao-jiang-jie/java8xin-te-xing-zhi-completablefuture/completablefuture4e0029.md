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

```
public static void thenAccept1(){
        CompletableFuture<String> future = new CompletableFuture<>();
        future.complete("42");
        future
                .thenAccept(System.out::println)
                .thenAccept(v -> System.out.println("done"));
        //42
        //done
 }

public static void thenAccept2(){
        String result = CompletableFuture.supplyAsync(() -> "hello").thenApply(s -> s + " world").join();
        System.out.println(result);//hello world
}
```



