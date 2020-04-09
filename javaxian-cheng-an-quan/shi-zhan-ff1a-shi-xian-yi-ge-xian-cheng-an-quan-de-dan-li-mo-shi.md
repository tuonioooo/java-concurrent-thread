# 实战：实现一个线程安全的单例模式

在GoF的23种设计模式中，单例模式是比较简单的一种。然而，有时候越是简单的东西越容易出现问题。下面就单例设计模式详细的探讨一下。

所谓单例模式，简单来说，就是在整个应用中保证只有一个类的实例存在。就像是Java Web中的application，也就是提供了一个全局变量，用处相当广泛，比如保存全局数据，实现全局性的操作等。

1. 最简单的实现

首先，能够想到的最简单的实现是，把类的构造函数写成private的，从而保证别的类不能实例化此类，然后在类中提供一个静态的实例并能够返回给使用者。这样，使用者就可以通过这个引用使用到这个类的实例了。

```text
public class SingletonClass { 

  private static final SingletonClass instance = new SingletonClass(); 

  public static SingletonClass getInstance() { 
    return instance; 
  } 

  private SingletonClass() { 

  } 
}
```

如上例，外部使用者如果需要使用SingletonClass的实例，只能通过getInstance\(\)方法，并且它的构造方法是private的，这样就保证了只能有一个对象存在。

1. 性能优化——lazy loaded

上面的代码虽然简单，但是有一个问题——无论这个类是否被使用，都会创建一个instance对象。如果这个创建过程很耗时，比如需要连接10000次数据库\(夸张了…:-\)\)，并且这个类还并不一定会被使用，那么这个创建过程就是无用的。怎么办呢？

为了解决这个问题，我们想到了新的解决方案：

```text
public class SingletonClass { 

  private static SingletonClass instance = null; 

  public static SingletonClass getInstance() { 
    if(instance == null) { 
      instance = new SingletonClass(); 
    } 
    return instance; 
  } 

  private SingletonClass() { 

  } 

}
```

代码的变化有两处——首先，把instance初始化为null，直到第一次使用的时候通过判断是否为null来创建对象。因为创建过程不在声明处，所以那个final的修饰必须去掉。

我们来想象一下这个过程。要使用SingletonClass，调用getInstance\(\)方法。第一次的时候发现instance是null，然后就新建一个对象，返回出去；第二次再使用的时候，因为这个instance是static的，所以已经不是null了，因此不会再创建对象，直接将其返回。

这个过程就成为lazy loaded，也就是迟加载——直到使用的时候才进行加载。

1. 同步

上面的代码很清楚，也很简单。然而就像那句名言：“80%的错误都是由20%代码优化引起的”。单线程下，这段代码没有什么问题，可是如果是多线程，麻烦就来了。我们来分析一下：

线程A希望使用SingletonClass，调用getInstance\(\)方法。因为是第一次调用，A就发现instance是null的，于是它开始创建实例，就在这个时候，CPU发生时间片切换，线程B开始执行，它要使用SingletonClass，调用getInstance\(\)方法，同样检测到instance是null——注意，这是在A检测完之后切换的，也就是说A并没有来得及创建对象——因此B开始创建。B创建完成后，切换到A继续执行，因为它已经检测完了，所以A不会再检测一遍，它会直接创建对象。这样，线程A和B各自拥有一个SingletonClass的对象——单例失败！

解决的方法也很简单，那就是加锁：

```text
public class SingletonClass { 

  private static SingletonClass instance = null; 

  public synchronized static SingletonClass getInstance() { 
    if(instance == null) { 
      instance = new SingletonClass(); 
    } 
    return instance; 
  } 

  private SingletonClass() { 

  } 
}
```

是要getInstance\(\)加上同步锁，一个线程必须等待另外一个线程创建完成后才能使用这个方法，这就保证了单例的唯一性。

1. 又是性能

上面的代码又是很清楚很简单的，然而，简单的东西往往不够理想。这段代码毫无疑问存在性能的问题——synchronized修饰的同步块可是要比一般的代码段慢上几倍的！如果存在很多次getInstance\(\)的调用，那性能问题就不得不考虑了！

让我们来分析一下，究竟是整个方法都必须加锁，还是仅仅其中某一句加锁就足够了？我们为什么要加锁呢？分析一下出现lazy loaded的那种情形的原因。原因就是检测null的操作和创建对象的操作分离了。如果这两个操作能够原子地进行，那么单例就已经保证了。于是，我们开始修改代码：

```text
public class SingletonClass { 

  private static SingletonClass instance = null; 

  public static SingletonClass getInstance() { 
    synchronized (SingletonClass.class) { 
      if(instance == null) { 
        instance = new SingletonClass(); 
      } 
    }     
    return instance; 
  } 

  private SingletonClass() { 

  } 

}
```

首先去掉getInstance\(\)的同步操作，然后把同步锁加载if语句上。但是这样的修改起不到任何作用：因为每次调用getInstance\(\)的时候必然要同步，性能问题还是存在。如果……如果我们事先判断一下是不是为null再去同步呢？

```text
public class SingletonClass { 

  private static SingletonClass instance = null; 

  public static SingletonClass getInstance() { 
    if (instance == null) { 
      synchronized (SingletonClass.class) { 
        if (instance == null) { 
          instance = new SingletonClass(); 
        } 
      } 
    } 
    return instance; 
  } 

  private SingletonClass() { 

  } 
}
```

还有问题吗？首先判断instance是不是为null，如果为null，加锁初始化；如果不为null，直接返回instance。

这就是double-checked locking设计实现单例模式。到此为止，一切都很完美。我们用一种很聪明的方式实现了单例模式。

1. 从源头检查

下面我们开始说编译原理。所谓编译，就是把源代码“翻译”成目标代码——大多数是指机器代码——的过程。针对Java，它的目标代码不是本地机器代码，而是虚拟机代码。编译原理里面有一个很重要的内容是编译器优化。所谓编译器优化是指，在不改变原来语义的情况下，通过调整语句顺序，来让程序运行的更快。这个过程成为reorder。

要知道，JVM只是一个标准，并不是实现。JVM中并没有规定有关编译器优化的内容，也就是说，JVM实现可以自由的进行编译器优化。

下面来想一下，创建一个变量需要哪些步骤呢？一个是申请一块内存，调用构造方法进行初始化操作，另一个是分配一个指针指向这块内存。这两个操作谁在前谁在后呢？JVM规范并没有规定。那么就存在这么一种情况，JVM是先开辟出一块内存，然后把指针指向这块内存，最后调用构造方法进行初始化。

下面我们来考虑这么一种情况：线程A开始创建SingletonClass的实例，此时线程B调用了getInstance\(\)方法，首先判断instance是否为null。按照我们上面所说的内存模型，A已经把instance指向了那块内存，只是还没有调用构造方法，因此B检测到instance不为null，于是直接把instance返回了——问题出现了，尽管instance不为null，但它并没有构造完成，就像一套房子已经给了你钥匙，但你并不能住进去，因为里面还没有收拾。此时，如果B在A将instance构造完成之前就是用了这个实例，程序就会出现错误了！

于是，我们想到了下面的代码：

```text
public class SingletonClass { 

  private static SingletonClass instance = null; 

  public static SingletonClass getInstance() { 
    if (instance == null) { 
      SingletonClass sc; 
      synchronized (SingletonClass.class) { 
        sc = instance; 
        if (sc == null) { 
          synchronized (SingletonClass.class) { 
            if(sc == null) { 
              sc = new SingletonClass(); 
            } 
          } 
          instance = sc; 
        } 
      } 
    } 
    return instance; 
  } 

  private SingletonClass() { 

  } 

}
```

我们在第一个同步块里面创建一个临时变量，然后使用这个临时变量进行对象的创建，并且在最后把instance指针临时变量的内存空间。写出这种代码基于以下思想，即synchronized会起到一个代码屏蔽的作用，同步块里面的代码和外部的代码没有联系。因此，在外部的同步块里面对临时变量sc进行操作并不影响instance，所以外部类在instance=sc;之前检测instance的时候，结果instance依然是null。

不过，这种想法完全是

错误

的！同步块的释放保证在此之前——也就是同步块里面——的操作必须完成，但是并不保证同步块之后的操作不能因编译器优化而调换到同步块结束之前进行。因此，编译器完全可以把instance=sc;这句移到内部同步块里面执行。这样，程序又是错误的了！

1. 解决方案

说了这么多，难道单例没有办法在Java中实现吗？其实不然！

在JDK 5之后，Java使用了新的内存模型。volatile关键字有了明确的语义——在JDK1.5之前，volatile是个关键字，但是并没有明确的规定其用途——被volatile修饰的写变量不能和之前的读写代码调整，读变量不能和之后的读写代码调整！因此，只要我们简单的把instance加上volatile关键字就可以了。

```text
public class SingletonClass { 

  private volatile static SingletonClass instance = null; 

  public static SingletonClass getInstance() { 
    if (instance == null) { 
      synchronized (SingletonClass.class) { 
        if(instance == null) { 
          instance = new SingletonClass(); 
        } 
      } 
    } 
    return instance; 
  } 

  private SingletonClass() { 

  } 
}
```

然而，这只是JDK1.5之后的Java的解决方案，那之前版本呢？其实，还有另外的一种解决方案，并不会受到Java版本的影响：

```text
public class SingletonClass { 

  private static class SingletonClassInstance { 
    private static final SingletonClass instance = new SingletonClass(); 
  } 

  public static SingletonClass getInstance() { 
    return SingletonClassInstance.instance; 
  } 

  private SingletonClass() { 

  } 

}
```

在这一版本的单例模式实现代码中，我们使用了Java的静态内部类。这一技术是被JVM明确说明了的，因此不存在任何二义性。在这段代码中，因为SingletonClass没有static的属性，因此并不会被初始化。直到调用getInstance\(\)的时候，会首先加载SingletonClassInstance类，这个类有一个static的SingletonClass实例，因此需要调用SingletonClass的构造方法，然后getInstance\(\)将把这个内部类的instance返回给使用者。由于这个instance是static的，因此并不会构造多次。

由于SingletonClassInstance是私有静态内部类，所以不会被其他类知道，同样，static语义也要求不会有多个实例存在。并且，JSL规范定义，类的构造必须是原子性的，非并发的，因此不需要加同步块。同样，由于这个构造是并发的，所以getInstance\(\)也并不需要加同步。

至此，我们完整的了解了单例模式在Java语言中的时候，提出了两种解决方案。个人偏向于第二种，并且Effiective Java也推荐的这种方式。

