# CompletionService讲解

## 应用场景 <a id="&#x5E94;&#x7528;&#x573A;&#x666F;"></a>

当向Executor提交多个任务并且希望获得它们在完成之后的结果，如果用FutureTask，可以循环获取task，并调用get方法去获取task执行结果，但是如果task还未完成，获取结果的线程将阻塞直到task完成，由于不知道哪个task优先执行完毕，使用这种方式效率不会很高。在jdk5时候提出接口CompletionService，它整合了Executor和BlockingQueue的功能，可以更加方便在多个任务执行时获取到任务执行结果。

### 案例 <a id="&#x6848;&#x4F8B;"></a>

* **需求**：不使用求和公式，计算从1到100000000相加的和。
* **分析设计**：需求指明不能使用求和公式，只能循环依次相加，为了提高效率，我们可以将1到100000000的数分为n段由n个task执行，执行结束后merge结果求最后的和。
* **代码实现**： ![CompletionService&#x4F7F;&#x7528;&#x6848;&#x4F8B;](http://upload-images.jianshu.io/upload_images/3994601-f62c2f904cc23bfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
  * 声明task执行载体，线程池executor；
  * 声明CompletionService，通过参数指定执行task的线程池，存放已完成状态task的阻塞队列，队列默认为基于链表结构的阻塞队列LinkedBlockingQueue；
  * 调用submit方法提交task；
  * 调用take方法获取已完成状态task。

## CompletionService源码分析 <a id="completionservice&#x6E90;&#x7801;&#x5206;&#x6790;"></a>

CompletionService接口提供五个方法：

* `Future<V> submit(Callable<V> task)` 提交Callable类型的task；
* `Future<V> submit(Runnable task, V result)` 提交Runnable类型的task；
* `Future<V> take() throws InterruptedException` 获取并移除已完成状态的task，如果目前不存在这样的task，则等待；
* `Future<V> poll()` 获取并移除已完成状态的task，如果目前不存在这样的task，返回null；
* `Future<V> poll(long timeout, TimeUnit unit) throws InterruptedException` 获取并移除已完成状态的task，如果在指定等待时间内不存在这样的task，返回null。

接下来我们来看看CompletionService接口的具体实现：ExecutorCompletionService。

**ExecutorCompletionService实现分析**

* 成员变量 ![&#x6210;&#x5458;&#x53D8;&#x91CF;](http://upload-images.jianshu.io/upload_images/3994601-cd745dcd8d74d31e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ExecutorCompletionService有三个成员变量：
  * executor：执行task的线程池，创建CompletionService必须指定；
  * aes：主要用于创建待执行task；
  * completionQueue：存储已完成状态的task，默认是基于链表结构的阻塞队列LinkedBlockingQueue。
* 构造方法 ![&#x6784;&#x9020;&#x65B9;&#x6CD5;](http://upload-images.jianshu.io/upload_images/3994601-06a5c5087e295311.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) ExecutorCompletionService提供两个构造方法，具体的使用具体情况具体分析，使用者可以根据业务场景来进行选择。
* task提交  
  ExecutorCompletionService提供submit方法来提交Callable类型或者Runnable类型的task：  
  ![&#x7EBF;&#x7A0B;&#x63D0;&#x4EA4;.png](http://upload-images.jianshu.io/upload_images/3994601-615b4bc31f66428b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
  具体的执行流程如下：

  1. 参数校验，不符合条件的task抛出异常，程序结束；
  2. 将Callable类型或者Runnable类型的task构造成FutureTask；
  3. 把构造好的FutureTask交由线程池executor执行。

  看到这里可能大家会比较疑惑了，task调用submit方法可以提交，**完成的task是什么时候被加入到completionQueue里的呢？**

  针对这个问题，从submit方法的源码可以看出，在提交到线程池的时候需要将FutureTask封装成QueueingFuture，我们来看看QueueingFuture的具体实现：  
  ![QueueingFuture&#x5B9E;&#x73B0;](http://upload-images.jianshu.io/upload_images/3994601-4349ad3b95ae05f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
  从源码可以看出，QueueingFuture是FutureTask的子类，实现了done方法，在task执行完成之后将当前task添加到completionQueue，done方法的具体调用在FutureTask的finishCompletion方法，上篇介绍FutureTask的文章已经做过具体的分析，在这里就不再赘述了。

* 已完成状态task获取 CompletionService的take方法和poll方法都可以获取已完成状态的task，我们来看看具体的实现： ![take&#x3001;poll&#x5B9E;&#x73B0;](http://upload-images.jianshu.io/upload_images/3994601-2821d2e5f8ae8167.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240) 从源码可以看出，take和poll都是调用BlockingQueue提供的方法。既然take和poll都可以获取到已完成状态的task，那么他们的区别是什么呢？
  * take在获取并移除已完成状态的task时，如果目前暂时不存在这样的task，等待，直到存在这样的task；
  * poll在获取并移除已完成状态的task时，如果目前暂时不存在这样的task，不等待，直接返回null。

