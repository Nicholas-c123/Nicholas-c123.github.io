---
layout: post
title:  "Java ReentrantReadWriteLock和CopyOnwriteArrayList的区别"
date:   2017-04-01 15:26:12 +0800
categories: 多线程
excerpt: 读写锁ReadWriteLock和CopyOnWriteArrayList的区别辨析
---

最近复习了一下这两种机制的区别。一个是并发容器，另一个是锁，那我为什么还要找区别呢。  
咳咳，是这样的，大概CopyOnWriteArrayList是一种并发容器，主要运用于读多写少的场景。例如，我们做一个网上商城，搜索栏需要屏蔽一些不和谐的关键字。通常某些不和谐的关键字 **你懂得** 又是搜索频率非常多的，所以我们不应该把这样的数据存放在数据库中。而CopyOnWriteArrayList 和CopyOnWriteSet可以提供一个高效的并发读的容器。但是呢，这里有个小陷阱，摘抄JDK注释如下。
```
It is best suited for applications in which set sizes generally stay small, read-only operations vastly outnumber mutative operations, and you need to prevent interference among threads during traversal.
It is thread-safe.
Mutative operations (add, set, remove, etc.) are expensive since they usually entail copying the entire underlying array.
Iterators do not support the mutative remove operation.
Traversal via iterators is fast and cannot encounter interference from other threads. Iterators rely on unchanging snapshots of the array at the time the iterators were constructed.
```
注意 Iterators rely on unchanging snapshots of the array at the time the iterators were constructed.翻译一下，迭代器是基于迭代器创建的时候底层数组的快照的。换句话说，如果有一个另外的线程在并发修改这个容器的时候，当我们调用contains的时候，底层的数组也还是在原来的快照版本中搜索。也就是数据的更新不是实时的。对于一个商城黑名单业务，来说，我们把黑名单的更新放在夜深人静的时候即可，能不能实时地屏蔽黑名单的关键字似乎也无所谓。
###但是如果有需要数据实时更新，但是又读多写少的情况呢
That's where the concurrent ReentrantReadWriteLock comes into play.
稍微解释一下ReentrantReadWriteLock，可重入读写锁，其基本语义和数据库中的读写锁是一致的。
也就是其实现了读写互斥，写写互斥和读读不互斥，多个线程可以同时获取读锁，但是要写的话，必须要所有读锁都必须先释放。因此，其实现了数据的实时更新和多读少写情景下的效率优化，其底层也是使用了AQS的实现。
最后还是摘抄JDK给的代码片段。嘿嘿，大家感受下读写锁的使用。
```java
//Sample usages. Here is a code sketch showing how to perform lock downgrading after updating a cache (exception handling is particularly tricky when handling multiple locks in a non-nested fashion):
   class CachedData {
    Object data;
    volatile boolean cacheValid;
    final ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
 
    void processCachedData() {
      rwl.readLock().lock();
      if (!cacheValid) {
        // Must release read lock before acquiring write lock
        rwl.readLock().unlock();
        rwl.writeLock().lock();
        try {
          // Recheck state because another thread might have
          // acquired write lock and changed state before we did.
          if (!cacheValid) {
            data = ...
            cacheValid = true;
          }
          // Downgrade by acquiring read lock before releasing write lock
          rwl.readLock().lock();
        } finally {
          rwl.writeLock().unlock(); // Unlock write, still hold read
        }
      }
 
      try {
        use(data);
      } finally {
        rwl.readLock().unlock();
      }
    }
  }
```