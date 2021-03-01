#### 强引用

如下，把一个对象赋给一个引用变量a，这个对象处于可达状态，永远不会被垃圾回收，JVM宁愿抛出OOM，因此强引用是造成内存泄漏的主要原因之一。

```java
Object a = new Object()；
a = null; // 释放强引用
```

#### 软引用

用SoftReference类来实现，可以引用队列联合使用，对于只有软引用的对象来说，当系统内存足够时它不会被回收，当系统内存空间不足时它会被回收。软引用通常用来实现缓存。

```java
Object a = new Object()；
SoftReference<Object> sr = new SoftReference<Object>(a);
a = null; // 释放强引用
// 先判断软引用是否已被回收
if (wr != null) {
  Object obj = sr.get(); // 获取软引用里的对象
}
```


#### 弱引用

用WeakReference类来实现，可以引用队列联合使用，它比软引用的生存期更短，对于只有弱引用的对象来说，只要垃圾回收机制一运行，不管JVM的内存空间是否足够，总会回收该对象占用的内存。

```java
Object a = new Object()；
WeakReference<Object> wr = new WeakReference<Object>(a);
a = null; // 释放强引用
// 先判断弱引用是否已被回收
if (wr != null) {
  Object obj = wr.get(); // 获取弱引用里的对象
}

```

#### 虚引用

用PhantomReference类来实现，它不能单独使用，必须和引用队列联合使用。主要作用是跟踪对象被垃圾回收的状态。

```java
Object a = new Object()；
ReferenceQueue<Object> queue = new ReferenceQueue<Object>();
PhantomReference<Object> pr = new PhantomReference<Object>(a, queue);
a = null; // 释放强引用
```
