# Executor 框架

在 Java 5 之后，并发编程引入了一堆新的启动、调度和管理线程的API。Executor 框架便是 Java 5 中引入的，其内部使用了线程池机制，它在 java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在 Java 5之后，通过 Executor 来启动线程比使用 Thread 的 start 方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免 this 逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象用 Executor 在构造器中。

Executor 框架包括：线程池，Executor，Executors，ExecutorService，CompletionService，Future，Callable 等。

Executor 接口中之定义了一个方法 execute（Runnable command），该方法接收一个 Runable 实例，它用来执行一个任务，任务即一个实现了 Runnable 接口的类。ExecutorService 接口继承自 Executor 接口，它提供了更丰富的实现多线程的方法，比如，ExecutorService 提供了关闭自己的方法，以及可为跟踪一个或多个异步任务执行状况而生成 Future 的方法。 可以调用 ExecutorService 的 shutdown（）方法来平滑地关闭 ExecutorService，调用该方法后，将导致 ExecutorService 停止接受任何新的任务且等待已经提交的任务执行完成\(已经提交的任务会分两类：一类是已经在执行的，另一类是还没有开始执行的\)，当所有已经提交的任务执行完毕后将会关闭 ExecutorService。因此我们一般用该接口来实现和管理多线程。

ExecutorService 的生命周期包括三种状态：运行、关闭、终止。创建后便进入运行状态，当调用了 shutdown（）方法时，便进入关闭状态，此时意味着 ExecutorService 不再接受新的任务，但它还在执行已经提交了的任务，当素有已经提交了的任务执行完后，便到达终止状态。如果不调用 shutdown（）方法，ExecutorService 会一直处在运行状态，不断接收新的任务，执行新的任务，服务器端一般不需要关闭它，保持一直运行即可。

Executors 提供了一系列工厂方法用于创先线程池，返回的线程池都实现了 ExecutorService 接口。

```
创建固定数目线程的线程池。
public static ExecutorService newFixedThreadPool(int nThreads)

创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线 程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。
public static ExecutorService newCachedThreadPool()

创建一个单线程化的Executor。
public static ExecutorService newSingleThreadExecutor()

创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。
public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)
```

这四种方法都是用的 Executors 中的 ThreadFactory 建立的线程，下面就以上四个方法做个比较：

**newCachedThreadPool\(\)**

* 缓存型池子，先查看池中有没有以前建立的线程，如果有，就 reuse 如果没有，就建一个新的线程加入池中
* 缓存型池子通常用于执行一些生存期很短的异步型任务 因此在一些面向连接的 daemon 型 SERVER 中用得不多。但对于生存期短的异步任务，它是 Executor 的首选。
* 能 reuse 的线程，必须是 timeout IDLE 内的池中线程，缺省 timeout 是 60s,超过这个 IDLE 时长，线程实例将被终止及移出池。

  > 注意，放入 CachedThreadPool 的线程不必担心其结束，超过 TIMEOUT 不活动，其会自动被终止。

**newFixedThreadPool\(int\)**

* newFixedThreadPool 与 cacheThreadPool 差不多，也是能 reuse 就用，但不能随时建新的线程。
* 其独特之处:任意时间点，最多只能有固定数目的活动线程存在，此时如果有新的线程要建立，只能放在另外的队列中等待，直到当前的线程中某个线程终止直接被移出池子。
* 和 cacheThreadPool 不同，FixedThreadPool 没有 IDLE 机制（可能也有，但既然文档没提，肯定非常长，类似依赖上层的 TCP 或 UDP IDLE 机制之类的），所以 FixedThreadPool 多数针对一些很稳定很固定的正规并发线程，多用于服务器。
* 从方法的源代码看，cache池和fixed 池调用的是同一个底层 池，只不过参数不同:
  * fixed 池线程数固定，并且是0秒IDLE（无IDLE）。
  * cache 池线程数支持 0-Integer.MAX\_VALUE\(显然完全没考虑主机的资源承受能力），60 秒 IDLE 。

**newScheduledThreadPool\(int\)**

* 调度型线程池
* 这个池子里的线程可以按 schedule 依次 delay 执行，或周期执行

**SingleThreadExecutor\(\)**

* 单例线程，任意时间池中只能有一个线程
* 用的是和 cache 池和 fixed 池相同的底层池，但线程数目是 1-1,0 秒 IDLE（无 IDLE）

一般来说，CachedTheadPool 在程序执行过程中通常会创建与所需数量相同的线程，然后在它回收旧线程时停止创建新线程，因此它是合理的 Executor 的首选，只有当这种方式会引发问题时（比如需要大量长时间面向连接的线程时），才需要考虑用 FixedThreadPool。（该段话摘自《Thinking in Java》第四版）

**Executor 执行 Runnable 任务**

通过 Executors 的以上四个静态工厂方法获得 ExecutorService 实例，而后调用该实例的 execute（Runnable command）方法即可。一旦 Runnable 任务传递到 execute\(\)方法，该方法便会自动在一个线程上执行。下面是 Executor 执行 Runnable 任务的示例代码：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TestCachedThreadPool {
    public static void main(String[] args) {
        ExecutorService executorService = Executors.newCachedThreadPool();
        // ExecutorService executorService = Executors.newFixedThreadPool(5);
        // ExecutorService executorService = Executors.newSingleThreadExecutor();  
        for (int i = 0; i < 5; i++) {
            executorService.execute(new TestRunnable());
            System.out.println("************* a" + i + " *************");
        }
        executorService.shutdown();
    }
}

class TestRunnable implements Runnable {
    public void run() {
        System.out.println(Thread.currentThread().getName() + "线程被调用了。");
    }
}
```

执行后的结果如下：

![](http://wiki.jikexueyuan.com/project/java-concurrency/images/executor.jpg)

从结果中可以看出，pool-1-thread-1 和 pool-1-thread-2 均被调用了两次，这是随机的，execute 会首先在线程池中选择一个已有空闲线程来执行任务，如果线程池中没有空闲线程，它便会创建一个新的线程来执行任务。

**Executor 执行 Callable 任务**

在 Java 5 之后，任务分两类：一类是实现了 Runnable 接口的类，一类是实现了 Callable 接口的类。两者都可以被 ExecutorService 执行，但是 Runnable 任务没有返回值，而 Callable 任务有返回值。并且 Callable 的 call\(\)方法只能通过 ExecutorService 的 submit\(Callabletask\) 方法来执行，并且返回一个Future，是表示任务等待完成的 Future。

Callable 接口类似于 Runnable，两者都是为那些其实例可能被另一个线程执行的类设计的。但是 Runnable 不会返回结果，并且无法抛出经过检查的异常而 Callable 又返回结果，而且当获取返回结果时可能会抛出异常。Callable 中的 call\(\)方法类似 Runnable 的 run\(\)方法，区别同样是有返回值，后者没有。

当将一个 Callable 的对象传递给 ExecutorService 的 submit 方法，则该 call 方法自动在一个线程上执行，并且会返回执行结果 Future 对象。同样，将 Runnable 的对象传递给 ExecutorService 的 submit 方法，则该 run 方法自动在一个线程上执行，并且会返回执行结果 Future 对象，但是在该 Future 对象上调用 get 方法，将返回 null。

下面给出一个 Executor 执行 Callable 任务的示例代码：

```
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.*;

public class CallableDemo{
    public static void main(String[] args){
        ExecutorService executorService = Executors.newCachedThreadPool();
        List<Future<String>> resultList = new ArrayList<Future<String>>();

        //创建10个任务并执行   
        for (int i = 0; i < 10; i++){
            //使用ExecutorService执行Callable类型的任务，并将结果保存在future变量中   
            Future<String> future = executorService.submit(new TaskWithResult(i));
            //将任务执行结果存储到List中   
            resultList.add(future);
        }

        //遍历任务的结果   
        for (Future<String> fs : resultList){
            try{
                while(!fs.isDone);//Future返回如果没有完成，则一直循环等待，直到Future返回完成  
                System.out.println(fs.get());     //打印各个线程（任务）执行的结果   
            }catch(InterruptedException e){
                e.printStackTrace();
            }catch(ExecutionException e){
                e.printStackTrace();
            }finally{
                //启动一次顺序关闭，执行以前提交的任务，但不接受新任务  
                executorService.shutdown();
            }
        }
    }
}

class TaskWithResult implements Callable<String>{
    private int id;

    public TaskWithResult(int id){
        this.id = id;
    }

    /**
     * 任务的具体过程，一旦任务传给ExecutorService的submit方法， 
     * 则该方法自动在一个线程上执行 
     */
    public String call() throws Exception {
        System.out.println("call()方法被自动调用！！！    " + Thread.currentThread().getName());
        //该返回结果将被Future的get方法得到  
        return "call()方法被自动调用，任务返回的结果是：" + id + "    " + Thread.currentThread().getName();
    }
}
```

执行结果如下：

![](http://wiki.jikexueyuan.com/project/java-concurrency/images/executor1.jpg)

从结果中可以同样可以看出，submit 也是首先选择空闲线程来执行任务，如果没有，才会创建新的线程来执行任务。另外，需要注意：如果 Future 的返回尚未完成，则 get\(\)方法会阻塞等待，直到 Future 完成返回，可以通过调用 isDone\(\)方法判断 Future 是否完成了返回。

我们大致来看下 Executors 的源码，newCachedThreadPool 的不带 RejectedExecutionHandler 参数（即第五个参数，线程数量超过 maximumPoolSize 时，指定处理方式）的构造方法如下：

```
public static ExecutorService newCachedThreadPool() {
        return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
        60L, TimeUnit.SECONDS,
        new SynchronousQueue<Runnable>());
        }
```

它将 corePoolSize 设定为 0，而将 maximumPoolSize 设定为了 Integer 的最大值，线程空闲超过 60 秒，将会从线程池中移除。由于核心线程数为 0，因此每次添加任务，都会先从线程池中找空闲线程，如果没有就会创建一个线程（SynchronousQueue决定的，后面会说）来执行新的任务，并将该线程加入到线程池中，而最大允许的线程数为 Integer 的最大值，因此这个线程池理论上可以不断扩大。

再来看 newFixedThreadPool 的不带 RejectedExecutionHandler 参数的构造方法，如下：

```
public static ExecutorService newFixedThreadPool(int nThreads) {  
    return new ThreadPoolExecutor(nThreads, nThreads,  
                                  0L, TimeUnit.MILLISECONDS,  
                                  new LinkedBlockingQueue<Runnable>());  
}
```

它将 corePoolSize 和 maximumPoolSize 都设定为了 nThreads，这样便实现了线程池的大小的固定，不会动态地扩大，另外，keepAliveTime 设定为了 0，也就是说线程只要空闲下来，就会被移除线程池。

## 



