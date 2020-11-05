# HashMap



## jdk7

数据结构：数组+链表

链表使用头插法

多线程下会出现死锁

```java
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    for (Entry<K,V> e : table) {
        while(null != e) {
            //多线程扩容会出现死锁的情况
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

**主要逻辑如下：**

1. 对索引数组中的元素遍历
2. 对链表上的每一个节点遍历：用 next 取得要转移那个元素的下一个，将 e 转移到新 Hash 表的头部，使用头插法插入节点。
3. 循环直到链表节点全部转移
4. 循环直到所有索引数组全部转移

遍历旧数组，将旧数组元素通过**头插法**的方式，迁移到新数组的。经过这几步会发现转移的时候是逆序的，A->B->C迁移后会变成C->B->A，HashMap 的死锁问题就出在这个transfer()函数上。

## 例子

我们假设有二个线程T1、T2，HashMap容量为2，T1线程放入key A、B、C、D、E。

在T1线程中A、B、C Hash值相同，于是形成一个链接，假设为A->B->C，而D、E Hash值不同，于是容量不足，需要新建一个更大尺寸的hash表，然后把数据从老的Hash表中迁移到新的Hash表中(refresh)。

这时T2线程进来了，T1执行到代码Entry<K,V> next = e.next线程A挂起，此刻e**指向A，next指向B**。

T2线程也准备放入新的key，这时也发现容量不足进行refresh。T2开始执行transfer函数中的while循环，会把原来的table变成一个table（线程T2自己的栈中），再写入到内存中。因为转移的时候是逆序的，refresh之后原来的链表结构假设为C->B->A。

T1继续执行执行完一个while循环后，e=B，T1的链表是A。

继续执行T1，此时e=B且B->A,所以next=A，执行一个while循环后，e=A，T1的链表是B->A。

继续执行T1，此时e=A，所以next=null，执行一个while循环后，e=null,线程A的链表是A->B->A

这时就形成A.next=B，B.next=A的环形链表，一旦取值进入这个环形链表就会陷入死循环。

## JDK8

数组+链表或红黑树

树化阈值：等于8

取消树化阈值：等于6

避免链表和树的频繁切换

## Jdk7与jdk8的区别

- jdk8中会将链表会转变为红黑树

- 新节点插入链表的顺序不相同（jdk7是插入头结点，jdk8因为要遍历链表把链表变为红黑树所以采用插入尾结点）
- hash算法简化
- resize的逻辑修改（jdk7会出现死循环，jdk8不会）









[美团HashMap码分析](https://tech.meituan.com/2016/06/24/java-hashmap.html)