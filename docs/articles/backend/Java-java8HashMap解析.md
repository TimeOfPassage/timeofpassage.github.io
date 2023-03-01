# Java-java8HashMap解析

## HashMap(非线程安全的)

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
  
}
```

HashMap实现了Map接口，Cloneable接口，Serializable接口，继承了AbstractMap

Serializable接口： 支持jdk自带序列化机制

Map接口：map核心功能接口

Cloneable接口：支持clone，实现实例拷贝[java.lang.Object#clone()]

## Map接口

```java
public interface Map<K,V> {
    int size(); // 获取map大小
    boolean isEmpty(); // 判读map是否为空
    boolean containsKey(Object key); // 判断map是否存在key
    boolean containsValue(Object value);// 判断map是否存在值=value
    V get(Object key);// 根据key返回对应value
    V put(K key, V value);// 增加元素
    V remove(Object key); // 根据key移除元素
    void putAll(Map<? extends K, ? extends V> m); // 批量添加元素
    void clear(); // 清空map
    Set<K> keySet(); // 返回所有key信息
    Collection<V> values();// 返回所有value信息
    Set<Map.Entry<K, V>> entrySet(); // 返回map所有元素信息
    interface Entry<K,V> {
        K getKey();
        V getValue();
        V setValue(V value);
        boolean equals(Object o);
        int hashCode();
        public static <K extends Comparable<? super K>, V> Comparator<Map.Entry<K,V>> comparingByKey() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getKey().compareTo(c2.getKey());
        }
        public static <K, V extends Comparable<? super V>> Comparator<Map.Entry<K,V>> comparingByValue() {
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> c1.getValue().compareTo(c2.getValue());
        }
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByKey(Comparator<? super K> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getKey(), c2.getKey());
        }
        public static <K, V> Comparator<Map.Entry<K, V>> comparingByValue(Comparator<? super V> cmp) {
            Objects.requireNonNull(cmp);
            return (Comparator<Map.Entry<K, V>> & Serializable)
                (c1, c2) -> cmp.compare(c1.getValue(), c2.getValue());
        }
    }
  	// 以下方法用于比较
    boolean equals(Object o);
    int hashCode();
    // Defaultable methods
    default V getOrDefault(Object key, V defaultValue) {
        V v;
        return (((v = get(key)) != null) || containsKey(key))
            ? v
            : defaultValue;
    }
    default void forEach(BiConsumer<? super K, ? super V> action) {
        Objects.requireNonNull(action);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
            action.accept(k, v);
        }
    }
    default void replaceAll(BiFunction<? super K, ? super V, ? extends V> function) {
        Objects.requireNonNull(function);
        for (Map.Entry<K, V> entry : entrySet()) {
            K k;
            V v;
            try {
                k = entry.getKey();
                v = entry.getValue();
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }

            // ise thrown from function is not a cme.
            v = function.apply(k, v);

            try {
                entry.setValue(v);
            } catch(IllegalStateException ise) {
                // this usually means the entry is no longer in the map.
                throw new ConcurrentModificationException(ise);
            }
        }
    }
    default V putIfAbsent(K key, V value) {
        V v = get(key);
        if (v == null) {
            v = put(key, value);
        }

        return v;
    }
    default boolean remove(Object key, Object value) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, value) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        remove(key);
        return true;
    }
    default boolean replace(K key, V oldValue, V newValue) {
        Object curValue = get(key);
        if (!Objects.equals(curValue, oldValue) ||
            (curValue == null && !containsKey(key))) {
            return false;
        }
        put(key, newValue);
        return true;
    }
    default V replace(K key, V value) {
        V curValue;
        if (((curValue = get(key)) != null) || containsKey(key)) {
            curValue = put(key, value);
        }
        return curValue;
    }
    default V computeIfAbsent(K key,
            Function<? super K, ? extends V> mappingFunction) {
        Objects.requireNonNull(mappingFunction);
        V v;
        if ((v = get(key)) == null) {
            V newValue;
            if ((newValue = mappingFunction.apply(key)) != null) {
                put(key, newValue);
                return newValue;
            }
        }

        return v;
    }
    default V computeIfPresent(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue;
        if ((oldValue = get(key)) != null) {
            V newValue = remappingFunction.apply(key, oldValue);
            if (newValue != null) {
                put(key, newValue);
                return newValue;
            } else {
                remove(key);
                return null;
            }
        } else {
            return null;
        }
    }
    default V compute(K key,
            BiFunction<? super K, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        V oldValue = get(key);

        V newValue = remappingFunction.apply(key, oldValue);
        if (newValue == null) {
            // delete mapping
            if (oldValue != null || containsKey(key)) {
                // something to remove
                remove(key);
                return null;
            } else {
                // nothing to do. Leave things as they were.
                return null;
            }
        } else {
            // add or replace old mapping
            put(key, newValue);
            return newValue;
        }
    }
    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }
}
```

## HashMap方法部分

![image-20210717191523396](../../_media/image/20210717191523396-methods.png)



本文主要关注`构造方法`，`put`和`get`方法

### 构造方法

**实现源码(优化过后的、基于jdk1.8)**

```java
public HashMap(int initialCapacity, float loadFactor) {
  // 判断传入的初始容量是否小于0，如果小于则报错提示
  if (initialCapacity < 0) {
    throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
  }
  // 判断传入的初始容量是否大于最大值，如果大于则报错提示
  // static final int MAXIMUM_CAPACITY = 1 << 30; 
  if (initialCapacity > MAXIMUM_CAPACITY){
    initialCapacity = MAXIMUM_CAPACITY;
  }
  // 判断加载因子是否小于等于0，并且加载因子是否是合法数字
  if (loadFactor <= 0 || Float.isNaN(loadFactor)) {
    throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
  }
  this.loadFactor = loadFactor;
  // tableSizeFor的功能（不考虑大于最大容量的情况）是返回大于输入参数且最近的2的整数次幂的数。比如10，则返回16
  this.threshold = tableSizeFor(initialCapacity);
}
```

如果默认使用无参数构造，则加载因子是0.75，默认容量是16；

这里的加载因子设置和数组容量后续扩容有关

### put方法

**实现源码(优化过后的、基于jdk1.8)**

```java
public V put(K key, V value) {
  return putVal(hash(key), key, value, false, true);
}
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
  Node<K,V>[] tab;
  Node<K,V> p;
  int n, i;
  // 表进行扩容并返回表长度
  if ((tab = table) == null || (n = tab.length) == 0) {
    tab = resize();
    n = tab.length;
  }
  i = (n - 1) & hash;
  p = tab[i];
  if (p == null) {
    // 构造新节点加入
    tab[i] = newNode(hash, key, value, null);
  } else {
    Node<K,V> e; K k;
    if (p.hash == hash && ((k = p.key) == key || (key != null && key.equals(k)))) {
      e = p;
    } else if (p instanceof TreeNode) {
      e = ((TreeNode<K, V>) p).putTreeVal(this, tab, hash, key, value);
    } else {
      for (int binCount = 0; ; ++binCount) {
        e = p.next
          if (e == null) {
            p.next = newNode(hash, key, value, null);
            // 当多个key最终hash结果一致的时候，这种现象称为hash碰撞，
            // 此时相同的hash对应的元素是链表结构
            // 数量大于8,则链表转换成红黑树
            if (binCount >= TREEIFY_THRESHOLD - 1) { // -1 for 1st
              treeifyBin(tab, hash);
            }
            break;
          }
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
          break;
        }
        p = e;
      }
    }
    if (e != null) { // existing mapping for key
      // 存在相同的key，变更对应的value
      V oldValue = e.value;
      if (!onlyIfAbsent || oldValue == null) {
        e.value = value;
      }
      afterNodeAccess(e);
      return oldValue;
    }
  }
  ++modCount;
  // 判断添加元素后是否达到阈值，如果达到阈值则进行扩容
  if (++size > threshold) {
    resize();
  }
  afterNodeInsertion(evict);
  return null;
}
```

### get方法

**实现源码(优化过后的、基于jdk1.8)**

```java
public V get(Object key) {
  Node<K,V> e;
  e = getNode(hash(key), key)
  return e == null ? null : e.value;
}
final Node<K,V> getNode(int hash, Object key) {
  Node<K,V>[] tab; 
  Node<K,V> first, e; 
  int n; K k;
  tab = table;
  n = tab.length;
  first = tab[(n - 1) & hash];
  if (tab != null && n > 0 && first != null) {
    // always check first node
    if (first.hash == hash && ((k = first.key) == key || (key != null && key.equals(k)))) {
      return first;
    }
    e = first.next;
    if (e != null) {
      // 如果是红黑树，直接通过getTreeNode获取对应元素值
      if (first instanceof TreeNode) {
        return ((TreeNode<K,V>)first).getTreeNode(hash, key);
      }
      // 不是红黑树，是链表直接循环遍历
      do {
        if (e.hash == hash && ((k = e.key) == key || (key != null && key.equals(k)))) {
          return e;
        }
      } while ((e = e.next) != null);
    }
  }
  return null;
}
```

