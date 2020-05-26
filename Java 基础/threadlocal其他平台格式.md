## 写在前面

ThreadLocal 基本用法本文就不介绍了，如果有不知道的小伙伴可以先了解一下，本文只研究 ThreadLocal 内存泄漏这一问题。

## ThreadLocal 会发生内存泄漏吗？

先给出结论：**如果你使用不当是有可能发生内存泄露的**

![](https://img2020.cnblogs.com/blog/1326851/202005/1326851-20200520215852096-141968393.png)

<br/>

每个 Thread 里面都有一个 ThreadLocalMap，而 ThreadLocalMap 中真正承载数据的是一个 Entry 数组，Entry 的 Key 是 threadlocal 对象的弱引用。

这里或许有的小伙伴有疑问，Entry 的 Key 是 threadlocal 对象的弱引用，为什么要用弱引用，换成强引用行不行？

首先，我们先了解一下什么是弱引用？

> 弱引用一般是用来描述非必需对象的，被弱引用关联的对象只能生存到下一次垃圾收集发生之前。**当垃圾收集器工作时，无论当前内存是否足够，都会回收掉只被弱引用关联的对象**。

实际开发中，当我们不需要 threadlocal 后，为了 GC 将 threadlocal 变量置为 null，没有任何强引用指向堆中的 threadlocal 对象时，堆中的 threadlocal 对象将会被 GC 回收，假设现在 Key 持有的是 threadlocal 对象的强引用，如果当前线程仍然在运行，那么从当前线程一直到 threadlocal 对象还是存在强引用，由于当前线程仍在运行的原因导致 threadlocal 对象无法被 GC，这就发生了内存泄漏。相反，弱引用就不存在此问题，当栈中的 threadlocal 变量置为 null 后，堆中的 threadlocal 对象只有一个 Key 的弱引用关联，下一次 GC 的时候堆中的 threadlocal 对象就会被回收，**使用弱引用对于 threadlocal 对象而言是不会发生内存泄漏的**。

那么，第二个问题来了，是不是 Key 持有的是 threadlocal 对象的弱引用就一定不会发生内存泄漏呢？

结论是：**如果你使用不当还是有可能发生内存泄露**，但是，这里发生内存泄漏的地方和上面不同。

当 threadlocal 使用完后，将栈中的 threadlocal 变量置为 null，threadlocal 对象下一次 GC 会被回收，那么 Entry 中的与之关联的弱引用 key 就会变成 null，如果此时当前线程还在运行，那么 Entry 中的 key 为 null 的 Value 对象并不会被回收（存在强引用），这就发生了内存泄漏，当然这种内存泄漏分情况，如果当前线程执行完毕会被回收，那么 Value 自然也会被回收，但是如果使用的是线程池呢，线程跑完任务以后放回线程池（线程没有销毁，不会被回收），Value 会一直存在，这就发生了内存泄漏。

## 如何更好的降低内存泄漏的风险呢？

ThreadLocal 为了降低内存泄露的可能性，**在 set，get，remove 的时候都会清除此线程 ThreadLocalMap 里 Entry 数组中所有 Key 为 null 的 Value**。所以，当前线程使用完 threadlocal 后，我们可以通过调用 ThreadLocal 的 remove 方法进行清除从而降低内存泄漏的风险。

## 最后

如果大家想要实时关注我更新的文章以及我分享的干货的话，可以关注我的公众号 **我们都是小白鼠**。

![](https://img2020.cnblogs.com/blog/1326851/202003/1326851-20200307235900287-613114059.png)
