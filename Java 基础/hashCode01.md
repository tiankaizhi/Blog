
## 前言

第一次接触 hashCode() 的时候是看到了 String 字符串类中 hashCode() 方法，后来知道它是重写了 Object 类的 hashCode() 方法，今天就来聊一聊 hashCode 有什么用？

## 什么是 Hash

　　Hash，一般翻译做 “散列”，就是把任意长度的输入，通过散列算法，变换成固定长度的输出，该输出就是散列值。这种转换是一种压缩映射，也就是，散列值的空间通常远小于输入的空间，所以不同的输入可能会散列成相同的输出，所以不可能从散列值来确定唯一的输入值。简单的说 Hash 就是一种将任意长度的消息压缩到某一固定长度的的函数。

### HashCode() 方法的作用是什么？

　　hashCode() 方法的主要作用是为了配合基于散列的数据结构的存储容器（这里的存储容器并不一定指的集合，String 类也可以理解为一种存储容器）一起正常运行，这样的容器有 hashMap ，hashSet，hashTable。为什么说它存在的意义是为了配合这些容器的正常运转呢？

　　假设现在有一种场景，向 hashSet 中插入对象（很明显，hashSet 中是不允许有重复的元素出现的），然而我现在集合里面已经有 1 亿个数据了，那我第  1 亿零 1 个插入的时候按道理是要使用 equals() 方法和里面的 1 亿个数据进行逐一比较，你觉得合适吗？而 hashCode() 就是解决这一个问题的。当向集合中添加新的对象的时候，先调用这个对象的 hashCode() 方法，得到这个对象对应的 hashCode 值。例如：在 hashMap 里面它就是通过 table 中的 Entry 中 hash 字段来保存每一个 Entry 的 key 的 hashCode 值，这个 hashCode 值就作为每一个 Entry 的 hashCode 值。

来看一下 hashMap

 1 public V put(K key, V value) {
 2         if (table == EMPTY_TABLE) {
 3             inflateTable(threshold);
 4         }
 5         if (key == null)
 6             return putForNullKey(value);
 7         int hash = hash(key);  　// 首先，对 key 对象进行 hash 算法，得到一个 hash 值（这个 hash 算法要尽可能的做到每个 key 的 hash 值不一样）
 8         int i = indexFor(hash, table.length);  // 然后根据 key 的 hash 值再结合 table 的长度算出来这个 Entry 该存放的索引
 9         for (Entry<K,V> e = table[i]; e != null; e = e.next) {  // 根据上面的索引，遍历这个位置 Entry 链表
10             Object k;
11             if (e.hash == hash && ((k = e.key) == key || key.equals(k))) { // 这里它并没有拿到这 Entry 链表之后对 key 直接使用 equals() 方法进行比较，而是如果发现了 hash
12                 V oldValue = e.value;　　　　　　　　　　　　　　　　　　　　　　　// 值相等之后再对 key 进行 equals() 进行比较。上面已经说明了，hashMap 集合会把每一个 key 的
13                 e.value = value;　　　　　　　　　　　　　　　　　　　　　　　　　　// hash 值存储到 每一个 Entry 中，到时候再从 Entry 里面取出来进行比较
14                 e.recordAccess(this);
15                 return oldValue;
16             }
17         }
18
19         modCount++;
20         addEntry(hash, key, value, i);
21         return null;
22     }
public boolean equals(Object obj) {
　　return (this == obj);
}
上面的 equals 方法如果你存储的对象没有重写 equals 方法就会调用 Object 中的，Object 中的 equals() 方法比较的是两个引用的值是否相等，也就是两个引用所代表的对象在堆内存中的地址是否相等，如果相等，那么这两个对象一定是同一个 对象。这个案例并不明显，如果你存储的对象重写了 equals() 方法，比如你重写的 equals() 方法中比较的是两个对象的每一个字段值是否相等；那么不使用 hashCode()，直接使用 equals() 方法比较就太浪费时间了。

那么重写一个对象的 hashCode() 方法有什么讲究的地方呢？我们先来看看默认的 Object 中的 hashCode() 方法是怎么实现的？

static inline intptr_t get_next_hash(Thread * Self, oop obj) {
  intptr_t value = 0 ;
  if (hashCode == 0) {
     // This form uses an unguarded global Park-Miller RNG,
     // so it's possible for two threads to race and generate the same RNG.
     // On MP system we'll have lots of RW access to a global, so the
     // mechanism induces lots of coherency traffic.
     value = os::random() ;
  } else
  if (hashCode == 1) {
     // This variation has the property of being stable (idempotent)
     // between STW operations.  This can be useful in some of the 1-0
     // synchronization schemes.
     intptr_t addrBits = intptr_t(obj) >> 3 ;
     value = addrBits ^ (addrBits >> 5) ^ GVars.stwRandom ;
  } else
  if (hashCode == 2) {
     value = 1 ;            // for sensitivity testing
  } else
  if (hashCode == 3) {
     value = ++GVars.hcSequence ;
  } else
  if (hashCode == 4) {
     value = intptr_t(obj) ;
  } else {
     // Marsaglia's xor-shift scheme with thread-specific state
     // This is probably the best overall implementation -- we'll
     // likely make this the default in future releases.
     unsigned t = Self->_hashStateX ;
     t ^= (t << 11) ;
     Self->_hashStateX = Self->_hashStateY ;
     Self->_hashStateY = Self->_hashStateZ ;
     Self->_hashStateZ = Self->_hashStateW ;
     unsigned v = Self->_hashStateW ;
     v = (v ^ (v >> 19)) ^ (t ^ (t >> 8)) ;
     Self->_hashStateW = v ;
     value = v ;
  }

  value &= markOopDesc::hash_mask;
  if (value == 0) value = 0xBAD ;
  assert (value != markOopDesc::no_hash, "invariant") ;
  TEVENT (hashCode: GENERATE) ;
  return value;
}
该实现位于 hotspot/src/share/vm/runtime/synchronizer.cpp 文件下，有兴趣的童鞋可以看看。

根据上面的代码可以得出一个结论：hotSpot 默认情况下，hashCode() 返回值并不是有些朋友说的那样就是对象的存储地址（确实有些 JVM 在实现时是直接返回对象的存储地址，但是大多时候并不是这样），只能说可能存储地址有一定关联。

因此有人会说，可以直接根据 hashcode 值判断两个对象是否相等吗？肯定是不可以的，因为不同的对象可能会生成相同的 hashcode 值。虽然不能根据 hashcode 值判断两个对象是否相等，但是可以直接根据 hashcode 值判断两个对象不等，如果两个对象的 hashcode 值不等，则必定是两个不同的对象。如果要判断两个对象是否真正相等，必须通过 equals 方法。也就是说对于两个对象，如果调用 equals() 方法得到的结果为 true，则两个对象的 hashcode 值必定相等；如果equals() 方法得到的结果为 false，则两个对象的 hashcode 值可以相同，可以不同；如果两个对象的 hashcode 值不等，则 equals() 方法得到的结果必定为 false；

为什么一个类在重写 equals() 方法的同时，必须重写 hashCode() 方法？

其实重写 equals 方法的同时，必须重写 hashCode 方法是针对 HashSet 集合而言的

看一下 Demo

public class HashSetTest {
    public static void main(String[] args){

        Person person1 = new Person();
        person1.setName("tkz");
        person1.setAge(23);
        person1.setAddress("tcs");

        Person person2 = new Person();
        person2.setName("tkz");
        person2.setAge(23);
        person2.setAddress("tcs");

        HashSet<Person> set = new HashSet<>();
        System.out.println("person1 的 hashCode 值是："+ person1.hashCode());
        System.out.println("person2 的 hashCode 值是："+ person2.hashCode());

        System.out.println(person1.equals(person2));

        set.add(person1);
        set.add(person2);
        System.out.println(set.size());
    }
}
public class Person {
    private String name;
    private Integer age;
    private String address;

    public String getName() {
        return this.name;
    }

    public Integer getAge() {
        return this.age;
    }

    public String getAddress() {
        return this.address;
    }


    public void setName(String name) {
        this.name = name;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public boolean equals(Object o) {
        if (o == this) return true;
        if (!(o instanceof Person)) return false;
        Person other = (Person) o;
        if (!other.canEqual(this)) return false;
        Object this$name = getName();
        Object other$name = other.getName();
        if (this$name == null ? other$name != null : !this$name.equals(other$name)) return false;
        Object this$age = getAge();
        Object other$age = other.getAge();
        if (this$age == null ? other$age != null : !this$age.equals(other$age)) return false;
        Object this$address = getAddress();
        Object other$address = other.getAddress();
        return this$address == null ? other$address == null : this$address.equals(other$address);
    }
}
打印信息

person1 的 hashCode 值是：1476011703
person2 的 hashCode 值是：1603195447
true
2
上面这个案例就是典型的 Person 类重写了 equals() 方法，但是没有重写 hashCode() 方法，我们可以看到其实 person1 和 person2 这两个对象的属性值是一样的，按照实际运用的角度来说就是两个相同的对象，这个可以通过重写 equals() 方法来实现，但是你没有重写 hashCode() 方法，那就意味着两个相同的对象的 hashCode 值是不同的，那么，对于 HashSet 容器来说就会出问题。

简单看一下源码

public boolean add(E e) {
　　return map.put(e, PRESENT)==null;
}
 1 public V put(K key, V value) {
 2         if (table == EMPTY_TABLE) {
 3             inflateTable(threshold);
 4         }
 5         if (key == null)
 6             return putForNullKey(value);
 7         int hash = hash(key);
 8         int i = indexFor(hash, table.length);
 9         for (Entry<K,V> e = table[i]; e != null; e = e.next) {
10             Object k;
11             if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
12                 V oldValue = e.value;
13                 e.value = value;
14                 e.recordAccess(this);
15                 return oldValue;
16             }
17         }
18
19         modCount++;
20         addEntry(hash, key, value, i);
21         return null;
22     }
上面的第 7 行对一个对象做 hash() 运算，根据下面的源码实际上就是调用这个对象的 hashCode() 方法然后得出的值进行移位运算，算出来的结果进行 indexFor 方法得出最后要存放的数组的下标位置，所以说只要 hashCode 值不同，hashMap 就会保存这两个对象。这就是为什么要重写了 equals() 方法后一定要重写 hashCode() 方法的原因，如果你不重写，那么两个相同属性值的对象（即相等）就会在 hashSet 里面保存两份。

 1 final int hash(Object k) {
 2         int h = hashSeed;
 3         if (0 != h && k instanceof String) {
 4             return sun.misc.Hashing.stringHash32((String) k);
 5         }
 6
 7         h ^= k.hashCode();
 8
 9         // This function ensures that hashCodes that differ only by
10         // constant multiples at each bit position have a bounded
11         // number of collisions (approximately 8 at default load factor).
12         h ^= (h >>> 20) ^ (h >>> 12);
13         return h ^ (h >>> 7) ^ (h >>> 4);
14     }
