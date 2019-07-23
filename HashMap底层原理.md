HashMap底层是采用**数组加链表**的方式来实现的。

图示：

![hashmap数据结构](https://user-gold-cdn.xitu.io/2019/7/23/16c1e2e9f587f2dc?w=512&h=442&f=png&s=51523)

1）当使用put方法时，通过对Key进行Hash函数运算得到一个Hash值。因为得到的Hash值比较大，为了满足能够得到对应的数组下标，将hash值与数组长度进行取余运算，得到Key对应的数组下标值。

先查找出数组位置是否存在对象。存在，则把里面的链表拿出来，判断链表中是否存在key值与传递过来的Key值一样的对象。存在，则把传递过来的value取代链表Key对应的value，并返回旧值；不存在，则直接将该entry对象加到链表里面。

2）当使用get方法，通过对Key进行Hash函数运算得到一个Hash值。因为得到的Hash值比较大，为了满足能够得到对应的数组下标，将hash值与数组长度进行取余运算，得到Key对应的数组下标值。

先查找出数组位置是否存在对象。若不存在，则返回空。若存在，则遍历链表，判断链表中是否存在Key值与传递过来的Key值一样的对象。存在，则把Key值对应的value取出并返回；不存在，则返回空。

3）在JDK7中，进行addEntry方法（在put时，调用的为链表增加节点的方法）时，若当前数组内存储的元素的个数大于等于【size*阀值（0.75）】，且要存放的这个节点在数组中的位置上不为空的时候，进行扩容操作。

扩容操作就是将当前数组容量扩大为原来数组的两倍，并重新计算Hash值与对应的下标，然后将原数组中的数据赋值到扩容后的新数组中。

**4）JDK1.7与JDK1.8的区别：**

①插入链表的顺序不同（JDK7是插入头结点，JDK8因为要遍历链表把链表变成红黑树所以采用插入尾结点的方法）

②链表会转变成红黑树

③hash算法简化

④resize【扩容】的逻辑修改（JDK7会出现死循环(死锁)，JDK8不会）

5）简化Hash算法的原因：

JDK7中保证数组的散列性是因为对链表进行查询操作时还是需要遍历整个链表的。如果整个数组中就只有一个空间被用到了，那么整个HashMap中的元素就都会放置在数组中该位置的链表中，进行遍历链表的效率就降低了。

JDK8中当链表中的元素超过8个时，链表就会变成红黑树，红黑树的遍历效率就比链表要高的多，因为对于Hash算法保证散列性的要求就没有那么高了，因此，hash算法就现对于JDK7简化了。

**JDK8：数组默认长度为16，当链表中的元素超过8个时，链表就会改成红黑树。**

6）插入链表的顺序不同的原因：

JDK8中当链表中元素个数超过8时，就会将链表改成红黑树。要知道当前链表的个数，那就必须要遍历当前链表，所以直接将新结点加到尾部会比将新结点加在头部的操作简单。因此，JDK8中采用插入尾结点的方法插入结点。

手写HashMap的put与get方法：
```java
package com.pdsu.test;

import java.util.Collection;
import java.util.Map;
import java.util.Set;

public class MyHashMap<K,V>{
    private Entry<K,V>[] table;
    private static Integer CAPACITY=8;
    private int size=0;

    public MyHashMap() {
        table=new Entry[CAPACITY];
    }

    class Entry<K,V>{
        private K k;
        private V v;
        private Entry<K,V> next;

        public Entry(K k, V v, Entry<K, V> next) {
            this.k = k;
            this.v = v;
            this.next = next;
        }

        public K getK() {
            return k;
        }

        public void setK(K k) {
            this.k = k;
        }

        public V getV() {
            return v;
        }

        public void setV(V v) {
            this.v = v;
        }

        public Entry<K, V> getNext() {
            return next;
        }

        public void setNext(Entry<K, V> next) {
            this.next = next;
        }
    }
    public int size() {
        return size;
    }

    public boolean isEmpty() {
        return false;
    }

    public boolean containsKey(Object key) {
        return false;
    }

    public boolean containsValue(Object value) {
        return false;
    }

    public V get(Object key) {
        int hash = key.hashCode();
        int index = hash % table.length;
        //get方法先遍历指定数组下标的整个链表，若Key存在则返回Key对应的值；若没有，则返回空
        for(Entry<K,V> entry=table[index];entry!=null;entry=entry.next){
            if(entry.k.equals(key)){
                return entry.v;
            }
        }
        return null;
    }

    public V put(K key, V value) {
//        Entry entry = new Entry(key, value, null);
        //通过计算Key的Hash值，且进行取余操作，得到数组的下标
        int hash = key.hashCode();
        int index = hash % table.length;
        //遍历链表，若当前put进HashMap中的Key与原来的Key一致，则先返回旧的值，在将新值赋值给对应的key
        for(Entry<K,V> entry=table[index];entry!=null;entry=entry.next){
            if(entry.k.equals(key)){
                V oldValue = entry.v;
                entry.v=value;
                return oldValue;
            }
        }
        //若put进HashMap的key不一致，则直接插入数组，若出现hash冲突，则将新的值从链表的头部插入
        addEntry(key, value, index);

        return null;
    }

    private void addEntry(K key, V value, int index) {
        Entry entry = new Entry(key, value, table[index]);
        table[index]=entry;
        size++;
    }

    public V remove(Object key) {
        return null;
    }

    public void putAll(Map<? extends K, ? extends V> m) {

    }

    public void clear() {

    }

    public Set<K> keySet() {
        return null;
    }

    public Collection<V> values() {
        return null;
    }

    public Set<Map.Entry<K,V>> entrySet() {
        return null;
    }

    public static void main(String[] args) {
        MyHashMap<String, String> hashMap = new MyHashMap<>();
        for(int i=0;i<10;i++){
            hashMap.put("周瑜"+i,"既生瑜，何生亮"+i);
        }
        System.out.println(hashMap.get("1"));
    }
}
```
