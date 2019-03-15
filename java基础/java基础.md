#### 1，缓存池概念

如： new Integer(123) 与 Integer.valueOf(123) 的区别 

```
new Integer(123) 每次都会新建一个对象；
Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。
```

```java
Integer x = new Integer(123);
Integer y = new Integer(123);
System.out.println(x == y);    // false
Integer z = Integer.valueOf(123);
Integer k = Integer.valueOf(123);
System.out.println(z == k);   // true
```



#### 2，String设计为不可变有什么好处？

- **可以缓存 hash 值** 

  ​	因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。 

  

- **String Pool 的需要** 

  ​	如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。 

  ![1552563255551](https://gitee.com/qianjiangtao/my-image/blob/master/java/1552563255551.png)

- **安全性** 

  ​	String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 对象的那一方以为现在连接的是其它主机，而实际情况却不一定是。 

  

- **线程安全** 

  String 不可变性天生具备线程安全，可以在多个线程中安全地使用。 

  

####3，String，`StringBuffer` and `StringBuilder`

**1. 可变性**

- String 不可变
- StringBuffer 和 StringBuilder 可变

**2. 线程安全**

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步



#### 4，隐式转换

因为字面量 1 是 int 类型，它比 short 类型精度要高，因此不能隐式地将 int 类型下转型为 short 类型。 

```java
short s1 = 1;
// s1 = s1 + 1; //不能执行
```

但是使用 += 或者 ++ 运算符可以执行隐式类型转换。 

```java
s1 += 1;
// s1++;
```

上面的语句相当于将 s1 + 1 的计算结果进行了向下转型： 

```java
s1 = (short) (s1 + 1);
```



####5，访问权限

![1552564469222](https://gitee.com/qianjiangtao/my-image/raw/master/java/1552564469222.png)



#### 6，深拷贝和浅拷贝

- 浅拷贝：对基本数据类型进行值传递，对引用数据类型进行引用传递般的拷贝，此为浅拷贝。 
- 深拷贝：对基本数据类型进行值传递，对引用数据类型，创建一个新的对象，并复制其内容，此为深拷贝。 



####7,静态变量及语句加载顺序

- 不存在继承关系的情况下

  ```java
  public static String staticField = "静态变量";
  static {
      System.out.println("静态语句块");
  }
  public String field = "实例变量";
  {
      System.out.println("普通语句块");
  }
  public InitialOrderTest() {
      System.out.println("构造函数");
  }
  ```

- 存在继承的情况下

  ```java
  父类（静态变量、静态语句块）
  子类（静态变量、静态语句块）
  父类（实例变量、普通语句块）
  父类（构造函数）
  子类（实例变量、普通语句块）
  子类（构造函数）
  ```

####8，反射

**反射的优点: **

- **可扩展性** ：应用程序可以利用全限定名创建可扩展对象的实例，来使用来自外部的用户自定义类。
- **类浏览器和可视化开发环境** ：一个类浏览器需要可以枚举类的成员。可视化开发环境（如 IDE）可以从利用反射中可用的类型信息中受益，以帮助程序员编写正确的代码。
- **调试器和测试工具** ： 调试器需要能够检查一个类里的私有成员。测试工具可以利用反射来自动地调用类里定义的可被发现的 API 定义，以确保一组测试中有较高的代码覆盖率。

**反射的缺点：** 

尽管反射非常强大，但也不能滥用。如果一个功能可以不用反射完成，那么最好就不用。在我们使用反射技术时，下面几条内容应该牢记于心。

- **性能开销** ：反射涉及了动态类型的解析，所以 JVM 无法对这些代码进行优化。因此，反射操作的效率要比那些非反射操作低得多。我们应该避免在经常被执行的代码或对性能要求很高的程序中使用反射。
- **安全限制** ：使用反射技术要求程序必须在一个没有安全限制的环境中运行。如果一个程序必须在有安全限制的环境中运行，如 Applet，那么这就是个问题了。
- **内部暴露** ：由于反射允许代码执行一些在正常情况下不被允许的操作（比如访问私有的属性和方法），所以使用反射可能会导致意料之外的副作用，这可能导致代码功能失调并破坏可移植性。反射代码破坏了抽象性，因此当平台发生改变的时候，代码的行为就有可能也随着变化。



####9，泛型擦除

代码一：

```java
Class c1 = new ArrayList<Integer>().getClass();
Class c2 = new ArrayList<String>().getClass(); 
System.out.println(c1 == c2);
//结果: true
```



> 显然在平时使用中，`ArrayList<Integer>()`和`new ArrayList<String>()`是完全不同的类型，但是在这里，程序却的的确确会输出`true`。
>
> 这就是Java泛型的类型擦除造成的，因为不管是`ArrayList<Integer>()`还是`new ArrayList<String>()`，都在编译器被编译器擦除成了`ArrayList`。那编译器为什么要做这件事？原因也和大多数的Java让人不爽的点一样——兼容性。由于泛型并不是从Java诞生就存在的一个特性，而是等到SE5才被加入的

代码二：

```java
List<Integer> list = new ArrayList<Integer>();
Map<Integer, String> map = new HashMap<Integer, String>();
System.out.println(Arrays.toString(list.getClass().getTypeParameters()));
System.out.println(Arrays.toString(map.getClass().getTypeParameters()));
//结果
[E]
[K, V]
```

我们期待的是得到泛型参数的类型，但是实际上我们只得到了一堆占位符。 

####10，强引用，软引用，弱引用，虚引用

![1552641168445](https://gitee.com/qianjiangtao/my-image/raw/master/java/1552641168445.png)

#### 11，容器介介绍

- **Collection**

  ![1552638789125](https://gitee.com/qianjiangtao/my-image/raw/master/java/1552638789125.png)

  1.Set

  - TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。
  - HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。
  - LinkedHashSet：具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序。

  2.List

  - ArrayList：基于动态数组实现，支持随机访问。线程不安全
  - Vector：和 ArrayList 类似，但它是线程安全的。
  - LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

  3.Queue

  - [LinkedList](http://www.tianxiaobo.com/2018/01/31/LinkedList-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-JDK-1-8/)：可以用它来实现双向队列。
  - PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

- **Map**

  ![1552639432570](https://gitee.com/qianjiangtao/my-image/raw/master/java/1552639432570.png)

- [TreeMap](http://www.tianxiaobo.com/2018/01/11/TreeMap%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/)：基于红黑树实现。
- [HashMap](http://www.tianxiaobo.com/2018/01/18/HashMap-%E6%BA%90%E7%A0%81%E8%AF%A6%E7%BB%86%E5%88%86%E6%9E%90-JDK1-8/)：基于哈希表实现。
- HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程可以同时写入 HashTable 并且不会导致数据不一致。它是遗留类，不应该去使用它。现在可以使用 ConcurrentHashMap 来支持线程安全，并且 ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
- [LinkedHashMap](http://www.tianxiaobo.com/2018/01/31/LinkedList-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90-JDK-1-8/)：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。

