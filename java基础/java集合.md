## Java基础

### 一，集合

####  1.List

#####  1.1 `ArrayList`

- 为什么避免在for循环中对list进行删除操作？

  ```shell
  1.arrayList没有自动缩容机制导致底层数组大量的空闲空间不能被释放  ##调用trimToSize()进行缩小
  2.可能出现并发修改异常Java.util.ConcurrentModificationException，如果并发操作，需要对 Iterator 对象加锁。
  3.推荐使用迭代器删除
  ```

- `ArrayList`扩容机制

  ```
  1.发现容量不足，计算新的长度默认是1.5倍
  2.如果新的长度任然无法满足期望值则使用期望长度
  3.创建新的list，将老的数据copy过去
  ```

- 快速失败机制

  ```
  在 Java 集合框架中，很多类都实现了快速失败机制。该机制被触发时，会抛出并发修改异常ConcurrentModificationException
  ```



#####  1.2 `LinkedList`

- 擅长添加，删除操作，不擅长随机位置访问 
- 做实现栈，队列

**比较：**

```
1.ArrayList底层是数组结果,查询和修改快
2.LinkedList底层是链表结构的,增和删比较快,查询和修改比较慢
3.共同的特点是线程不安全
```



####  2.Map

#####  2.1 `HashMap`

- 特点

  ```
  1.HashMap 允许 null 键和 null 值，在计算哈键的哈希值时，null 键哈希值为 0
  2.键值对无需
  3.非线程安全
  4.HashMap 则使用了拉链式的散列算法
  ```

- `hashmap`引入了红黑树，那么在查找的时候是如何遍历红黑树？

- `hashmap`插入流程是怎么样的？

  ```
  1.当桶数组 table 为空时，通过扩容的方式初始化 table
  2.查找要插入的键值对是否已经存在，存在的话根据条件判断是否用新值替换旧值
  3.如果不存在，则将键值对链入链表中，并根据链表长度决定是否将链表转为红黑树
  4.判断键值对数量是否大于阈值，大于的话则进行扩容操作
  ```

- 扩容机制

  ```java
  table中的元素只有两种情况：
  元素hash值第N+1位为0：不需要进行位置调整
  元素hash值第N+1位为1：调整至原索引的两倍位置
  在resize方法中，确定元素hashi值第N+1位是否为0：
  若为0，则使用loHead与loTail，将元素移至新table的原索引处
  若不为0，则使用hiHead与hiHead，将元素移至新table的两倍索引处
  ```

- 链表树化

  ```shell
  链表长度大于等于 TREEIFY_THRESHOLD //8
  桶数组容量大于等于 MIN_TREEIFY_CAPACITY //64
  树化过程：
  1.比较键与键之间 hash 的大小，如果 hash 相同，继续往下比较
  2.检测键类是否实现了 Comparable 接口，如果实现调用 compareTo 方法进行比较
  3.如果仍未比较出大小，就需要进行仲裁了，仲裁方法为 tieBreakOrder（大家自己看源码吧）
  ```

  

一，多线程（https://www.cnblogs.com/skywang12345/p/java_threads_category.html）

