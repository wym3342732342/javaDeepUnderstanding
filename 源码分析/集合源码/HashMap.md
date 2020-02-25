## 前言：

&emsp;&emsp;此篇主要从`构造`、`put`、`get`讲解`HashMap`



## 一、主要成员

```java
/**
  * 数组的默认初始长度，java规定hashMap的数组长度必须是2的次方
  * 扩展长度时也是当前长度 << 1。
  */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 

// 数组的最大长度
static final int MAXIMUM_CAPACITY = 1 << 30;//int 的 最大值/2

// 默认负载因子，当元素个数超过这个比例则会执行数组扩充操作。
static final float DEFAULT_LOAD_FACTOR = 0.75f;

// 树形化阈值，当链表节点个大于等于TREEIFY_THRESHOLD - 1时，
// 会将该链表换成红黑树。
static final int TREEIFY_THRESHOLD = 8;

// 解除树形化阈值，当链表节点小于等于这个值时，会将红黑树转换成普通的链表。
static final int UNTREEIFY_THRESHOLD = 6;

// 最小树形化的容量，即：当内部数组长度小于64时，不会将链表转化成红黑树，而是优先扩充数组。
static final int MIN_TREEIFY_CAPACITY = 64;

// 这个就是hashMap的内部数组了，而Node则是链表节点对象。
transient Node<K,V>[] table;

// 下面三个容器类成员，作用相同，实际类型为HashMap的内部类KeySet、Values、EntrySet。
// 他们的作用并不是缓存所有的key或者所有的value，内部并没有持有任何元素。
// 而是通过他们内部定义的方法，从三个角度（视图）操作HashMap，更加方便的迭代。
// 关注点分别是键，值，映射。
transient Set<K> keySet;  // AbstractMap的成员
transient Collection<V> values; // AbstractMap的成员
transient Set<Map.Entry<K,V>> entrySet;

// 元素个数，注意和内部数组长度区分开来。
transient int size;

// 再上一篇文章中说过，是容器结构的修改次数，fail-fast机制。
transient int modCount;

// 阈值，超过这个值时扩充数组。 threshold = capacity * load factor，具体看上面的静态常量。
int threshold;

// 装在因子，具体看上面的静态常量。
final float loadFactor;
```



## 二、构造方法

**先来看看无参构造：**

```java
/**
 * 使用默认容量构建一个空的HashMap
 * 容量为16，负载因子为0.75
 */
public HashMap() {
  this.loadFactor = DEFAULT_LOAD_FACTOR;
}
```



**创建一个指定容量初始容量的HashMap：**

```java
/**
 * 负载因子仍然是默认的0.75
 */
public HashMap(int initialCapacity) {
  this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```



**指定初始容量和负载因子：**

```java
public HashMap(int initialCapacity, float loadFactor) {
  if (initialCapacity < 0)
    throw new IllegalArgumentException("Illegal initial capacity: " +
                                       initialCapacity);
  if (initialCapacity > MAXIMUM_CAPACITY)
    initialCapacity = MAXIMUM_CAPACITY;
  if (loadFactor <= 0 || Float.isNaN(loadFactor))
    throw new IllegalArgumentException("Illegal load factor: " +
                                       loadFactor);
  this.loadFactor = loadFactor;
  this.threshold = tableSizeFor(initialCapacity);
}
```

> &emsp;&emsp;从源码可以看出创建指定初始化容量和负载因子的HashMap时，会对参数做一个校验，然后计算出一个阀值【超过时扩容】。并且三个构造都没有去初始化底层数组。



**再来看一下计算底层数组大小的方法：**

```java
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

> &emsp;&emsp;该方法呢做了一些或运算后能达到一个16一下都是16，16后是32，32后是64一次类推的一个效果



## 三、常用方法

### 3.1 put方法

```java
/**
 * 并不是真正的put
 */
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}
```

> 我们发现put并没有进行真正的添加操作，而是调用了我们的另外一个方法：`putVal`.
>
> - hash()方法：通过hashCode和与运算得出的一个值

#### 3.1.1 putVal方法

```java
/**
 * 实现Map.put和相关方法。
 *
 * @param hash 哈希密钥
 * @param key K值
 * @param value value值
 * @param onlyIfAbsent 如果为true，请不要更改现有值
 * @param 如果为false，则表处于创建模式。
 * @return 前一个值；如果没有，则为null
 */
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
               boolean evict) {
    Node<K,V>[] tab;//当前的数组
    Node<K,V> p;//原位置上的节点
  	int n, i;//n是当前数组的长度，i是通过hash计算出来的存放位置
    if ((tab = table) == null || (n = tab.length) == 0)
      	//当前数组为空,扩容数组，给n赋值
        n = (tab = resize()).length;
  	//计算存放位置，判断当前位置是否存放节点
    if ((p = tab[i = (n - 1) & hash]) == null)
      	//存入数组
        tab[i] = newNode(hash, key, value, null);
    else {//存放位置没有正确计算
        Node<K,V> e;//暂存节点，如果key出现了冲突会存入被冲突对象
      	K k;//用于存放现有节点的Key
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
          	//当前节点的hash值 key 都相等
            e = p;
        else if (p instanceof TreeNode)//hash值或key值不想等，并且当前是红黑树
           //将p放入后，返回值存入e
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        else {//当前key值，或hash值不等 并且是链表形式
            for (int binCount = 0; ; ++binCount) {//遍历链表
                if ((e = p.next) == null) {//如果链表尾部没有值了
                    //将本节点放到结尾
                    p.next = newNode(hash, key, value, null);
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                      	//如果链表的长度已经达到了阀值，将链表转为红黑树
                        treeifyBin(tab, hash);
                    break;
                }
                //如果下一个节点与当前节点相同
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    break;//直接结束循环
                p = e;//用于指向下一节点
            }
        }
        if (e != null) { //key值冲突
            V oldValue = e.value;
            if (!onlyIfAbsent || oldValue == null)//如果当前为可覆盖模式，或者没有旧值
                e.value = value;//赋值
            afterNodeAccess(e);//hashMap中无用，什么都不做
            return oldValue;//返回旧的值
        }
    }
   //没有冲突的情况
    ++modCount;//容器修改次数+1
    if (++size > threshold)//判断当前size是否大于阀值
        resize();//扩容
    afterNodeInsertion(evict);//也是什么都不做
    return null;//返回null，没有旧值
}
```

> putval方法的几个点：判断底层数组有没有初始化，没有初始化就会初始化，计算`hashcode`和对应位置，判断`key`是否冲突，不冲突放入，判断链表的元素数量是否超过了阀值，超过了转为红黑树。如果是红黑树就调用红黑树节点提供的方法存入。

### 3.2 HashMap扩容方法：

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;//拿到当前的数组
    int oldCap = (oldTab == null) ? 0 : oldTab.length;//判断就数组的长度
    int oldThr = threshold;//老阀值
    int newCap, newThr = 0;//新的数组长度，新的阀值
    if (oldCap > 0) {//老数组非空
        if (oldCap >= MAXIMUM_CAPACITY) {//如果当前长度大于了最大值
            threshold = Integer.MAX_VALUE;//阀值变为int的最大值
            return oldTab;//返回老的数组
        }
      //新的数组长度小于最大值，并且老数组长度大于初始长度【如果此时新数组长度为最大值，阀值将由下次扩容是变化】
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                 oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; //新的阀值将是老阀值的2倍
    }
    else if (oldThr > 0) //数组没有初始化但存在阀值，也就是说自定义底层容器大小了。
        newCap = oldThr;//新的数组长度就等于阀值
    else {//初始化阀值，那么在创建HashMap时没有带参数
        newCap = DEFAULT_INITIAL_CAPACITY;//新的长度就是默认长度
      	//16 * 0.75
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    if (newThr == 0) {//如果新的阀值为0
        float ft = (float)newCap * loadFactor;//使用新的数组长度 * 负载因子
        //计算出新的阀值【是当前数组*0.75还是最大值】
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                  (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;//阀值等于新阀值
    @SuppressWarnings({"rawtypes","unchecked"})
    Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];//创建新的数组
    table = newTab;
  
    if (oldTab != null) {//如果旧的数组不为空
        for (int j = 0; j < oldCap; ++j) {//遍历旧数组
            Node<K,V> e;//暂存节点
            if ((e = oldTab[j]) != null) {//如果节点不为null
                oldTab[j] = null;//赋给e后赋值为空
                if (e.next == null)//e不存在下一个节点
                    newTab[e.hash & (newCap - 1)] = e;//重新计算存放位置放入
                else if (e instanceof TreeNode)//e是红黑树
                  	//将树分割存入
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {//存在下一个节点，并且是链表
                  	//按原始链表顺序，过滤出来扩容后位置不变的元素（低位=0），放在一起  
                    Node<K,V> loHead = null,//lo头
                 			 				loTail = null;//lo尾
                  	//按原始链表顺序，过滤出来扩容后位置改变到（index+oldCap）的元素（高位=0），放在一起  
                    Node<K,V> hiHead = null,//hi头
                  						hiTail = null;//hi尾
                    Node<K,V> next;//下一个
                    do {//循环遍历
                        next = e.next;
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)//尾部等于null，说明第一个元素
                                loHead = e;//头等于e
                            else
                                loTail.next = e;//否则尾的下一个=e
                            loTail = e;//唯一等于e
                        }
                        else {
                            if (hiTail == null)//尾部等于null，说明第一个元素
                                hiHead = e;//头部赋值
                            else
                                hiTail.next = e;//尾部的下一个赋值
                            hiTail = e;//尾部赋值
                        }
                    } while ((e = next) != null);//遍历下一节点，直至没有
                  	//位置不练的元素
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                  	//位置改变的元素
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;//返回新的数组
}
```

> 扩容方法包含了几块内容：`初始化数组`，`扩容并计算新阀值`，`将旧数组中的数据转到新数组`。
>
> 此处有两个问题：
>
> 一、`看不懂(e.hash & oldCap) == 0`什么意思
>
> ```java
> public class Test {
>     public static void main(String[] args) {
>         int oldCap = 32;
>         int newCap = 64;
>         int notChangeCount = 0;//统计变更位置了的个数
>         int changeCount = 0;//统计没有变更为值的个数
>         for (Integer i = 0; i < 10000; i++) {
>             if ((hash(i) & (oldCap - 1)) == (hash(i) & (newCap - 1))) {
>                 System.out.println("低位： " + (hash(i)&oldCap));
>                 System.out.println("旧值： " + (hash(i) & (oldCap - 1)));
>                 System.out.println("新值： " + (hash(i) & (newCap - 1)));
>                 System.out.println("这个数： " + i);
>                 changeCount++;
>             }else {
>                 if ((hash(i) & (newCap - 1)) - (hash(i) & (oldCap - 1)) == oldCap) {
>                     notChangeCount++;
>                 }
>             }
>         }
>         System.out.println("不需要变更位置： " + notChangeCount);//推测错误
>         System.out.println("需要变更位置： " + changeCount);//推测错误
>     }
> 
>     static final int hash(Object key) {
>         int h;
>         return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
>     }
> }
> ```
>
> 运行上边这段程序的值，`(e.hash & oldCap) == 0`时新的位置和老的位置必然相同
>
> **【前提：`数组大小必须遵许hashMap的扩容关系`】**
>
> 二、自定义了底层容器大小
>
> ```java
> else if (oldThr > 0) //数组没有初始化但存在阀值，也就是说自定义底层容器大小了。
> ```
>
> &emsp;&emsp;我们可以看到，走了这行代码就是自定义大小必走的代码。此时我们看看这个函数其功能：不满16为16，大于16不满32为32以此类推，这样也就保障了扩容时能将数据放到规定位置。
>
> ```java
> static final int tableSizeFor(int cap) {//上方已有介绍
> ```



### 3.3 get方法

```java
public V get(Object key) {
  Node<K,V> e;
  return (e = getNode(hash(key), key)) == null ? null : e.value;
}
```

> get方法还是对getNode的一个封装。

#### 3.3.1 getNode方法

```java
final Node<K,V> getNode(int hash, Object key) {
  Node<K,V>[] tab;//暂存当前数组
  Node<K,V> first, e;//存储第一节点
  int n;//当前数组的长度
  K k;
  if ((tab = table) != null && (n = tab.length) > 0 &&
      (first = tab[(n - 1) & hash]) != null) {//当前数组不为空，并且对应的位置有节点
    if (first.hash == hash && //检测第一个节点的key是不是要找的key
        ((k = first.key) == key || (key != null && key.equals(k))))
      return first;
    if ((e = first.next) != null) {//第一个不是要找的，遍历
      if (first instanceof TreeNode)//如果是红黑树实现时
        return ((TreeNode<K,V>)first).getTreeNode(hash, key);//使用红黑树查找对应key
      do {
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
          return e;
      } while ((e = e.next) != null);//循环遍历比对key
    }
  }
  return null;//如果没有找到就返回null
}
```

> getNode是相对比较简单的，简单看看就能理解。



### 3.4 remove方法

```java
public V remove(Object key) {
  Node<K,V> e;
  return (e = removeNode(hash(key), key, null, false, true)) == null ? null : e.value;
}
```

> remove方法同样是对removeNode的封装

#### 3.4.1 removeNode方法

```java
/**
 * 实现Map.remove和相关方法。
 *
 * @param hash key的hash值
 * @param key key
 * @param value 一个值，在matchValue为true时，删除的节点必须与当前value相等
 * @param matchValue 如果为true，则仅在值相等时删除
 * @param movable 
 * @return 返回删除节点，如果没有则为null
 */
final Node<K,V> removeNode(int hash, Object key, Object value,boolean matchValue, boolean movable) {
  Node<K,V>[] tab;//暂存当前数组
  Node<K,V> p;
  int n, index;//n为当前数组长度
  if ((tab = table) != null && (n = tab.length) > 0 &&
      (p = tab[index = (n - 1) & hash]) != null) {//如果当前数组存在，且对应的哈说地址存在
    Node<K,V> node = null, e;
    K k;
    V v;
    if (p.hash == hash &&
        ((k = p.key) == key || (key != null && key.equals(k))))//如果刚好第一个节点是要删除的
      node = p;//node就等于p
    else if ((e = p.next) != null) {//如果不是第一个节点
      if (p instanceof TreeNode)
        node = ((TreeNode<K,V>)p).getTreeNode(hash, key);//如果是红黑树，按照红黑树处理
      else {
        do {//循环遍历
          if (e.hash == hash &&
              ((k = e.key) == key ||
               (key != null && key.equals(k)))) {
            node = e;//对应节点赋给node
            break;
          }
          p = e;
        } while ((e = e.next) != null);
      }
    }
    if (node != null && (!matchValue || (v = node.value) == value ||
                         (value != null && value.equals(v)))) {//当前找到对应节点，并且不比对值删除，或者值也想等
      if (node instanceof TreeNode)//如果是红黑树，调用红黑树删除
        ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
      else if (node == p)//如果是第一个节点
        tab[index] = node.next;
      else
        p.next = node.next;//不为第一个节点
      ++modCount;
      --size;
      afterNodeRemoval(node);
      return node;
    }
  }
  return null;
}
```

