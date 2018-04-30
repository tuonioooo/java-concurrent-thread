# Fork/Join框架的异常处理

ForkJoinTask在执行的时候可能会抛出异常，但是我们没办法在主线程里直接捕获异常，

所以ForkJoinTask提供了isCompletedAbnormally\(\)方法来检查任务是否已经抛出异常或已经被

取消了，并且可以通过ForkJoinTask的getException方法获取异常。使用如下代码。

```
if(task.isCompletedAbnormally())
{
    System.out.println(task.getException());
}
```

getException方法返回Throwable对象，如果任务被取消了则返回CancellationException。如

果任务没有完成或者没有抛出异常则返回null。

