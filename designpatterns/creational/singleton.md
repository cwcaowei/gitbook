# 单例

#### 饿汉式单例
饿汉式单例是在类加载的时候就立即创建对象，绝对线程安全，在线程还没出现以前就实例化了，不会存在访问安全问题，优点是没有锁及复杂逻辑，效率高，缺点是不管使不使用都占着空间，浪费内存。
```java
public class Singleton {

    private static final Singleton singleton = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return singleton;
    }
}
```

```java
public class Singleton {

    private static Singleton singleton = null;
    
    static {
        singleton = new Singleton();
    }

    private Singleton() {}

    public static Singleton getInstance() {
        return singleton;
    }
}
```


#### 懒汉式单例
懒汉式单例是在获取实例的方法被调用时才创建对象。
```java
public class Singleton {

    private static Singleton singleton = null;

    private Singleton() {}

    // 在线程数量比较多情况下，会导致大批量线程出现阻塞，从而导致程序运行性能大幅下降
    public synchronized static Singleton getInstance() {
        if (singleton == null) {
            singleton = new Singleton();
        }
        return singleton;
    }
}
```

```java
public class Singleton {

    private static volatile Singleton singleton = null;

    private Singleton() {}

    // 双重检查锁
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.getClass()) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
```java
public class Singleton {

    private Singleton() {}

    public static final Singleton getInstance() {
        // 调用时才会初始化内部类
        return InnerSingleton.singleton;
    }

    // 静态内部类
    // 这种形式没有饿汉式的内存浪费，也没有synchronized的性能问题
    private static class InnerSingleton {
        private static final Singleton singleton = new Singleton();
    }

}
```
#### 登记式单例

#### 破坏单例
- 反射

- 序列化与反序列化