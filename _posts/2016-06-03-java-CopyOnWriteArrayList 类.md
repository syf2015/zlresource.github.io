﻿---
layout: post
title:  "java并发容器源码分析-CopyOnWriteArrayList 类"
date:  2016-06-03
categories: JAVA
---

java并发容器源码分析-CopyOnWriteArrayList 类

---

- 目录
{:toc}

#### concurrent中并发容器优点

JDK5中添加了新的concurrent包，其中包含了很多并发容器，这些容器针对多线程环境进行了优化，大大提高了容器类在并发环境下的执行效率。

一、CopyOnWriteArrayList/CopyOnWriteArraySet
    JDK中并没有提供CopyOnWriteMap，我们可以参考CopyOnWriteArrayList来实现一个
二、BlockingQueue
三、ConcurrentHashMap
  
####  一、CopyOnWriteArrayList 类

CopyOnWrite容器即写时复制的容器.当我们往一个容器添加元素的时候，不直接往当前容器添加，而是先将当前容器进行Copy，
复制出一个新的容器，然后新的容器里添加元素，添加完元素之后，再将原容器的引用指向新的容器。
这样做的好处是我们可以对CopyOnWrite容器进行并发的读，而不需要加锁，因为当前容器不会添加任何元素。
    
适用场合：读操作远远多于写操作的应用非常适合。比如白名单，黑名单，少量更新的场景
    
缺点：① 数据一致性问题：只能保证数据的最终一致性，不能保证数据的实时一致性
      ② 内存占用问题：写时复制机制，所以在进行写操作时，会有旧，新两个对象
      ③ 过多的写操作复制会相当耗时
    
##### 1.继承了List接口

```java
      public class CopyOnWriteArrayList<E>  
          implements List<E>, RandomAccess, Cloneable, java.io.Serializable { 
        ...
      }
```

#####  2.读操作加锁，写操作不加锁

写操作：写操作类实现中对其进行了加锁（以免多个线程写时复制出了多份）
   
读操作:读的时候不需要加锁，如果读的时候有多个线程正在向CopyOnWriteArrayList添加数据，
       读还是会读到旧的数据，因为写的时候不会锁住旧的CopyOnWriteArrayList。
      
//读：get无需加锁

```java
      private E get(Object[] a, int index) {
        return (E) a[index];
      }

      //写：set操作是同步加锁的ReentrantLock


      public E set(int index, E element) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            E oldValue = get(elements, index);

            if (oldValue != element) {
                int len = elements.length;
                // [1] 创建一个新数组，将原array的值拷贝至新数组  
                Object[] newElements = Arrays.copyOf(elements, len);
                // [2] set的元素  
                newElements[index] = element;
                // [3] 替换底层array数组  
                setArray(newElements);
            } else {
                // Not quite a no-op; ensures volatile write semantics
                setArray(elements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
 ```
    
##### 3.add和remove操作，修改操作均复制数组

```java
      public boolean add(E e) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            // [1] 创建一个新数组，将原array的值拷贝至新数组  
            Object[] newElements = Arrays.copyOf(elements, len + 1);
            newElements[len] = e;
            setArray(newElements);
            return true;
        } finally {
            lock.unlock();
        }
    }
    public E remove(int index) {
        final ReentrantLock lock = this.lock;
        lock.lock();
        try {
            Object[] elements = getArray();
            int len = elements.length;
            E oldValue = get(elements, index);
            int numMoved = len - index - 1;
            if (numMoved == 0)
                setArray(Arrays.copyOf(elements, len - 1));
            else {
                Object[] newElements = new Object[len - 1];
                System.arraycopy(elements, 0, newElements, 0, index);
                System.arraycopy(elements, index + 1, newElements, index,
                                 numMoved);
                setArray(newElements);
            }
            return oldValue;
        } finally {
            lock.unlock();
        }
    }
  ``` 
  
