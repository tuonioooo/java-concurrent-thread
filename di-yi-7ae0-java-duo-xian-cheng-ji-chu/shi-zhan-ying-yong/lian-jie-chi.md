## 连接池

连接池是创建和管理一个连接的缓冲池的技术，这些连接准备好被任何需要它们的[线程](https://baike.baidu.com/item/线程/103101)使用。

## **优点**

这种连接“汇集”起来的技术基于这样的一个事实：对于大多数应用程序，当它们正在处理通常需要数毫秒完成的[事务](https://baike.baidu.com/item/事务)时，仅需要能够访问[JDBC](https://baike.baidu.com/item/JDBC)连接的 1 个线程。当不处理事务时，这个连接就会闲置。相反，连接池允许闲置的连接被其它需要的线程使用。

事实上，当一个线程需要用 JDBC 对一个 GBase 或其它数据库操作时，它从池中请求一个连接。当这个线程使用完了这个连接，将它返回到连接池中，这样这就可以被其它想使用它的线程使用。

当连接从池中“借出”，它被请求它的[线程](https://baike.baidu.com/item/线程)专有地使用。从编程的角度来看，这和用户的线程每当需要一个 JDBC 连接的时候调用DriverManager.getConnection\(\) 是一样的，采用[连接池技术](https://baike.baidu.com/item/连接池技术)，可通过使用新的或已有的连接结束线程。

连接池可以极大的改善用户的 Java 应用程序的性能，同时减少全部资源的使用。连接池主要的优点有：

**减少连接创建时间**

虽然与其它数据库相比 GBase 提供了较为快速连接功能，但是创建新的 JDBC 连接仍会招致网络和 JDBC 驱动的开销。如果这类连接是“循环”使用的，使用该方式这些花销就可避免。

**简化的编程模式**

当使用连接池时，每一个单独的线程能够像创建了一个自己的 JDBC 连接一样操作，允许用户直接使用JDBC[编程技术](https://baike.baidu.com/item/编程技术)。

**受控的资源使用**

如果用户不使用连接池，而是每当[线程](https://baike.baidu.com/item/线程)需要时创建一个新的连接，那么用户的应用程序的资源使用会产生非常大的浪费并且可能会导致高负载下的异常发生。注意，每个连到 GBase 的连接在客户端和服务器端都有花销（内存，CPU，[上下文切换](https://baike.baidu.com/item/上下文切换)等等）。每个连接均会对应用程序和 GBase 服务器的可用资源带来一定的限制。不管这些连接是否在做有用的工作，仍将使用这些资源中的相当一部分。

连接池能够使性能最大化，同时还能将资源利用控制在一定的水平之下，如果超过该水平，应用程序将崩溃而不仅仅是变慢。



## 运作原理

在实际应用开发中，特别是在WEB应用系统中，如果[JSP](https://baike.baidu.com/item/JSP)、[Servlet](https://baike.baidu.com/item/Servlet)或EJB使用[JDBC](https://baike.baidu.com/item/JDBC)

直接访问数据库中的数据，每一次数据访问请求都必须经历建立数据库连接、打开数据库、存取数据和关闭数据库连接等步骤，而连接并打开数据库是一件既消耗资源又费时的工作，如果频繁发生这种数据库操作，系统的性能必然会急剧下降，甚至会导致系统崩溃。

[数据库连接池](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0)

技术是解决这个问题最常用的方法，在许多应用程序服务器（例如：Weblogic,WebSphere,JBoss）中，基本都提供了这项技术，无需自己编程，但是，深入了解这项技术是非常必要的。

数据库连接池技术的思想非常简单，将数据库连接作为对象存储在一个Vector对象中，一旦数据库连接建立后，不同的数据库访问请求就可以共享这些连接，这样，通过复用这些已经建立的数据库连接，可以克服上述缺点，极大地节省系统资源和时间。

数据库连接池的主要操作如下：

（1）建立数据库连接池对象（服务器启动）。

（2）按照事先指定的参数创建初始数量的数据库连接（即：空闲连接数）。

（3）对于一个数据库访问请求，直接从连接池中得到一个连接。如果[数据库连接池](https://baike.baidu.com/item/%E6%95%B0%E6%8D%AE%E5%BA%93%E8%BF%9E%E6%8E%A5%E6%B1%A0)

对象中没有空闲的连接，且连接数没有达到最大（即：最大活跃连接数），创建一个新的数据库连接。

（4）存取数据库。

（5）关闭数据库，释放所有数据库连接（此时的关闭数据库连接，并非真正关闭，而是将其放入空闲队列中。如实际空闲连接数大于初始空闲连接数则释放连接）。

（6）释放数据库连接池对象（服务器停止、维护期间，释放数据库连接池对象，并释放所有连接）。

## 实现模式

**1、连接池模型**

本文讨论的连接池包括一个连接池类（DBConnectionPool）和一个连接池管理类（DBConnetionPoolManager）。连接池类是对某一数据库所有连接的“缓冲池”，主要实现以下功能：①从连接池获取或创建可用连接；②使用完毕之后，把连接返还给连接池；③在系统关闭前，断开所有连接并释放连接占用的系统资源；④还能够处理无效连接（原来登记为可用的连接，由于某种原因不再可用，如超时，通讯问题），并能够限制连接池中的连接总数不低于某个预定值和不超过某个预定值。

连接池管理类是连接池类的外覆类（wrapper），符合单例模式，即系统中只能有一个连接池管理类的实例。其主要用于对多个连接池对象的管理，具有以下功能：①装载并注册特定数据库的JDBC驱动程序；②根据属性文件给定的信息，创建连接池对象；③为方便管理多个连接池对象，为每一个连接池对象取一个名字，实现连接池名字与其实例之间的映射；④跟踪客户使用连接情况，以便需要时关闭连接释放资源。连接池管理类的引入主要是为了方便对多个连接池的使用和管理，如系统需要连接不同的数据库，或连接相同的数据库但由于安全性问题，需要不同的用户使用不同的名称和密码。

**2、连接池实现**

下面给出连接池类和连接池管理类的主要属性及所要实现的基本接口：

public class DBConnectionPool implements TimerListener{

private int checkedOut;//已被分配出去的连接数

private ArrayList freeConnections = new ArrayList\(\);//容器，空闲池，根据//创建时间顺序存放已创建但尚未分配出去的连接

private int minConn;//连接池里连接的最小数量

private int maxConn;//连接池里允许存在的最大连接数

private String name;//为这个连接池取个名字，方便管理

private String password;//连接数据库时需要的密码

private String url;//所要创建连接的数据库的地址

private String user;//连接数据库时需要的用户名

public Timer timer;//定时器

public DBConnectionPool\(String name, String URL, String user, String

password, int maxConn\)//公开的构造函数

public synchronized void freeConnection\(Connection con\) //使用完毕之后，//把连接返还给空闲池

public synchronized Connection getConnection\(long timeout\)//得到一个连接，//timeout是等待时间

public synchronized void release\(\)//断开所有连接，释放占用的系统资源

private Connection newConnection\(\)//新建一个数据库连接

public synchronized void TimerEvent\(\) //定时器事件处理函数

}

public class DBConnectionManager {

static private DBConnectionManager instance;//连接池管理类的唯一实例

static private int clients;//客户数量

private ArrayList drivers = new ArrayList\(\);//容器，存放数据库驱动程序

private HashMap pools = new HashMap \(\);//以name/value的形式存取连接池//对象的名字及连接池对象

static synchronized public DBConnectionManager getInstance\(\)//如果唯一的//实例instance已经创建，直接返回这个实例;否则，调用私有[构造函数](https://baike.baidu.com/item/%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)，创//建连接池管理类的唯一实例

private DBConnectionManager\(\)//私有构造函数,在其中调用初始化函数init\(\)

public void freeConnection\(String name, Connection con\)// 释放一个连接，//name是一个连接池对象的名字

public Connection getConnection\(Stringname\)//从名字为name的连接池对象//中得到一个连接

public Connection getConnection\(Stringname, long time\)//从名字为name

//的连接池对象中取得一个连接，time是等待时间

public synchronized void release\(\)//释放所有资源

private void createPools\(Properties props\)//根据属性文件提供的信息，创建//一个或多个连接池

private void init\(\)//初始化连接池管理类的唯一实例，由私有[构造函数](https://baike.baidu.com/item/%E6%9E%84%E9%80%A0%E5%87%BD%E6%95%B0)调用

private void loadDrivers\(Properties props\)//装载数据库驱动程序

}

**3、连接池使用**

上面所实现的连接池在程序开发时如何应用到系统中呢？下面以Servlet为例说明连接池的使用。

Servlet的生命周期是：在开始建立servlet时，调用其初始化（init）方法。之后每个用户请求都导致一个调用前面建立的实例的service方法的线程。最后，当服务器决定卸载一个servlet时，它首先调用该servlet的 destroy方法。

根据servlet的特点，我们可以在初始化函数中生成连接池管理类的唯一实例（其中包括创建一个或多个连接池）。如：

public void init\(\) throws ServletException

{

connMgr = DBConnectionManager.getInstance\(\);

}

然后就可以在service方法中通过连接池名称使用连接池，执行数据库操作。最后在destroy方法中释放占用的系统资源，如：

public void destroy\(\) {

connMgr.release\(\); super.destroy\(\);

}

