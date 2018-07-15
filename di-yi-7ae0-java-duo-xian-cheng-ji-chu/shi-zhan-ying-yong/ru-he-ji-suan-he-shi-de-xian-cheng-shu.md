# 如何计算合适的线程数

**如何合理地估算线程池大小？**

这个问题虽然看起来很小，却并不那么容易回答。大家如果有更好的方法欢迎赐教，先来一个天真的估算方法：假设要求一个系统的TPS（Transaction Per Second或者Task Per Second）至少为20，然后假设每个Transaction由一个线程完成，继续假设平均每个线程处理一个Transaction的时间为4s。那么问题转化为：

**如何设计线程池大小，使得可以在1s内处理完20个Transaction？**

计算过程很简单，每个线程的处理能力为0.25TPS，那么要达到20TPS，显然需要20/0.25=80个线程。

很显然这个估算方法很天真，因为它没有考虑到CPU数目。一般服务器的CPU核数为16或者32，如果有80个线程，那么肯定会带来太多不必要的线程上下文切换开销。  
  
再来第二种简单的但不知是否可行的方法（N为CPU总核数）：

* 如果是CPU密集型应用，则线程池大小设置为N+1
* 如果是IO密集型应用，则线程池大小设置为2N+1

如果一台服务器上只部署这一个应用并且只有这一个线程池，那么这种估算或许合理，具体还需自行测试验证。

接下来在这个文档：服务器性能IO优化 中发现一个估算公式：

> 最佳线程数目 = （（线程等待时间+线程CPU时间）/线程CPU时间 ）\* CPU数目

比如平均每个线程CPU运行时间为0.5s，而线程等待时间（非CPU运行时间，比如IO）为1.5s，CPU核心数为8，那么根据上面这个公式估算得到：\(\(0.5+1.5\)/0.5\)\*8=32。这个公式进一步转化为：

> 最佳线程数目 = （线程等待时间与线程CPU时间之比 + 1）\* CPU数目



可以得出一个结论：

_**线程等待时间所占比例越高，需要越多线程。线程CPU时间所占比例越高，需要越少线程。**_

上一种估算方法也和这个结论相合。

一个系统最快的部分是CPU，所以决定一个系统吞吐量上限的是CPU。增强CPU处理能力，可以提高系统吞吐量上限。但根据短板效应，真实的系统吞吐量并不能单纯根据CPU来计算。那要提高系统吞吐量，就需要从“系统短板”（比如网络延迟、IO）着手：

* 尽量提高短板操作的并行化比率，比如多线程下载技术
* 增强短板能力，比如用NIO替代IO

第一条可以联系到Amdahl定律，这条定律定义了串行系统并行化后的加速比计算公式：

> 加速比=优化前系统耗时 / 优化后系统耗时



加速比越大，表明系统并行化的优化效果越好。Addahl定律还给出了系统并行度、CPU数目和加速比的关系，加速比为Speedup，系统串行化比率（指串行执行代码所占比率）为F，CPU数目为N：

> Speedup &lt;= 1 / \(F + \(1-F\)/N\)



当N足够大时，串行化比率F越小，加速比Speedup越大。

写到这里，我突然冒出一个问题。

**是否使用线程池就一定比使用单线程高效呢？**

答案是否定的，比如Redis就是单线程的，但它却非常高效，基本操作都能达到十万量级/s。从线程这个角度来看，部分原因在于：

* 多线程带来线程上下文切换开销，单线程就没有这种开销
* 锁

当然“Redis很快”更本质的原因在于：Redis基本都是内存操作，这种情况下单线程可以很高效地利用CPU。而多线程适用场景一般是：存在相当比例的IO和网络操作。

所以即使有上面的简单估算方法，也许看似合理，但实际上也未必合理，都需要结合系统真实情况（比如是IO密集型或者是CPU密集型或者是纯内存操作）和硬件环境（CPU、内存、硬盘读写速度、网络状况等）来不断尝试达到一个符合实际的合理估算值。



最后来一个“Dark Magic”估算方法（因为我暂时还没有搞懂它的原理），使用下面的类：

```
package pool_size_calculate;

import java.math.BigDecimal;
import java.math.RoundingMode;
import java.util.Timer;
import java.util.TimerTask;
import java.util.concurrent.BlockingQueue;

/**
 * A class that calculates the optimal thread pool boundaries. It takes the
 * desired target utilization and the desired work queue memory consumption as
 * input and retuns thread count and work queue capacity.
 *
 * @author Niklas Schlimm
 *
 */
public abstract class PoolSizeCalculator {

	/**
	 * The sample queue size to calculate the size of a single {@link Runnable}
	 * element.
	 */
	private final int SAMPLE_QUEUE_SIZE = 1000;

	/**
	 * Accuracy of test run. It must finish within 20ms of the testTime
	 * otherwise we retry the test. This could be configurable.
	 */
	private final int EPSYLON = 20;

	/**
	 * Control variable for the CPU time investigation.
	 */
	private volatile boolean expired;

	/**
	 * Time (millis) of the test run in the CPU time calculation.
	 */
	private final long testtime = 3000;

	/**
	 * Calculates the boundaries of a thread pool for a given {@link Runnable}.
	 *
	 * @param targetUtilization
	 *            the desired utilization of the CPUs (0 <= targetUtilization <= 	 *            1) 	 * @param targetQueueSizeBytes 	 *            the desired maximum work queue size of the thread pool (bytes) 	 */ 	protected void calculateBoundaries(BigDecimal targetUtilization, 			BigDecimal targetQueueSizeBytes) { 		calculateOptimalCapacity(targetQueueSizeBytes); 		Runnable task = creatTask(); 		start(task); 		start(task); // warm up phase 		long cputime = getCurrentThreadCPUTime(); 		start(task); // test intervall 		cputime = getCurrentThreadCPUTime() - cputime; 		long waittime = (testtime * 1000000) - cputime; 		calculateOptimalThreadCount(cputime, waittime, targetUtilization); 	} 	private void calculateOptimalCapacity(BigDecimal targetQueueSizeBytes) { 		long mem = calculateMemoryUsage(); 		BigDecimal queueCapacity = targetQueueSizeBytes.divide(new BigDecimal( 				mem), RoundingMode.HALF_UP); 		System.out.println("Target queue memory usage (bytes): " 				+ targetQueueSizeBytes); 		System.out.println("createTask() produced " 				+ creatTask().getClass().getName() + " which took " + mem 				+ " bytes in a queue"); 		System.out.println("Formula: " + targetQueueSizeBytes + " / " + mem); 		System.out.println("* Recommended queue capacity (bytes): " 				+ queueCapacity); 	} 	/** 	 * Brian Goetz' optimal thread count formula, see 'Java Concurrency in 	 * Practice' (chapter 8.2) 	 *  	 * @param cpu 	 *            cpu time consumed by considered task 	 * @param wait 	 *            wait time of considered task 	 * @param targetUtilization 	 *            target utilization of the system 	 */ 	private void calculateOptimalThreadCount(long cpu, long wait, 			BigDecimal targetUtilization) { 		BigDecimal waitTime = new BigDecimal(wait); 		BigDecimal computeTime = new BigDecimal(cpu); 		BigDecimal numberOfCPU = new BigDecimal(Runtime.getRuntime() 				.availableProcessors()); 		BigDecimal optimalthreadcount = numberOfCPU.multiply(targetUtilization) 				.multiply( 						new BigDecimal(1).add(waitTime.divide(computeTime, 								RoundingMode.HALF_UP))); 		System.out.println("Number of CPU: " + numberOfCPU); 		System.out.println("Target utilization: " + targetUtilization); 		System.out.println("Elapsed time (nanos): " + (testtime * 1000000)); 		System.out.println("Compute time (nanos): " + cpu); 		System.out.println("Wait time (nanos): " + wait); 		System.out.println("Formula: " + numberOfCPU + " * " 				+ targetUtilization + " * (1 + " + waitTime + " / " 				+ computeTime + ")"); 		System.out.println("* Optimal thread count: " + optimalthreadcount); 	} 	/** 	 * Runs the {@link Runnable} over a period defined in {@link #testtime}. 	 * Based on Heinz Kabbutz' ideas 	 * (http://www.javaspecialists.eu/archive/Issue124.html). 	 *  	 * @param task 	 *            the runnable under investigation 	 */ 	public void start(Runnable task) { 		long start = 0; 		int runs = 0; 		do { 			if (++runs > 5) {
				throw new IllegalStateException("Test not accurate");
			}
			expired = false;
			start = System.currentTimeMillis();
			Timer timer = new Timer();
			timer.schedule(new TimerTask() {
				public void run() {
					expired = true;
				}
			}, testtime);
			while (!expired) {
				task.run();
			}
			start = System.currentTimeMillis() - start;
			timer.cancel();
		} while (Math.abs(start - testtime) > EPSYLON);
		collectGarbage(3);
	}

	private void collectGarbage(int times) {
		for (int i = 0; i < times; i++) {
			System.gc();
			try {
				Thread.sleep(10);
			} catch (InterruptedException e) {
				Thread.currentThread().interrupt();
				break;
			}
		}
	}

	/**
	 * Calculates the memory usage of a single element in a work queue. Based on
	 * Heinz Kabbutz' ideas
	 * (http://www.javaspecialists.eu/archive/Issue029.html).
	 *
	 * @return memory usage of a single {@link Runnable} element in the thread
	 *         pools work queue
	 */
	public long calculateMemoryUsage() {
		BlockingQueue queue = createWorkQueue();
		for (int i = 0; i < SAMPLE_QUEUE_SIZE; i++) {
			queue.add(creatTask());
		}
		long mem0 = Runtime.getRuntime().totalMemory()
				- Runtime.getRuntime().freeMemory();
		long mem1 = Runtime.getRuntime().totalMemory()
				- Runtime.getRuntime().freeMemory();
		queue = null;
		collectGarbage(15);
		mem0 = Runtime.getRuntime().totalMemory()
				- Runtime.getRuntime().freeMemory();
		queue = createWorkQueue();
		for (int i = 0; i < SAMPLE_QUEUE_SIZE; i++) {
			queue.add(creatTask());
		}
		collectGarbage(15);
		mem1 = Runtime.getRuntime().totalMemory()
				- Runtime.getRuntime().freeMemory();
		return (mem1 - mem0) / SAMPLE_QUEUE_SIZE;
	}

	/**
	 * Create your runnable task here.
	 *
	 * @return an instance of your runnable task under investigation
	 */
	protected abstract Runnable creatTask();

	/**
	 * Return an instance of the queue used in the thread pool.
	 *
	 * @return queue instance
	 */
	protected abstract BlockingQueue createWorkQueue();

	/**
	 * Calculate current cpu time. Various frameworks may be used here,
	 * depending on the operating system in use. (e.g.
	 * http://www.hyperic.com/products/sigar). The more accurate the CPU time
	 * measurement, the more accurate the results for thread count boundaries.
	 *
	 * @return current cpu time of current thread
	 */
	protected abstract long getCurrentThreadCPUTime();

}
```



然后自己继承这个抽象类并实现它的三个抽象方法，比如下面是我写的一个示例（任务是请求网络数据），其中我指定期望CPU利用率为1.0（即100%），任务队列总大小不超过100,000字节：

```
package pool_size_calculate;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.lang.management.ManagementFactory;
import java.math.BigDecimal;
import java.net.HttpURLConnection;
import java.net.URL;
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

public class SimplePoolSizeCaculatorImpl extends PoolSizeCalculator {

	@Override
	protected Runnable creatTask() {
		return new AsyncIOTask();
	}

	@Override
	protected BlockingQueue createWorkQueue() {
		return new LinkedBlockingQueue(1000);
	}

	@Override
	protected long getCurrentThreadCPUTime() {
		return ManagementFactory.getThreadMXBean().getCurrentThreadCpuTime();
	}

	public static void main(String[] args) {
		PoolSizeCalculator poolSizeCalculator = new SimplePoolSizeCaculatorImpl();
		poolSizeCalculator.calculateBoundaries(new BigDecimal(1.0), new BigDecimal(100000));
	}

}

/**
 * 自定义的异步IO任务
 * @author Will
 *
 */
class AsyncIOTask implements Runnable {

	@Override
	public void run() {
		HttpURLConnection connection = null;
		BufferedReader reader = null;
		try {
			String getURL = "http://baidu.com";
			URL getUrl = new URL(getURL);

			connection = (HttpURLConnection) getUrl.openConnection();
			connection.connect();
			reader = new BufferedReader(new InputStreamReader(
					connection.getInputStream()));

			String line;
			while ((line = reader.readLine()) != null) {
				// empty loop
			}
		}

		catch (IOException e) {

		} finally {
			if(reader != null) {
				try {
					reader.close();
				}
				catch(Exception e) {

				}
			}
			connection.disconnect();
		}

	}

}
```

得到的输出如下：

```
Target queue memory usage (bytes): 100000
createTask() produced pool_size_calculate.AsyncIOTask which took 40 bytes in a queue
Formula: 100000 / 40
* Recommended queue capacity (bytes): 2500
Number of CPU: 4
Target utilization: 1
Elapsed time (nanos): 3000000000
Compute time (nanos): 47181000
Wait time (nanos): 2952819000
Formula: 4 * 1 * (1 + 2952819000 / 47181000)
* Optimal thread count: 256
```

推荐的任务队列大小为2500，线程数为256，有点出乎意料之外。我可以如下构造一个线程池：

```
ThreadPoolExecutor pool = new ThreadPoolExecutor(256, 256, 0L, TimeUnit.MILLISECONDS, new LinkedBlockingQueue(2500));
```



