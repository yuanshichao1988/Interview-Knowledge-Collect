Java面试题整理

1、SpringMVC的执行流程

> 1. 用户发送请求至前端控制器DispatcherServlet 
> 2. DispatcherServlet收到请求调用HandlerMapping处理器映射器。 
> 3. 处理器映射器找到具体的处理器，生成处理器对象及处理器拦截器(如果有则生成)一并返回给DispatcherServlet。 
> 4. DispatcherServlet调用HandlerAdapter处理器适配器 
> 5. HandlerAdapter经过适配调用具体的处理器(Controller，也叫后端控制器)。 
> 6. Controller执行完成返回ModelAndView 
> 7. HandlerAdapter将controller执行结果ModelAndView返回给DispatcherServlet 
> 8. DispatcherServlet将ModelAndView传给ViewReslover视图解析器 
> 9. ViewReslover解析后返回具体View 
> 10. DispatcherServlet根据View进行渲染视图（即将模型数据填充至视图中）。 
> 11. DispatcherServlet响应用户

2、JVM内存溢出具体指哪些内存溢出？都会抛出什么异常？

> [内存溢出的解决思路](http://www.cnblogs.com/200911/p/3965108.html)

3、String类为什么是不可变类？

>  - 缓存池
>  - hashcode
>  - 多线程中不确定值

4、常用JVM设置参数有哪些？

> - **-Xms** 设置JVM最小可用内存
> - **-Xmx** 设置JVM最大可用内存
> - **-Xmn** 设置年轻代大小
> - **-Xss** 设置每个线程的堆栈大小（在相同物理内存下，减小这个值能生成更多的线程。但是也不能无限生成，最多3000~5000个）
> - **-XX:NewRatio** 设置年轻代(包括Eden和两个Survivor区)与年老代的比值。（设置为4,则年轻代与年老代所占比值为1:4，年轻代占整个堆栈的1/5）
> - **-XX:SurvivorRatio** 设置年轻代中Eden区与Survivor区的大小比值（设置为4，则两个Survivor区与一个Eden区的比值为2：4，一个Survivor区占整个年轻代的1/6）
> - **-XX:MaxPermSize** 设置持久代大小
> - **-XX:MaxTenuringThreshold** 设置垃圾最大年龄（如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代。对于年老代比较多的应用,可以提高效率.如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概论）

5、谈谈Spring事物传播特性

> - `REQUIRED` 如果存在一个事务，则支持当前事务。如果没有事务则开启一个新的事务
> - `MANDATORY` 支持当前事务，如果当前没有事务，就抛出异常
> - `NEVER `以非事务方式执行，如果当前存在事务，则抛出异常
> - `NOT_SUPPORTED` 以非事务方式执行操作，如果当前存在事务，就把当前事务挂起
> - `REQUIRES_NEW `新建事务，如果当前存在事务，把当前事务挂起
> - `SUPPORTS` 支持当前事务，如果当前没有事务，就以非事务方式执行
> - `NESTED` 支持当前事务，新增Save Point点，与当前事务同步提交或回滚。 
>   嵌套事务一个非常重要的概念就是内层事务依赖于外层事务。外层事务失败时，会回滚内层事务所做的动作。而内层事务操作失败并不会引起外层事务的回滚
> - `NESTED` 与 `REQUIRES_NEW`的区别 ?
>   - 它们非常 类似,都像一个嵌套事务，如果不存在一个活动的事务，都会开启一个新的事务。
>   - 使用`PROPAGATION_REQUIRES_NEW`时，内层事务与外层事务就像两个独立的事务一样，一旦内层事务进行了提交后，外层事务不能对其进行回滚。两个事务互不影响。两个事务不是一个真正的嵌套事务。同时它需要`JTA` 事务管理器的支持。 
>   - 使用`PROPAGATION_NESTED`时，外层事务的回滚可以引起内层事务的回滚。而内层事务的异常并不会导致外层事务的回滚，它是一个真正的嵌套事务。

6、MySql索引的类型和实现方式？

7、Tomcat请求的处理流程？

8、如何防止Sql注入？

- 最简单的在登陆的时候用户名填写：**`‘or 1 = 1 –`**
- 如何应对：
  - 参数化，禁止使用`sql`拼接
  - 对参数进行正则过滤
  - 过滤字符串中的关键字
  - 替换所有单引号
  - 可以在前端使用js进行数据过滤

9、谈谈HashMap、ConcurrnetHashMap

- HashMap

  - **非线程安全**，**无序**。在涉及到多线程并发的情况，进行get操作有可能会引起死循环，导致CPU利用率接近100%。
  - 存储`null` key和 `null` value
  - `fail-fast`机制，多线程修改会产生`ConcurrentModificationException`)
  - 解决方案有`Hashtable`和`Collections.synchronizedMap(hashMap)`，不过这两个方案基本上是对读写进行加锁操作，一个线程在读写元素，其余线程必须等待，性能可想而知。
  - 单向链表数组
  - put方法
  - resize()方法，创建一个新Node数组，将原数组的元素复制到新的数组中；
  - hash碰撞如何解决（循环链表，在next节点上插入）
  - 为了减少hash碰撞最好在初始化的时候预估数据的大小，在初始化的时候制定初始值
  - jdk1.8改成了如果链表长度大于8就使用红黑树存储

- ConcurrnetHashMap [占小狼的博客](https://www.jianshu.com/p/c0642afe03e0)

  - 不允许`null`的key和value

  - Jdk1.7

    - 采用分段锁的机制实现并发的更新操作，底层采用数组+链表的存储结构。
    - 两个核心静态内部类 Segment和HashEntry；
    - Segment继承ReentrantLock用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶；
    - HashEntry 用来封装映射表的键 / 值对；
    - 每个桶是由若干个 HashEntry 对象链接起来的链表。

  - jdk1.8

    - 抛弃了`Segment`分段锁机制。利用`CAS`+`Synchronized`来保证并发更新的安全，底层采用数组+链表+红黑树的存储结构；

    - **Node[] table**：默认为null，初始化发生在第一次插入操作（`initTable`方法），默认大小为`16`的数组，用来存储Node节点数据，扩容时大小总是2的幂次方

    - **nextTable**：默认为null，扩容时新生成的数组，其大小为原数组的两倍；

    - **sizeCtl** ：默认为0，用来控制table的初始化和扩容操作；

      - **-1 **代表table正在初始化
      - **-N **表示有N-1个线程正在进行扩容操作

    - **Node**：保存`key`，`value`及`key`的`hash`值的数据结构。其中`value`和`next`都用**`volatile`**修饰，保证并发的可见性。

      ```java
      class Node<K,V> implements Map.Entry<K,V> {
          final int hash;
          final K key;
          volatile V val;
          volatile Node<K,V> next;
          ......
      }
      ```

      **table初始化** 

      - 如何在并发执行initTable()方法的时候保证只执行一次的呢？
        - 通过`Unsafe.compareAndSwapInt`方法去修改sizeCtl的值为`-1`，有且只有一个线程能够修改成功，其它线程通过Thread.yield()让出CPU时间片等待table初始化完成。

      **`put`操作** 

      - 采用`CAS`+`synchronized`实现并发插入或更新操作
      - 根据`spread`方法计算hashcode值
      - 根据`(n - 1) & hash` 定位`table`数组索引位置
      - 获取`Unsafe.getObjectVolatile`方法获取`table`中对应的索引到元素**f**
      - 如果**f**为`null`，说明`table`中这个位置第一次插入元素，利用`Unsafe.compareAndSwapObject`方法插入Node节点。如果CAS成功，说明Node节点已经插入，随后`addCount(1L, binCount)`方法会检查当前容量是否需要进行扩容。如果CAS失败，说明有其它线程提前插入了节点，**`自旋重新尝试`**在这个位置插入节点。
      - 如果f的hash值为-1，说明当前f是ForwardingNode节点，意味有其它线程正在扩容，则一起进行扩容操作。
      - 其余情况把新的Node节点按链表或红黑树的方式插入到合适的位置，这个过程采用同步内置锁实现并发

10、JVM GC(minor gc，full gc)

> **JVM垃圾算法**：
>
> - [**引用计数法**](https://www.cnblogs.com/cielosun/p/6674431.html#11-%E5%BC%95%E7%94%A8%E8%AE%A1%E6%95%B0%E6%B3%95)（循环引用问题）
> - [**可达性分析**](https://www.cnblogs.com/cielosun/p/6674431.html#12-%E5%8F%AF%E8%BE%BE%E6%80%A7%E5%88%86%E6%9E%90)（思路就是通过一系列名为GC Roots**的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的）。Java中可以被作为GC Roots中的对象有：
>   - 虚拟机栈(栈桢中的本地变量表)中的引用的对象
>   - 方法区中的类静态属性引用的对象
>   - 方法区中的常量引用的对象
>   - 本地方法栈中JNI（Native方法）的引用的对象
>
> **垃圾回收算法**：
>
> - [标记-清除算法(Mark-Sweep)](https://www.cnblogs.com/cielosun/p/6674431.html#21-%E6%A0%87%E8%AE%B0-%E6%B8%85%E9%99%A4%E7%AE%97%E6%B3%95mark-sweep) 【会产生大量碎片】
> - [复制算法(Copying)](https://www.cnblogs.com/cielosun/p/6674431.html#22-%E5%A4%8D%E5%88%B6%E7%AE%97%E6%B3%95copying) 【内存被压缩一半，效率低】
> - [标记-整理算法(Mark-Compact)](https://www.cnblogs.com/cielosun/p/6674431.html#23-%E6%A0%87%E8%AE%B0-%E6%95%B4%E7%90%86%E7%AE%97%E6%B3%95mark-compact) 【将存活对象移向内存的一端。然后清除端边界外的对象】
> - [分代收集算法(Generational Collection)](https://www.cnblogs.com/cielosun/p/6674431.html#24-%E5%88%86%E4%BB%A3%E6%94%B6%E9%9B%86%E7%AE%97%E6%B3%95generational-collection)
>   - 新生代（复制算法，年龄>15就被移动到老生代中）
>     - Eden
>     - Survivor 0（From），Survivor 1（To）
>     - Young GC ，也叫 Minor GC
>     - 触发条件：
>       - 新生代采用“空闲指针”的方式来控制GC触发，指针保持最后一个在新生代分配的对象位置，当有新的对象要分配内存时，用于检查空间是否足够，不够就触发GC
>   - 老生代（标记-整理算法，一般经过2次以上标记）
>     - Full GC ，也叫Major GC
>     - 触发条件：
>       - 老生代空间不足（避免创建过大对象）
>       - Pemanet Generation空间不足 （增大Perm Gen空间，避免太多静态对象）
>       - GC后晋升到老生代的平均大小大于老生代剩余空间 （控制好新生代和旧生代的比例）
>       - 手动调用System.gc()；（垃圾回收不要手动触发，尽量依靠JVM自身的机制）
>
> **垃圾回收器** ：
>
> - [Serial/Serial Old](https://www.cnblogs.com/cielosun/p/6674431.html#31-serialserial-old)
> - [ParNew](https://www.cnblogs.com/cielosun/p/6674431.html#32-parnew)
> - [Parallel Scavenge](https://www.cnblogs.com/cielosun/p/6674431.html#33-parallel-scavenge)
> - [Parallel Old](https://www.cnblogs.com/cielosun/p/6674431.html#34-parallel-old)
> - [CMS](https://www.cnblogs.com/cielosun/p/6674431.html#35-cms)
> - [G1](https://www.cnblogs.com/cielosun/p/6674431.html#36-g1)
>
> [参考文章]：
>
> 1. [文章1](https://www.cnblogs.com/cielosun/p/6674431.html)
> 2. [文章2](http://blog.csdn.net/ooppookid/article/details/51523486)

11、什么CAS，什么是ABA问题，如何解决？

> - CAS(Compare and Swap)，比较并交换，实现并发算法时常用到的一种技术
> - CAS的思想很简单：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。
> - 如果变量V初次读取的时候是A，并且在准备赋值的时候检查到它仍然是A。如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类`AtomicStampedReference`，它可以通过控制变量值的版本来保证CAS的正确性。

12、Spring中的IOC和AOP是如何实现的？

> - IOC是使用Java的反射机制实现的
> - AOP是根据动态代理机制实现的

13、什么是SpringBeans？

> **具体说来，它是被Spring框架容器初始化、配置和管理的对象。**

14、说说Dubbo的内部实现原理？

- [文章](https://www.cnblogs.com/panxuejun/p/6094790.html)
- 序列化/反序列化【hessian、protobuf、thrift、avro】
- 消息结构【服务名、方法名、方法参数类型&值、超时时间、requestID】
- 网络通信【Netty NIO】
- requestID 【异步请求，返回的结果线程如何找到发送的线程】
- zk

15、了解Netty么？

- 不了解。。。

16、说说什么是NIO、AIO？

17、常用的设计模式及好处？

- 工厂模式
  - 客户类和工厂类分开。消费者任何时候需要某种产品，只需向工厂请求即可。
  - 缺点是当产品修改时，工厂类也要做相应的修改
- 抽象工厂模式
  - 核心工厂类不再负责所有产品的创建，而是将具体创建的工作交给子类去做。更抽象，更灵活。
- 单例模式（double-check、静态内部类、枚举）
  - 全局只有一个实例，节省系统资源。所有的对象都访问一个实例，保证了数据的安全性。
- 适配器模式
  - 可以屏蔽掉一些不需要关注的细节

18、Java IO中经常使用的类？

19、了解Java中的线程池么？

20、Java中都有哪些锁？

21、MySql数据库都有哪些引擎？它们的区别是什么？

- 常用的分为InnoDB和MyISAM，区别如下：
  - 事物：InnoDB支持，MyISAM不支持
  - 外键：InnoDB支持，MyISAM不支持

22、谈谈MySql索引的类型和实现方式？

> - **唯一索引** 唯一索引是不允许其中任何两行具有相同索引值的索引。当现有数据中存在重复的键值时，大多数数据库不允许将新创建的唯一索引与表一起保存。数据库还可能防止添加将在表中创建重复键值的新数据。例如，如果在employee表中职员的姓(lname)上创建了唯一索引，则任何两个员工都不能同姓。 
> - **主键索引** 数据库表经常有一列或列组合，其值唯一标识表中的每一行。该列称为表的主键。 在数据库关系图中为表定义主键将自动创建主键索引，主键索引是唯一索引的特定类型。该索引要求主键中的每个值都唯一。当在查询中使用主键索引时，它还允许对数据的快速访问。 
> - **聚集索引** 在聚集索引中，表中行的物理顺序与键值的逻辑（索引）顺序相同。**一个表只能包含一个聚集索引。**

23、MySql慢查询如何定位和优化？

24、JVM内置了哪些垃圾回收器？

25、什么是Fail-Fast、Fail-Safe模式？

26、说说ThreadLocal

27、知道volatile关键字么？

> - 内存可见性
> - 禁止指令重排
> - 不能保证原子性

- 内存可见性：通俗来说就是，线程A对一个volatile变量的修改，对于**其它线程**来说是可见的，即线程**每次获取**volatile变量的**值都是最新的**。

- 不能保证原子性

- 操作普通变量的流程

  读操作会优先读取工作内存的数据，如果工作内存中不存在，则从主内存中拷贝一份数据到工作内存中；写操作只会修改工作内存的副本数据，这种情况下，其它线程就无法读取变量的最新值。

- volatile关键字修饰的变量

  对于volatile变量，读操作时JMM会把工作内存中对应的值设为无效，要求线程从主内存中读取数据；写操作时JMM会把工作内存中对应的数据刷新到主内存中，这种情况下，其它线程就可以读取变量的最新值。

28、synchronized关键字

29、使用过SpringCloud么？

30、知道AQS（同步器）么？

31、Spring事物的隔离级别？

32、MyBatis的执行流程？

33、Linux常用命令

> - [Linux下常用的shell命令记录](http://www.cnblogs.com/lizhenghn/p/3682014.html)

34、Redis的数据类型、常用命令和使用场景？

> - **`Redis`**常用的数据类型有： `String`、`List`、`Set`、`Sorted Set` 、`Hash`
> - 操作**`KEY`**常用命令：
>   - `DEL` key                         如果存在删除键
>   - `DUMP` key                        返回存储在指定键的值的序列化版本
>   - `EXISTS` key                      此命令检查该键是否存在
>   - `EXPIRE` key seconds              指定键的过期时间
>   - `EXPIREAT` key timestamp          指定的键过期时间。在这里，时间是在Unix时间戳格式
>   - `PEXPIRE` key milliseconds        设置键以毫秒为单位到期
>   - `PEXPIREAT` key milliseconds-timestamp        设置键在Unix时间戳指定为毫秒到期
>   - `KEYS` pattern                    查找与指定模式匹配的所有键
>   - `MOVE` key db                     移动键到另一个数据库
>   - `PERSIST` key                     移除过期的键
>   - `PTTL` key                        以毫秒为单位获取剩余时间的到期键。
>   - `TTL` key                         获取键到期的剩余时间。
>   - `RANDOMKEY`                       从Redis返回随机键
>   - `RENAME` key newkey               更改键的名称
>   - `RENAMENX` key newkey             重命名键，如果新的键不存在
>   - `TYPE` key                        返回存储在键的数据类型的值。
> - **String**
>   - 使用场景：普通单值存储、`JSON`格式存储、数字存储
>   - 常用命令：`SET`、`GET` 、`MSET`、`MGET`、`SETEX`、`SETNX`、`INCR`、`INCRBY`、`APPEND`
> - **List** 【双向链表，列表的最大长度为**2^32 - 1**，也即每个列表支持超过40亿个元素】
>   - 使用场景：关注列表、粉丝列表。轻量级消息队列，生产者`push`，消费者`pop/bpop`。
>   - 常用命令：
>     - `BLPOP`
>       `BLPOP key1 [key2 ] timeout `取出并获取列表中的第一个元素，或阻塞，直到有可用
>     - `BRPOP`
>       `BRPOP key1 [key2 ] timeout `取出并获取列表中的最后一个元素，或阻塞，直到有可用
>     - `BRPOPLPUSH`
>       `BRPOPLPUSH source destination timeout `从列表中弹出一个值，它推到另一个列表并返回它;或阻塞，直到有可用
>     - `LINDEX`
>       `LINDEX key index `从一个列表其索引获取对应的元素
>     - `LINSERT`
>       `LINSERT key BEFORE|AFTER pivot value `在列表中的其他元素之后或之前插入一个元素
>     - **`LLEN`**
>       `LLEN key `获取列表的长度
>     - **`LPOP`**
>       `LPOP key `获取并取出列表中的第一个元素
>     - **`LPUSH`**
>       `LPUSH key value1 [value2] `在前面加上一个或多个值的列表
>     - `LPUSHX`
>       `LPUSHX key value `在前面加上一个值列表，仅当列表中存在
>     - **`LRANGE`**
>       `LRANGE key start stop `从一个列表获取各种元素
>     - **`LREM`**
>       `LREM key count value `从列表中删除元素
>     - **`LSET`**
>       `LSET key index value `在列表中的索引设置一个元素的值
>     - `LTRIM`
>       `LTRIM key start stop `修剪列表到指定的范围内
>     - **`RPOP`**
>       `RPOP key `取出并获取列表中的最后一个元素
>     - `RPOPLPUSH`
>       `RPOPLPUSH source destination `删除最后一个元素的列表，将其附加到另一个列表并返回它
>     - **`RPUSH`**
>       `RPUSH key value1 [value2] `添加一个或多个值到列表
>     - `RPUSHX`
>       `RPUSHX key value `添加一个值列表，仅当列表中存在
> - Hash【Redis Hash对应Value内部实际就是一个HashMap，这个Hash的成员比较少时Redis为了节省内存会采用类似一维数组的方式来紧凑存储，而不会采用真正的HashMap结构】
>   - 使用场景：假设有多个用户及对应的用户信息，可以用来存储以用户ID为key，将用户信息序列化为比如json格式做为value进行保存
>   - 常用命令：
>     - `HDEL`
>     - `HDEL key field[field...] `删除对象的一个或几个属性域，不存在的属性将被忽略
>     - `HEXISTS`
>     - `HEXISTS key field `查看对象是否存在该属性域
>     - `HGET`
>     - `HGET key field `获取对象中该field属性域的值
>     - `HGETALL`
>     - `HGETALL key `获取对象的所有属性域和值
>     - `HINCRBY`
>     - `HINCRBY key field value `将该对象中指定域的值增加给定的value，原子自增操作，只能是integer的属性值可以使用
>     - `HINCRBYFLOAT`
>     - `HINCRBYFLOAT key field increment `将该对象中指定域的值增加给定的浮点数
>     - `HKEYS`
>     - `HKEYS key `获取对象的所有属性字段
>     - `HVALS`
>     - `HVALS key `获取对象的所有属性值
>     - `HLEN`
>     - `HLEN key `获取对象的所有属性字段的总数
>     - `HMGET`
>     - `HMGET key field[field...] `获取对象的一个或多个指定字段的值
>     - `HSET`
>     - `HSET key field value `设置对象指定字段的值
>     - `HMSET`
>     - `HMSET key field value [field value ...] `同时设置对象中一个或多个字段的值
>     - `HSETNX`
>     - `HSETNX key field value `只在对象不存在指定的字段时才设置字段的值
>     - `HSTRLEN`
>     - `HSTRLEN key field `返回对象指定field的value的字符串长度，如果该对象或者field不存在，返回0.
>     - `HSCAN`
>     - `HSCAN key cursor [MATCH pattern][COUNT count] `类似`SCAN`命令
>   - **Set**
>     - 使用场景：微博应用中，每个用户关注的人存在一个集合中，就很容易实现求两个人的共同好友功能。
>     - 常用命令：
>       - `SADD`
>       - `SADD key member [member ...]` 添加一个或者多个元素到集合(set)里
>       - `SACRD`
>       - `SCARD key `获取集合里面的元素数量
>       - `SDIFF`
>       - `SDIFF key [key ...] `获得队列不存在的元素
>       - `SDIFFSTORE`
>       - `SDIFFSTORE destination key [key ...] `获得队列不存在的元素，并存储在一个关键的结果集
>       - `SINTER`
>       - `SINTER key [key ...] `获得两个集合的交集
>       - `SINTERSTORE`
>       - `SINTERSTORE destination key [key ...] `获得两个集合的交集，并存储在一个集合中
>       - `SISMEMBER`
>       - `SISMEMBER key member `确定一个给定的值是一个集合的成员
>       - `SMEMBERS`
>       - `SMEMBERS key `获取集合里面的所有key
>       - `SMOVE`
>       - `SMOVE source destination member `移动集合里面的一个key到另一个集合
>       - `SPOP`
>       - `SPOP key [count] `获取并删除一个集合里面的元素
>       - `SRANDMEMBER`
>       - `SRANDMEMBER key [count] `从集合里面随机获取一个元素
>       - `SREM`
>       - `SREM key member [member ...] `从集合里删除一个或多个元素，不存在的元素会被忽略
>       - `SUNION`
>       - `SUNION key [key ...] `添加多个set元素
>       - `SUNIONSTORE`
>       - `SUNIONSTORE destination key [key ...] `合并set元素，并将结果存入新的set里面
>       - `SSCAN`
>       - `SSCAN key cursor [MATCH pattern] [COUNT count] `迭代`set`里面的元素
>
> [参考链接]
>
> - [Redis常用数据类型介绍、使用场景及其操作命令](https://www.cnblogs.com/lizhenghn/p/5322887.html)
> - [Redis](https://www.cnblogs.com/Eason-S/p/5837774.html)

35、微服务中如何保证数据的一致性？

> - 1、同步事件服务，通过发送同步消息通知来保证
> - 2、异步事件服务
> - 3、外部事件服务（额外的网络通信）
> - 4、可靠事件通知（1. 事件的正确发送； 2. 事件的重复消费。）
> - 5、业务补偿机制（数据一致性的时效性很低，多个服务常常可能处于数据不一致的情况。）
> - 6、TCC(try、confirm、cancel)

36、ZooKeeper的使用场景？

- [zookeeper典型应用场景之一：master选举](https://www.cnblogs.com/nevermorewang/p/5611807.html)

37、**常用数据结构及其实现？**

38、Docker/区块链

39、什么情况下会产生死锁？

40、Redis有哪几种数据淘汰策略？【redis 每服务客户端执行一个命令的时候，会检测使用的内存是否超额。如果超额，即进行数据淘汰。】

> 1. **volatile-lru**：从<u>已设置过期时间</u>的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
> 2. **volatile-ttl**：从<u>已设置过期时间</u>的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
> 3. **volatile-random**：从<u>已设置过期时间</u>的数据集（server.db[i].expires）中任意选择数据淘汰
> 4. **allkeys-lru**：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
> 5. **allkeys-random**：从数据集（server.db[i].dict）中任意选择数据淘汰
> 6. **no-enviction（驱逐）**：禁止驱逐数据

41、Redis一个字符串类型的值能存储最大容量是多少？

> - 根据Redis官网的说法，应该是521MBMB。其余的（list、set、hash）都是2的32次方。

42、Redis集群方案应该怎么做？都有哪些方案？

43、消息队列的实现原理？

44、说说类的加载顺序？

> 1. 父类的static静态代码块
> 2. 子类的static静态代码块
> 3. 类内部变量
> 4. 静态成员类
> 5. 普通成员类

45、并发包下都用过哪些类？

> - [并发包诸类概览](http://www.raychase.net/1912)

46、Redis持久化方式？

- RDB（默认持久化方式，通过快照完成的。）
  - 当在指定时间内被更改的键的个数大于指定数值时就会进行快照。
  - 文件名是：dump.rdb
  - 可修改文件名和路径
  - Save （阻塞其他线程）
  - BGSave（fork方式，需要至少2倍的存储空间用于复制）
- AOF（将发送到Redis服务端的每一条命令都记录下来，并且保存到硬盘中的AOF文件）
  - 系统每30秒同步一次
    - appendfsyncalways  每次都同步 （最安全但是最慢）
    - appendfsync everysec  每秒同步 （默认的同步策略）
    - appendfsyncno  不主动同步，由操作系统来决定 （最快但是不安全）
  - 默认的文件名是appendonly.aof
  - BGREWRITEAOF 重写

47、Linux下Redis安装

> - mkdir -p /usr/local/src/redis
> - cd /usr/local/src/redis
> - wget <http://download.redis.io/releases/redis-2.8.11.tar.gz>
> - tar xzfredis-2.8.11.tar.gz
> - cdredis-2.8.11
> - make
> - makeinstall
> - 修改配置文件，使用redis后台运行：
> - vi /etc/redis.conf 
> - daemonize yes
> - 启动
> - redis-server/etc/redis.conf 

48、Redis事物是如何实现的？

> Redis事务通常会使用`MULTI,EXEC,WATCH`等命令来完成
>
> watch指令在redis事物中提供了CAS的行为。为了检测被watch的keys在是否有多个clients同时改变引起冲突，这些keys将会被监控。如果至少有一个被监控的key在执行exec命令前被修改，整个事物不执行任何动作，从而保证原子性操作，并且执行exec会得到null的结果。

49、Redis主从、集群？

50、TCP/IP三次握手、四次挥手

51、流量控制与滑动窗口？

52、什么情况下索引会失效？

53、什么是动态代理，都有哪些方式？

54、知道一致性hash么？

55、数据库的锁（行锁，表锁，页级锁，意向锁，读锁，写锁，悲观锁，乐观锁，以及加锁的select sql方式）

56、Java的线程模型？

- 使用一对一的线程模型实现的，一条Java线程就映射到一条轻量级进程之中。

- Java线程调度：线程调度是指系统为线程分配处理器使用权的过程

  - **协同式调度**

    线程的执行时间由线程本身来控制，线程把自己的工作执行完了之后，要主动通知系统切换到另外一个线程上。

  - **抢占式调度**（Java使用的线程调度方式就是抢占式调度）

    那么每个线程将由系统来分配执行时间，线程的切换不由线程本身来决定（在Java中，Thread.yield（）可以让出执行时间，但是要获取执行时间的话，线程本身是没有什么办法的）

  - 线程优先级(1-10，可设置，不靠谱)

- Java线程的状态

    Java语言定义了5种线程状态，在任意一个时间点，一个线程只能有且只有其中的一种状态，这5种状态分别如下。

  1. 新建（New）：创建后尚未启动的线程处于这种状态。

  2. 运行（Runable）：Runable包括了操作系统线程状态中的Running和Ready，也就是处于此状态的线程有可能正在执行，也有可能正在等待着CPU为它分配执行时间。

  3. 无限期等待（Waiting）：处于这种状态的线程不会被分配CPU执行时间，它们要等待被其他线程显式地唤醒。

     以下方法会让线程陷入无限期的等待状态：

  - 没有设置Timeout参数的Object.wait（）方法。

  - 没有设置Timeout参数的Thread.join（）方法。
  - LockSupport.park（）方法。

    4. 限期等待（Timed Waiting）：处于这种状态的线程也不会被分配CPU执行时间，不过无须等待被其他线程显式地唤醒，在一定时间之后它们会由系统自动唤            醒。

  ​     以下方法会让线程进入限期等待状态：

  - Thread.sleep（）方法。

  - 设置了Timeout参数的Object.wait（）方法。
  - 设置了Timeout参数的Thread.join（）方法。
  - LockSupport.parkNanos（）方法。
  - LockSupport.parkUntil（）方法。

    5. 阻塞（Blocked）：线程被阻塞了，“阻塞状态”与“等待状态”的区别是：“阻塞状态”在等待着获取到一个排他锁，这个事件将在另外一个线程放弃这个锁的时候            发生；而“等待状态”则是在等待一段时间，或者唤醒动作的发生。在程序等待进入同步区域的时候，线程将进入这种状态。
    6. 结束（Terminated）：已终止线程的线程状态，线程已经结束执行。

  ​

57、Java中如何序列化一个对象？

- ```java
  //序列化：
  ObjectOutputStream os = new ObjectOutputStream( new FileOutputStream("C:/wxp.txt")); 
  os.writeObject(user);
  os.close();
  //反序列化
  ObjectInputStream is = new ObjectInputStream( new FileInputStream("C:/wxp.txt"));  
  User temp = (User) is.readObject();
  ```

58、Java中多态的实现机制？

- 重写Override是父类与子类之间多态性的一种表现
- 重载Overload是一个类中多态性的一种表现.

59、数据库三范式是什么？

答案一：

- 一范式就是数据表中的每一列(字段)，必须是不可拆分的最小单元，也就是确保每一列的原子性
- 二范式就是要有主键，要求其他字段都依赖于主键
- 三范式就是要消除传递依赖，方便理解，可以看做是“消除冗余”。

答案二：

- 第一范式（1NF）：

  数据表中的每一列(字段)，必须是不可拆分的最小单元，也就是确保每一列的原子性。

  例如： userInfo: '山东省烟台市 1318162008' 依照第一范式必须拆分成userInfo: '山东省烟台市'　　 userTel: '1318162008'两个字段


- 第二范式（2NF）：

  满足1NF后要求表中的所有列，都必需依赖于主键，而不能有 任何一列与主键没有关系（一个表只描述一件事情）。例如：订单表只能描述订单相关的信息，所以所有的字段都必须与订单ID相关。

  产品表只能描述产品相关的信息，所以所有的字段都必须与产品ID相关。因此在同一张表中不能同时出现订单信息与产品信息。


- 第三范式（3NF）：

  满足2NF后，要求：表中的每一列都要与主键直接相关，而不是间接相关（表中的每一列只能依赖于主键）

  例如：订单表中需要有客户相关信息，在分离出客户表之后，订单表中只需要有一个用户ID即可，而不能有其他的客户信息，因为其他的用户信息是直接关联于用户ID，而不是关联于订单ID。

[查看知乎回答](https://www.zhihu.com/question/24696366)

60、数据库ACID

- 原子性
- 一致性
- 隔离性
- 持久性

61、Redis高级应用及技巧

> [Redis高级应用及技巧](http://blog.csdn.net/u011204847/article/details/51302109)

62、MyBatis原理

63、MQ的原理

64、Tomcat Server处理一个http请求的过程

> 假设来自客户的请求为：
> http://localhost:8080/wsota/wsota_index.jsp
> 1) 请求被发送到本机端口8080，被在那里侦听的Coyote HTTP/1.1 Connector获得
> 2) Connector把该请求交给它所在的Service的Engine来处理，并等待来自Engine的回应
> 3) Engine获得请求localhost/wsota/wsota_index.jsp，匹配它所拥有的所有虚拟主机Host
> 4) Engine匹配到名为localhost的Host（即使匹配不到也把请求交给该Host处理，因为该Host被定义为该Engine的默认主机）
> 5) localhost Host获得请求/wsota/wsota_index.jsp，匹配它所拥有的所有Context
> 6) Host匹配到路径为/wsota的Context（如果匹配不到就把该请求交给路径名为”"的Context去处理）
> 7) path=”/wsota”的Context获得请求/wsota_index.jsp，在它的mapping table中寻找对应的servlet
> 8) Context匹配到URL PATTERN为*.jsp的servlet，对应于JspServlet类
> 9) 构造HttpServletRequest对象和HttpServletResponse对象，作为参数调用JspServlet的doGet或doPost方法

> 10)Context把执行完了之后的HttpServletResponse对象返回给Host
> 11)Host把HttpServletResponse对象返回给Engine
> 12)Engine把HttpServletResponse对象返回给Connector
> 13)Connector把HttpServletResponse对象返回给客户browser

65、https://www.cnblogs.com/xiaozhang2014/p/7821104.html

66、Dubbo实现原理简单介绍

> - 通过[spring](http://lib.csdn.net/base/javaee) bean的方式管理配置及实例
> - ​