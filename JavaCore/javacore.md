# JAVA CORE

## HashMap 1.8

static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;  默认的初始化容量

static final float DEFAULT_LOAD_FACTOR = 0.75f; 默认加载因子，扩容因子

static final int TREEIFY_THRESHOLD = 8; 树型化的阈值

static final int MIN_TREEIFY_CAPACITY = 64; 树型化的最小容量

put流程

1. putVal(hash(key), key, value, false, true)

   1. return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16) //代表hashamap的key可以为null

2. 当前key在table中是不存在的就直接newNode(hash, key, value, null)，存入表中

3. 当前key的计算出来的位置在table中是存在的，那么就比较key的hash值是否相等&&key是否相等；

   如果是相同的，就将新的value替换旧值，返回旧值value；

4. 如果2不成立，则判断当前位置的结点是否属于TreeNode，如果属于TreeNode，则将新key/value存到该树中

5. 如果3不成立，则进行一个for循环，遍历当前节点上的链表，将新节点加入到链表的末尾

   1. 然后判断该链表的长度是否超过了8即TREEIFY_THRESHOLD，如果超过了，就将该链表转成红黑树
   2. 如果遇到相同的key，比较key的hash值是否相等&&key是否相等，如果是相同的，就将新的value替换旧值，返回旧值value

6. 最后判断当前table中的键值对数量超过了threshold（阈值），就要进行扩容，resize();

   newCap = oldCap << 1。

> ### 注：与1.7的区别
>
> 1.7插入新值时，是扩容后插值；1.8是先插值后扩容
>
> 1.7是头插法，这种操作在多线程扩容情况下会导致无限死循环；1.8是尾插法
>
> 1.7的结构是数组+链表；1.8的结构是数组+链表+红黑树

get取值就很直接

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
        Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
        if ((tab = table) != null && (n = tab.length) > 0 &&
            (first = tab[(n - 1) & hash]) != null) {
            if (first.hash == hash && // always check first node
                ((k = first.key) == key || (key != null && key.equals(k))))
                return first;
            if ((e = first.next) != null) {
                if (first instanceof TreeNode)
                    return ((TreeNode<K,V>)first).getTreeNode(hash, key);
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        return e;
                } while ((e = e.next) != null);
            }
        }
        return null;
    }
```

## ConcurrentHashMap

put操作

1. 计算key的hash码

   int hash = spread(key.hashCode());    

   ```
   static final int spread(int h) {
       return (h ^ (h >>> 16)) & HASH_BITS;
   }
   ```

2. 套了一个死循环 for (Node<K,V>[] tab = table;;) {}直到插入成功为止；

   1. 循环中先判断该表有没有初始化，没有就初始化Hash表

   2. 如果table存在，就计算当前key在table中的位置是否为null,内部使用的unsafe的可见性操作getObjectVolatile。如果为null,就进行插入操作，内部也使用的unsafe，CAS操作（无锁化）compareAndSwapObject，设置成功就直接跳出循环，结束。

      如果设置失败就直接进入下一次循环操作，直到插入成功。

   3. 如果当前位置是不为null,判断一下当前节点的状态是否为MOVED（移动状态），当前可能在进行扩容操作，辅助进行移动数据

   4. 如果以上几种判断都为false,则会进行和hashmap类似的插入操作，只不过是加了同步锁（synchronized），但是这里锁定对象不是整张表而是一个节点。里面的插入操作跟hashmap是一样的

3. 最后跳出循环后，增加元素个数的同时里面还会判断进行扩容操作，数据转移

get操作

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());//计算hash
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        
//eh=-1，说明该节点是一个ForwardingNode，正在迁移，此时调用ForwardingNode的find方法去nextTable里找。
//eh=-2，说明该节点是一个TreeBin，此时调用TreeBin的find方法遍历红黑树，由于红黑树有可能正在旋转变色，所以find里会有读写锁。
//eh>=0，说明该节点下挂的是一个链表，直接遍历该链表即可。
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

> ### 注：
>
> ConcurrentHashMap的取值操作是全程不加锁的，原因：
>
> - transient volatile Node<K,V>[] table;加了volatile,保证数组再扩容时的可见性
> - Node结点中volatile V val; 每个结点的值都加了volatile，保证结点值的修改的可见性
>
> 正是因为不加锁，所以读取数据的速度比其他线程安全的table要快很多