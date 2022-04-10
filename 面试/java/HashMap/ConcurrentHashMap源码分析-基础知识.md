# ConcurrentHashMap源码分析

### 一、 数据结构

数组+链表+红黑树

![image.png](https://fynotefile.oss-cn-zhangjiakou.aliyuncs.com/fynote/fyfile/2746/1648899910051/f534e3c2a61e424caf15206fafd48348.png)

### 二、正确理解安全性

```java
public class MyTest {

    public static ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

    public static void main(String[] args) {
        Integer count = map.get("count");
        if (count == null) {
            map.put("count", 1);
        } else {
            map.put("count",count + 1);
        }



        while (true) {
            Integer count = map.get("count");
            if (count == null) {
                // 将存储到map中后，会返回null，代表成功
                if (map.putIfAbsent("count", 1) == null) {
                    // putVal(key, value, true);
                    break;
                }
            } else {
                if (map.replace("count", count, count + 1)) {
                    break;
                }
            }
        }
    }

}
```

### 三、put方法开始

从put方法开始，发现put方法调用了putVal(key, value, false);

putIfAbsent，他调用的方式是：putVal(key, value, true);.

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key和value不允许为null~~~
    if (key == null || value == null) throw new NullPointerException();
    // 基于key计算出hash值
    int hash = spread(key.hashCode());
    // 声明了一个标识，暂时不管
    int binCount = 0;
    // 死循环，声明了一波tab，为当前ConcurrentHashMap的数组
    for (Node<K,V>[] tab = table;;) {
        // 因为Doug Lea老爷子，他的变量名都这样。
        // tab-数组  n-数组长度   i-数据要存储的索引位置
        // f-当前索引位置的数据
        Node<K,V> f; int n, i, fh;
        // 如果数组为null，或者数组的长度为0
        if (tab == null || (n = tab.length) == 0)
            // 初始化数组
            tab = initTable();
        // 数组已经初始化了，将数据插入的map中
        // 将 数组长度-1 & key的hash值，作为索引 ，配合tabAt方法获取这个索引位置的数据
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 当前索引位置没有数据，将数据插入到这个位置
            // 采用了CAS的方式，将数据插入，插入成功，返回true，  插入失败，返回false
            if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))
                // 成功！
                break;  
        }
    // 省略部分代码…………
}
tabAt(数组,i)-获取数组中i索引位置的数据（table[i]）
casTabAt(数组,1,2,3)-以CAS的方式，将数组中i位置的数据从2修改为3
```

### 四、spread方法（散列算法、尽量避免Hash冲突）

```java
// 调用方法拿hash值
int hash = spread(key.hashCode());

// 具体方法
static final int spread(int h) {

    return (h ^ (h >>> 16));
}



                  00000110 00011000 
                ^
00000110 00011000 00111010 00001110   - hash
&
00000000 00000000 00000000 00001111   - 15

(n - 1) & hash - 索引位置 

将key的hashcode值的高16位与低16位进行^运算，将结果与数组的长度-1进行&运算，得到当前数据需要添加到哪个索引位置，因为这样做可以尽可能的打散数据，避免hash冲突
// ------------------------

return (h ^ (h >>> 16)) & HASH_BITS;
保证key的hashcode值一定是一个正数！
因为负数有特殊含义：
static final int MOVED     = -1; // hash for forwarding nodes
static final int TREEBIN   = -2; // hash for roots of trees
static final int RESERVED  = -3; // hash for transient reservations
```

### 五、initTable方法

```java
sizeCtl：
-1：代表map数组正在初始化
小于-1：代表正在扩容(-2,有1个线程正在扩容，-3，代表有2个线程正在扩容)
0：还没有初始化
正数：如果没有初始化，代表要初始化的长度。  如果已经初始化了，代表扩容的阈值。
// 初始化数组
private final Node<K,V>[] initTable() {
    // …………
    Node<K,V>[] tab; int sc;
    // 判断数组初始化了嘛
    while ((tab = table) == null || tab.length == 0) {
        // sizeCtl复制给sc，并判断是否小于0
        if ((sc = sizeCtl) < 0)
            // 让一手~~~
            Thread.yield(); 
        // sizeCtl大于等于0，以CAS的方式，将sizeCtl设置为-1
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                // 我要开始初始化了！！！
                // 再次做了判断（单例模式懒汉的DCL）
                if ((tab = table) == null || tab.length == 0) {
                    // 获取数组初始化的长度，如果sc>0，以sc作为长度，如果sc为0，就默认16
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    // table初始化完毕
                    table = tab = nt;
                    // 得到下次扩容的阈值，赋值给sc
                    sc = n - (n >>> 2);
                }
            } finally {
                // 将sc复制给sizeCtl
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

### 六、hash冲突存储

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // …………
    for (Node<K,V>[] tab = table;;) {
        // …………
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            // 出现了hash冲突，需要将数据挂到链表上，或者添加到红黑树中
            V oldVal = null;
            // 锁住当前桶的位置
            synchronized (f) {
                // 拿到i索引位置的数据，判断跟锁是不是一个。
                if (tabAt(tab, i) == f) {
                    // 当前桶下是链表，或者是空！
                    if (fh >= 0) {
                        // 标识设置为1
                        binCount = 1;
                        // 
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            // 值一样！不是添加，是修改操作    String s = "123";  String s = new String("123");
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                // 获取到当前位置的value值
                                oldVal = e.val;
                                // 是否是IfAbsent，如果为false，不是，覆盖数据，如果为true，什么都不做，break
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            // 追加操作
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                // 如果next指向的是null，就直接插入到后面
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    // 说明桶下是棵树！
                    else if (f instanceof TreeBin) {
                        // 数据插入到红黑树
                    }
                }
            }
            if (binCount != 0) {
                // binCount是否大于等于8，如果大于等于8需要判断是否需要将链表转为红黑树
                if (binCount >= TREEIFY_THRESHOLD)
                    // 尝试转红黑树
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    return null;
}
```

### 七、treeifyBin方法

```java
// 转红黑树
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        // 获取数组长度是否小于64，
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            // 尝试扩容
            tryPresize(n << 1);
        // 转红黑树
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            //....
        }
    }
}
```

### 八、tryPresize方法

```java
private final void tryPresize(int size) {
    // -------------------------------------------------------
    // 对扩容数组长度做一波判断，判断是否达到最大值，其次是保证是2的n次幂
    int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY : tableSizeFor(size + (size >>> 1) + 1);


    // -------------------------------------------------------  
    int sc;
    // 拿到sizeCtl，是否大于0
    while ((sc = sizeCtl) >= 0) {
        // 有两种可能，第一种没初始化数组（putAll方法），第二种数组已经初始化（正常捋顺过来的）
        Node<K,V>[] tab = table; int n;
        // 这里是初始化数组的操作
        if (tab == null || (n = tab.length) == 0) {
            n = (sc > c) ? sc : c;
            if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                try {
                    if (table == tab) {
                        Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                        table = nt;
                        sc = n - (n >>> 2);
                    }
                } finally {
                    sizeCtl = sc;
                }
            }
        }
        // -------------------------------------------------------
        // 如果扩容长度，已经小于扩容阈值。（扩容完事了~~~）
        // 数组长度已经大于等于最大长度
        else if (c <= sc || n >= MAXIMUM_CAPACITY)
```
