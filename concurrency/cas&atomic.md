#### CAS的ABA问题

- 问题

  CAS是检查值没有发生变化则更新，但是如果一个值A变成了B，然后再变成A，刚好在做CAS时检查发现值并没有变化依然为A，但是实际上却发生了变化。

- 解决方案

  添加一个版本号，原来的变化路径A->B->A就变成了1A->2B->3A。
  atomic包中的AtomicStampedReference即是这样，首先检查当前引用是否等于预期引用，接着检查当前标识是否等于预期标识，都相等的情况下才会更新。

  ```java
  public boolean compareAndSet(V expectedReference,
                               V newReference,
                               int expectedStamp,
                               int newStamp) {
      Pair<V> current = pair;
      return
          expectedReference == current.reference &&
          expectedStamp == current.stamp &&
          ((newReference == current.reference &&
            newStamp == current.stamp) ||
           casPair(current, Pair.of(newReference, newStamp)));
  }
  ```
                                                   

#### Atomic原子类

- 基本类型对应：AtomicBoolean、AtomicInteger、AtomicLong

- 数组类型对应：AtomicIntegerArray、AtomicLongArray、AtomicReferenceArray

- 引用类型对应：AtomicReference、AtomicReferenceFieldUpdater、AtomicMarkableReference（除引用类型还带有一个boolean类型的标记位）

- 对象的属性类型对应：AtomicIntegerFieldUpdater、AtomicLongFieldUpdater、AtomicStampedReference

Atomic包里的类基本都是使用Unsafe的如下三个方法之一实现CAS的

```java
public final native boolean compareAndSwapObject(Object var1, long var2, Object var4, Object var5);

public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);

public final native boolean compareAndSwapLong(Object var1, long var2, long var4, long var6);
```

那么AtomicBoolean怎么实现呢？可以看到是先把Boolean转换成整型，再使用compareAndSwapInt进行CAS，所以原子更新char、float和double变量也可以用类似的思路来实现。

```java
public final boolean compareAndSet(boolean expect, boolean update) {
    int e = expect ? 1 : 0;
    int u = update ? 1 : 0;
    return unsafe.compareAndSwapInt(this, valueOffset, e, u);
}
```
