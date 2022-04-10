```java
while (true) {  
 Integer count = map.get("count");  
 if (count == null) {  
 if (map.putIfAbsent("count", 1) == null) break;  
 } else {  
 if (map.replace("count",count,count + 1)) break;  
 }  
}
```

put方法

从put方法开始，发现put方法调用了putVal(key, value, true)

map.putIfAbsent

![](C:\Users\hspcadmin\AppData\Roaming\marktext\images\2022-04-02-20-34-40-image.png)

# spread方法 （散列算法，尽量避免hash冲突）

![](C:\Users\hspcadmin\AppData\Roaming\marktext\images\2022-04-02-20-54-06-image.png)

将key的高16位和低16位进行异或，再

![](C:\Users\hspcadmin\AppData\Roaming\marktext\images\2022-04-02-20-55-15-image.png)





# concurrenthashmap



## 线程安全的方式

CAS + synchronized

CAS：compare ans swap 轻量级锁

```java
final V putVal(K key, V value, boolean onlyIfAbsent) {
        if (key == null || value == null) throw new NullPointerException();
        int hash = spread(key.hashCode());
        int binCount = 0;
        for (Node<K,V>[] tab = table;;) {
            Node<K,V> f; int n, i, fh;
            if (tab == null || (n = tab.length) == 0)
                tab = initTable();
            else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
                if (casTabAt(tab, i, null,
                             new Node<K,V>(hash, key, value, null)))
                    break;                   // no lock when adding to empty bin
            }
            else if ((fh = f.hash) == MOVED)
                tab = helpTransfer(tab, f);
            else {
                V oldVal = null;
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        if (fh >= 0) {
                            binCount = 1;
                            for (Node<K,V> e = f;; ++binCount) {
                                K ek;
                                if (e.hash == hash &&
                                    ((ek = e.key) == key ||
                                     (ek != null && key.equals(ek)))) {
                                    oldVal = e.val;
                                    if (!onlyIfAbsent)
                                        e.val = value;
                                    break;
                                }
                                Node<K,V> pred = e;
                                if ((e = e.next) == null) {
                                    pred.next = new Node<K,V>(hash, key,
                                                              value, null);
                                    break;
                                }
                            }
                        }
                        else if (f instanceof TreeBin) {
                            Node<K,V> p;
                            binCount = 2;
                            if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                           value)) != null) {
                                oldVal = p.val;
                                if (!onlyIfAbsent)
                                    p.val = value;
                            }
                        }
                    }
                }
                if (binCount != 0) {
                    if (binCount >= TREEIFY_THRESHOLD)
                        treeifyBin(tab, i);
                    if (oldVal != null)
                        return oldVal;
                    break;
                }
            }
        }
        addCount(1L, binCount);
        return null;
    }
```



## 扩容触发条件

+ 链表长度达到8，但是数组长度没有达到64 扩容

+ 数组上的数据，已经存放了75%，达到了阈值，扩容

+ 调用putAll方法，查询的Map比原数组长，直接扩容

## 扩容戳

ConcurrentHashMap，为了保证扩容时线程安全，并且保证多线程可以协助扩容

为每一次扩容生成一个扩容戳

扩容戳基于老数组长度生成，记录了指定长度的唯一标识，以及一起扩容的线程有多少个





扩容源码



```java
/**
     * Tries to presize table to accommodate the given number of elements.
     *
     * @param size number of elements (doesn't need to be perfectly accurate)
   
  *///
    //sizeCtl时 一个标识
    //sizeCtl = -1; 代表 Map正在初始化
    //sizeCtl < -1; 代表Map正在扩容
    //sizeCtl = 0; 代表还没有初始化
    //sizeCtl > 0; 代表已经初始化，数值代表扩容阈值，如果没有初始化，代表初始化的数组长度
   负载因子 默认 0.75 
    private final void tryPresize(int size) {
        int c = (size >= (MAXIMUM_CAPACITY >>> 1)) ? MAXIMUM_CAPACITY :
            tableSizeFor(size + (size >>> 1) + 1);
        //声明sc,将sizeCtl复制进来
        int sc;
        //sc 》=0 现在没扩容
        while ((sc = sizeCtl) >= 0) {
            Node<K,V>[] tab = table; int n;
            if (tab == null || (n = tab.length) == 0) {
                n = (sc > c) ? sc : c;
                if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
                    try {
                        if (table == tab) {
                            @SuppressWarnings("unchecked")
                            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                            table = nt;
                            sc = n - (n >>> 2);
                        }
                    } finally {
                        sizeCtl = sc;
                    }
                }
            }
            else if (c <= sc || n >= MAXIMUM_CAPACITY)
                break;
            else if (tab == table) {
                //开始扩容,生成扩容戳，一个数字
                int rs = resizeStamp(n);
                if (sc < 0) {
                    Node<K,V>[] nt;
                    // Doug lea的BUG
                    if ((sc >>> RESIZE_STAMP_SHIFT) != rs || 
                        sc == rs + 1 ||    //问题代码
                        sc == rs + MAX_RESIZERS ||    //问题代码
                         (nt = nextTable) == null ||
                        transferIndex <= 0)
                        break;
                    if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                        transfer(tab, nt);
                }
                else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                             (rs << RESIZE_STAMP_SHIFT) + 2))
                    transfer(tab, null);
            }
        }
    }


```

```java
 /**
     * Returns the stamp bits for resizing a table of size n.
     * Must be negative when shifted left by RESIZE_STAMP_SHIFT.
     */
 //生成扩容戳,比如n= 32
    static final int resizeStamp(int 
 n) {
         //获取n这个变量在二进制标识时，前面为0的个数
         // 26 | ((1 << 15)  == )
        return Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1));
    }
 0000000 0000000 0000000 0010000
 
 0000000 0000000 0000000 0000001
 
 0000000 0000000 0000000 0011010 = 26
 0000000 0000000 1000000 0000000 = 1 <<15  （保证最高位为1，1为符号位，所以为负数，这样就可以表示为正在扩容的状态）
=0000000 0000000 1000000 0011010 = 扩容戳（半成品） rs
 
 rs << RESIZE_STAMP_SHIFT) + 2
 0000000 0000000 1000000 0011010 = 扩容戳（半成品） rs
 1000000 0011010 0000000 0000000 = 扩容戳（完全体），低位表示正在扩容的线程个数 -1
 高16位是扩容的唯一标识，（基于原数组长度算的扩容表示）
 低16位表示扩容的线程个数
  
```

sc = sizeCtl

tab = 老数组

n =  老数组长度





## 六、扩容源码分析

```java
/**
     * Moves and/or copies the nodes in each bin to new table. See
     * above for explanation.
     */
    private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {            
     //计算扩容的间隔
     int n = tab.length, stride;
        if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
            stride = MIN_TRANSFER_STRIDE; // subdivide range
        //如果基于长度和CPU内核运算，得到的结果比16大，那么每次迁移数据的长度就用计算的
        //否则使用最少迁移数据的长度 16
        
        //如果nextTab == null 代表我是第一个进来扩容的线程
        //初始化新数组
        if (nextTab == null) {            // initiating
            try {
                @SuppressWarnings("unchecked")
                Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
                nextTab = nt;
            } catch (Throwable ex) {      // try to cope with OOME
                sizeCtl = Integer.MAX_VALUE;
                return;
            }
            nextTable = nextTab;
            transferIndex = n;
        }
        //获取新数组长度
        int nextn = nextTab.length;
        // 在桶迁移完数组后，放置的节点，代表当前位置已经迁移数据完毕
        fwd的hash值位-1，代表当前桶迁移数据完毕
        ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
        //adwance 代表当前线程还有活干么？
        boolean advance = true;
       // finishing 代表扩容完事了么？
        boolean finishing = false; // to ensure sweep before committing nextTab
            //死循环，i代表桶的上限值， bound代表下限值
            for (int i = 0, bound = 0;;) {
            Node<K,V> f; int fh;
            while (advance) {
                int nextIndex, nextBound;
                if (--i >= bound || finishing)
                    advance = false;
                //
                else if ((nextIndex = transferIndex) <= 0) {
                    i = -1;
                    advance = false;
                }
                //
                else if (U.compareAndSwapInt
                         (this, TRANSFERINDEX, nextIndex,
                          nextBound = (nextIndex > stride ?
                                       nextIndex - stride : 0))) {
                    bound = nextBound;
                    i = nextIndex - 1;
                    advance = false;
                }
            }
             //搬运数据
            if (i < 0 || i >= n || i + n >= nextn) {
                int sc;
                if (finishing) {
                    nextTable = null;
                    table = nextTab;
                    sizeCtl = (n << 1) - (n >>> 1);
                    return;
                }
                if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                    if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                        return;
                    finishing = advance = true;
                    i = n; // recheck before commit
                }
            }
            //获取i位置数据，如果为null,设置设置为fwd
            else if ((f = tabAt(tab, i)) == null)
                advance = casTabAt(tab, i, null, fwd);
            //如果当前位置的hash值为MOVED,说明是fwd节点，搬完了！
            else if ((fh = f.hash) == MOVED)
                advance = true; // already processed
            else {
                //有数据，加锁！
                synchronized (f) {
                    if (tabAt(tab, i) == f) {
                        Node<K,V> ln, hn;
                        //代表是树，fh = hash值
                        if (fh >= 0) {
                            int runBit = fh & n;
                            Node<K,V> lastRun = f;
                            for (Node<K,V> p = f.next; p != null; p = p.next) {
                                int b = p.hash & n;
                                if (b != runBit) {
                                    runBit = b;
                                    lastRun = p;
                                }
                            }
                            if (runBit == 0) {
                                ln = lastRun;
                                hn = null;
                            }
                            else {
                                hn = lastRun;
                                ln = null;
                            }
                            for (Node<K,V> p = f; p != lastRun; p = p.next) {
                                int ph = p.hash; K pk = p.key; V pv = p.val;
                                if ((ph & n) == 0)
                                    ln = new Node<K,V>(ph, pk, pv, ln);
                                else
                                    hn = new Node<K,V>(ph, pk, pv, hn);
                            }
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                        else if (f instanceof TreeBin) {
                            TreeBin<K,V> t = (TreeBin<K,V>)f;
                            TreeNode<K,V> lo = null, loTail = null;
                            TreeNode<K,V> hi = null, hiTail = null;
                            int lc = 0, hc = 0;
                            for (Node<K,V> e = t.first; e != null; e = e.next) {
                                int h = e.hash;
                                TreeNode<K,V> p = new TreeNode<K,V>
                                    (h, e.key, e.val, null, null);
                                if ((h & n) == 0) {
                                    if ((p.prev = loTail) == null)
                                        lo = p;
                                    else
                                        loTail.next = p;
                                    loTail = p;
                                    ++lc;
                                }
                                else {
                                    if ((p.prev = hiTail) == null)
                                        hi = p;
                                    else
                                        hiTail.next = p;
                                    hiTail = p;
                                    ++hc;
                                }
                            }
                            ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                                (hc != 0) ? new TreeBin<K,V>(lo) : t;
                            hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                                (lc != 0) ? new TreeBin<K,V>(hi) : t;
                            setTabAt(nextTab, i, ln);
                            setTabAt(nextTab, i + n, hn);
                            setTabAt(tab, i, fwd);
                            advance = true;
                        }
                    }
                }
            }
        }
    }


```



```java
n = 32
stride = 16 间隔
nextTab、nextTable  = 新数组
transferIndex = n = 32
nextn = 64
i = 0
bound = 0
f = ?
fn = ?
n
```
