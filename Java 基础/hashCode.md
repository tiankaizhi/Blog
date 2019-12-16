## 前言

第一次接触 <code><font color="#f52814">hashCode()</font></code> 的时候是看到了 <code><font color="#f52814">String 类</font></code> 中 <code><font color="#f52814">hashCode()</font></code> 方法，后来知道它是重写了 <code><font color="#f52814">Object 类</font></code> 的 <code><font color="#f52814">hashCode()</font></code> 方法，今天就来聊一聊 <code><font color="#f52814">hashCode()</font></code> 有什么用？

## Hash 是什么

<code><font color="#f52814">Hash</font></code>，一般翻译做 **散列**，就是把任意长度的输入，通过散列算法，变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，所以不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说 <code><font color="#f52814">Hash</font></code> 就是一种将任意长度的消息压缩到某一固定长度的的函数。

### HachCode() 方法的作用

<code><font color="#f52814">hashCode()</font></code> 方法的主要作用是为了 **配合基于散列的数据结构的存储容器（这里的存储容器并不一定指的集合，String 类也可以理解为一种存储容器）一起正常运行**，这样的容器有 <code><font color="#f52814">HashMap</font></code>，<code><font color="#f52814">HashSet</font></code>，<code><font color="#f52814">HashTable</font></code>。为什么说它存在的意义是为了配合这些容器的正常运转呢？

假设现在有一种场景，向 <code><font color="#f52814">HashSet</font></code> 中插入对象（很明显，<code><font color="#f52814">HashSet</font></code> 中是不允许有重复的元素出现的），然而我现在集合里面已经有 1 亿个数据了，那我第  1 亿零 1 个插入的时候按道理是要使用 <code><font color="#f52814">equals()</font></code> 方法和里面的 1 亿个数据进行逐一比较，你觉得合适吗？而 <code><font color="#f52814">hashCode()</font></code> 就是解决这一个问题的。当向集合中添加新的对象的时候，先调用这个对象的 <code><font color="#f52814">hashCode()</font></code> 方法，得到这个对象对应的 <code><font color="#f52814">hashCode 值</font></code>。例如：在 <code><font color="#f52814">HashMap</font></code> 里面它就是通过 <code><font color="#f52814">table</font></code> 中的 <code><font color="#f52814">Entry</font></code> 中 <code><font color="#f52814">hash</font></code> 字段来保存每一个 <code><font color="#f52814">Entry</font></code> 的 <code><font color="#f52814">key</font></code> 的 <code><font color="#f52814">hashCode 值</font></code>，这个 <code><font color="#f52814">hashCode 值</font></code> 就作为每一个 <code><font color="#f52814">Entry</font></code> 的 <code><font color="#f52814">hashCode 值</font></code>。

来看一下 <code><font color="#f52814">HashMap</font></code>

```java
public V put(K key, V value) {
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);
    }
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);  　// 首先，对 key 对象进行 hash 算法，得到一个 hash 值（这个 hash 算法要尽可能的做到每个 key 的 hash 值不一样）
    int i = indexFor(hash, table.length);  // 然后根据 key 的 hash 值再结合 table 的长度算出来这个 Entry 该存放的索引
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {  // 根据上面的索引，遍历这个位置 Entry 链表
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) { // 这里它并没有拿到这 Entry 链表之后对 key 直接使用 equals() 方法进行比较，而是如果发现了 hash
            V oldValue = e.value;　　　　　　　　　　　　　　　　　　　　　　　// 值相等之后再对 key 进行 equals() 进行比较。上面已经说明了，hashMap 集合会把每一个 key 的
            e.value = value;　　　　　　　　　　　　　　　　　　　　　　　　　　// hash 值存储到 每一个 Entry 中，到时候再从 Entry 里面取出来进行比较
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);
    return null;
}
```
```java
public boolean equals(Object obj) {
　　return (this == obj);
}
```

上面的 <code><font color="#f52814">equals()</font></code> 方法如果你存储的对象没有重写 equals 方法就会调用 Object 中的，Object 中的 equals() 方法比较的是两个引用的值是否相等，也就是两个引用所代表的对象在堆内存中的地址是否相等，如果相等，那么这两个对象一定是同一个 对象。这个案例并不明显，如果你存储的对象重写了 equals() 方法，比如你重写的 equals() 方法中比较的是两个对象的每一个字段值是否相等；那么不使用 hashCode()，直接使用 equals() 方法比较就太浪费时间了。

那么重写一个对象的 hashCode() 方法有什么讲究的地方呢？我们先来看看默认的 Object 中的 hashCode() 方法是怎么实现的？


























































































1
