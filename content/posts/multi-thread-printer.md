---
title: "两个线程交替打印1-100的数字"
date: 2019-09-04T00:25:05+08:00
---

### 分析&解决 

一道经典的面试题,考察点是Java的notify/wait机制. 看过网上的一些解法,大同小异,不过很多都存在一个问题:输出完成后,会有一个线程由于wait后无法被唤醒导致程序一直阻塞. 针对这一个做了些优化.wait时会先判断一下,如果后面本线程不会再输出就不再wait.代码如下:

```java
public class MultiThreadPrinter {
    private int num;
    private final int endInclude;
    private final Object lock = new Object();

    public MultiThreadPrinter(int num, int endInclude) {
        this.num = num;
        this.endInclude = endInclude;
    }

    public void print() {
        while (num <= endInclude) {
            synchronized (lock) {
                System.out.println(Thread.currentThread().getName() + " : " + num++);
                lock.notify();
                if (num <= endInclude) {
                    try {
                        lock.wait();
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
    }


    public static void main(String[] args) {
        MultiThreadPrinter printer = new MultiThreadPrinter(0, 100);
        new Thread(printer::print, "even").start();
        new Thread(printer::print, "odd").start();
    }

}

```

Java的notify/wait就不再赘述,这里提一下很可能被忽视的知识点.

#### num不需要使用volatile修饰

看过的blog中,很多都对num使用volatile进行修饰,期望以此保证其读写的可见性. 其实在这个题目中,没有意义!

当线程读写一个主存(堆)中的变量时,会先将这个变量复制到该线程的本地内存中, 后续的读写都是对本地内存中的这个变量进行操作,并且在**恰当的时候**刷写回主存.

而synchronize即锁,有如下内存语义:

- 获取到锁时,会将本地变量置为无效,
- 释放锁时,会将本地变量刷写回主存.

所以当进入synchronized时会将本地内存中的num置为无效,再次从主存中获取到的就是最新的值.退出synchronized时会将本地内存中的num刷写回主存. 而volatile读/写和获取/释放锁的内存语义相同,所以对于在synchronize块内读写的变量,使用volatile没有意义.



### 调用Object.notify()后并不会立刻唤醒其他等待线程

调用notify后,在退出synchronized块时才会唤醒其他线程.如果只看代码线性执行,可能有点不好理解.但是想一下synchronized块的作用,就很好理解了.