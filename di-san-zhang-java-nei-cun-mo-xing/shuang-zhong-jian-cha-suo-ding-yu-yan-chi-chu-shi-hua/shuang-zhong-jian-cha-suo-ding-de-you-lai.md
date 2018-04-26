# 双重检查锁定的由来

在Java程序中，有时候可能需要推迟一些高开销的对象初始化操作，并且只有在使用这些对象时才进行初始化。此时，程序员可能会采用延迟初始化。但要正确实现线程安全的延迟初

始化需要一些技巧，否则很容易出现问题。比如，下面是非线程安全的延迟初始化对象的示例代码。

```
public class UnsafeLazyInitialization {

	private static Instance instance;

	public static Instance getInstance() {

		if (instance == null) // 1：A线程执行

		instance = new Instance(); // 2：B线程执行

		return instance;

	}

}
```

在UnsafeLazyInitialization类中，假设A线程执行代码1的同时，B线程执行代码2。此时，线程A可能会看到instance引用的对象还没有完成初始化。对于UnsafeLazyInitialization类，我们可以对getInstance\(\)方法做同步处理来实现线程安全的延迟初始化。示例代码如下。

```
public class SafeLazyInitialization {

	private static Instance instance;

	public synchronized static Instance getInstance() {

	if (instance == null)

	instance = new Instance();

	return instance;

	}

}
```

由于对getInstance\(\)方法做了同步处理，synchronized将导致性能开销。如果getInstance\(\)方法被多个线程频繁的调用，将会导致程序执行性能的下降。反之，如果getInstance\(\)方法不会被多个线程频繁的调用，那么这个延迟初始化方案将能提供令人满意的性能。

在早期的JVM中，synchronized（甚至是无竞争的synchronized）存在巨大的性能开销。因此，人们想出了一个“聪明”的技巧：双重检查锁定（Double-Checked Locking）。人们想通过双重检查锁定来降低同步的开销。下面是使用双重检查锁定来实现延迟初始化的示例代码。

```
public class DoubleCheckedLocking { // 1

	private static Instance instance; // 2

	public static Instance getInstance() { // 3

		if (instance == null) { // 4:第一次检查

		synchronized (DoubleCheckedLocking.class) { // 5:加锁

		if (instance == null) // 6:第二次检查

		instance = new Instance(); // 7:问题的根源出在这里

		} // 8

		} // 9

		return instance; // 10

	} // 11

}
```

如上面代码所示，如果第一次检查instance不为null，那么就不需要执行下面的加锁和初始化操作。因此，可以大幅降低synchronized带来的性能开销。上面代码表面上看起来，似乎两全其美。

* 多个线程试图在同一时间创建对象时，会通过加锁来保证只有一个线程能创建对象。
* 在对象创建好之后，执行getInstance\(\)方法将不需要获取锁，直接返回已创建好的对象。
* 双重检查锁定看起来似乎很完美，但这是一个错误的优化！在线程执行到第4行，代码读取到instance不为null时，instance引用的对象有可能还没有完成初始化。



