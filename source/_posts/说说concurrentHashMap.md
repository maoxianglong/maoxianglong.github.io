---
title: concurrentHashMap源码阅读记录
date: 2017-07-28
tags: [java, 源码 , 集合 ]
categories: [java , 源码]
---

## 一、导论
这些天一直在看关于多线程和高并发的书籍，也对jdk中的并发措施了解了些许，看到concurrentHashMap的时候感觉知识点很乱，有必要写篇博客整理记录一下。

当资源在多线程下共享时会产生一些逻辑问题，这个时候类或者方法会产生不符合正常逻辑的结果，则不是线程安全的。纵观jdk的版本更新，可以看到jdk的开发人员在高并发和多线程下了很大的功夫，尽可能的通过jdk原生API来给开发人员带来最方便最轻松的高并发数据模型，甚至想完全为开发人员解决并发问题，可以看得出来jdk的开发人员确实很用心。但是在大量业务数据的逻辑代码的情况下高并发还是不可避免，也不可能完全通过jdk原生的并发API去解决这些并发问题，开发人员不得不自己去空值在高并发环境下的数据高可用性和一致性。

前面说了jdk原生的API已经有了很多的高并发产品，在java.util.concurrent包下有很多解决高并发，高吞吐量，多线程问题的API。比如线程池ThreadPoolExecutor，线程池工厂Executors，Future模式下的接口Future，阻塞队列BlockingQueue等等。

## 二、正文
1、数据的可见性

直接进入正题，concurrentHashMap相信用的人也很多，因为在数据安全性上确实比HashMap好用，在性能上比hashtable也好用。大家都知道线程在操作一个变量的时候，比如i++，jvm执行的时候需要经过两个内存，主内存和工作内存。那么在线程A对i进行加1的时候，它需要去主内存拿到变量值，这个时候工作内存中便有了一个变量数据的副本，执行完这些之后，再去对变量真正的加1，但是此时线程B也要操作变量，并且逻辑上也是没有维护多线程访问的限制，则很有可能在线程A在从主内存获取数据并在修改的时候线程B去主内存拿数据，但是这个时候主内存的数据还没有更新，A线程还没有来得及讲加1后的变量回填到主内存，这个时候变量在这两个线程操作的情况下就会发生逻辑错误。

2、原子性

原子性就是当某一个线程A修改i的值的时候，从取出i到将新的i的值写给i之间线程B不能对i进行任何操作。也就是说保证某个线程对i的操作是原子性的，这样就可以避免数据脏读。

3、volatile的作用

Volatile保证了数据在多线程之间的可见性，每个线程在获取volatile修饰的变量时候都回去主内存获取，所以当线程A修改了被volatile修饰的数据后其他线程看到的一定是修改过后最新的数据，也是因为volatile修饰的变量数据每次都要去主内存获取，在性能上会有些牺牲。

4、措施

HashMap在多线程的场景下是不安全的，hashtable虽然是在数据表上加锁，纵然数据安全了，但是性能方面确实不如HashMap。那么来看看concurrentHashMap是如何解决这些问题的。

concurrentHashMap由多个segment组成，每一个segment都包含了一个HashEntry数组的hashtable， 每一个segment包含了对自己的hashtable的操作，比如get，put，replace等操作(这些操作与HashMap逻辑都是一样的，不同的是concurrentHashMap在执行这些操作的时候加入了重入锁ReentrantLock)，这些操作发生的时候，对自己的hashtable进行锁定。由于每一个segment写操作只锁定自己的hashtable，所以可能存在多个线程同时写的情况，性能无疑好于只有一个hashtable锁定的情况。通俗的讲就是concurrentHashMap由多个hashtable组成。

5、源码

看下concurrentHashMap的remove操作

```
V remove(Object key, int hash, Object value) {
            lock();//重入锁
            try {
                int c = count - 1;
                HashEntry<K,V>[] tab = table;
                int index = hash & (tab.length - 1);
                HashEntry<K,V> first = tab[index];
                HashEntry<K,V> e = first;
                while (e != null && (e.hash != hash || !key.equals(e.key)))
                    e = e.next;

                V oldValue = null;
                if (e != null) {
                    V v = e.value;
                    if (value == null || value.equals(v)) {
                        oldValue = v;
                        // All entries following removed node can stay
                        // in list, but all preceding ones need to be
                        // cloned.
                        ++modCount;
                        HashEntry<K,V> newFirst = e.next;
                        for (HashEntry<K,V> p = first; p != e; p = p.next)
                            newFirst = new HashEntry<K,V>(p.key, p.hash,
                                                          newFirst, p.value);
                        tab[index] = newFirst;
                        count = c; // write-volatile
                    }
                }
                return oldValue;
            } finally {
                unlock();//释放锁
            }
        }
```
Count是被volatile所修饰，保证了count的可见性，避免操作数据的时候产生逻辑错误。segment中的remove操作和HashMap大致一样，HashMap没有lock()和unlock()操作。

看下concurrentHashMap的get源码

```
V get(Object key, int hash) {
            if (count != 0) { // read-volatile
                HashEntry<K,V> e = getFirst(hash);
　　　　　　　　//如果没有找到则直接返回null
                while (e != null) {
                    if (e.hash == hash && key.equals(e.key)) {
　　　　　　　　　　　　//由于没有加锁，在get的过程中，可能会有更新，拿到的key对应的value可能为null，需要单独判断一遍
                        V v = e.value;
　　　　　　　　　　　　//如果value为不为null，则返回获取到的value
                        if (v != null)
                            return v;
                        return readValueUnderLock(e); // recheck
                    }
                    e = e.next;
                }
            }
            return null;
        }

```
关于concurrentHashMap的get的相关说明已经在上面代码中给出了注释，这里就不多说了。

看下concurrentHashMap中的put

public V put(K key, V value) {
        if (value == null)
            throw new NullPointerException();
        int hash = hash(key.hashCode());
        return segmentFor(hash).put(key, hash, value, false);
}
可以看到concurrentHashMap不允许key或者value为null

接下来看下segment的put

```
V put(K key, int hash, V value, boolean onlyIfAbsent) {
            lock();
            try {
                int c = count;
                if (c++ > threshold) // ensure capacity
                    rehash();
                HashEntry<K,V>[] tab = table;
                int index = hash & (tab.length - 1);
                HashEntry<K,V> first = tab[index];
                HashEntry<K,V> e = first;
                while (e != null && (e.hash != hash || !key.equals(e.key)))
                    e = e.next;

                V oldValue;
                if (e != null) {
                    oldValue = e.value;
                    if (!onlyIfAbsent)
                        e.value = value;
                }
                else {
                    oldValue = null;
                    ++modCount;
                    tab[index] = new HashEntry<K,V>(key, hash, first, value);
                    count = c; // write-volatile
                }
                return oldValue;
            } finally {
                unlock();
            }
        }
```
 同样也是加入了重入锁，其他的基本和HashMap逻辑差不多。值得一提的是jdk8中添加的中的putval，这里就不多说了。
## 三、总结
ConcurrentHashmap将数据结构分为了多个Segment，也是使用重入锁来解决高并发，讲他分为多个segment是为了减小锁的力度，添加的时候加了锁，索引的时候没有加锁，使用volatile修饰count是为了保持count的可见性，都是jdk为了解决并发和多线程操作的常用手段。