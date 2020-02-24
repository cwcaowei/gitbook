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


#### 破坏单例
- 反射

```java
Class<?> clazz = Singleton.class;
// 通过反射拿到私有的构造方法
Constructor c = clazz.getDeclaredConstructor(null);
// 强制访问
c.setAccessible(true);
// 暴力初始化
Object o1 = c.newInstance();
Object o2 = Singleton.getInstance();
System.out.println(o1 == o2); // false
```

解决方法：在单例的构造方法中加以限制：

```java
private Singleton() {
    // 此处的判断作用有：
    // 1、若不加判断直接抛出异常，则内部类中无法创建实例
    // 2、若如上面的代码，在调用单例类提供的获取实例方法之前先暴力初始化，
    // if里的判断调用了内部类，仍会先初始化内部类，
    // 内部类在创建实例时到此判断，此时InnerSingleton.singleton为null，创建实例，
    // 接着继续暴力初始化在此处的判断，InnerSingleton.singleton不为null，
    // 所以此处InnerSingleton.singleton为null只有一种情况：初始化内部类时创建Singleton实例
    if (InnerSingleton.singleton != null) {
        throw new RuntimeException("no permition to new instance");
    }
}
```

- 序列化与反序列化

```java
Singleton s1 = null;
Singleton s2 = Singleton.getInstance();
FileOutputStream fos = new FileOutputStream("Singleton.obj");
ObjectOutputStream oos = new ObjectOutputStream(fos);
oos.writeObject(s2);
oos.flush();
oos.close();
FileInputStream fis = new FileInputStream("Singleton.obj");
ObjectInputStream ois = new ObjectInputStream(fis);
s1 = (Singleton)ois.readObject();
ois.close();
System.out.println(s1 == s2); // false
```
解决方法：在单例类中增加 readResolve()方法

```java
private Singleton readResolve() {
    return InnerSingleton.singleton;
}
```

至此一个完美的单例诞生：

```java
public class Singleton implements Serializable {

    private Singleton() {
      	// 防止反射破解
        if (InnerSingleton.singleton != null) {
            throw new RuntimeException("no permition to new instance");
        }
    }

    public static final Singleton getInstance() {
        // 调用时才会初始化内部类
        return InnerSingleton.singleton;
    }

    // 静态内部类
    // 这种形式没有饿汉式的内存浪费，也没有synchronized的性能问题
    private static class InnerSingleton {
        private static final Singleton singleton = new Singleton();
    }

  	// 防止序列化与反序列化破解
    private Singleton readResolve() {
        return InnerSingleton.singleton;
    }

}
```

原理：

```java
ObjectInputStream ois = new ObjectInputStream(fis);
s1 = (Singleton)ois.readObject();
```

```java
public final Object readObject()
        throws IOException, ClassNotFoundException
    {
    if (enableOverride) {
        return readObjectOverride();
    }

    // if nested read, passHandle contains handle of enclosing object
    int outerHandle = passHandle;
    try {
        Object obj = readObject0(false);  
        handles.markDependency(outerHandle, passHandle);
        ClassNotFoundException ex = handles.lookupException(passHandle);
        if (ex != null) {
            throw ex;
        }
        if (depth == 0) {
            vlist.doCallbacks();
        }
        return obj;
    } finally {
        passHandle = outerHandle;
        if (closed && depth == 0) {
            clear();
        }
    }
}
```

```java
private Object readObject0(boolean unshared) throws IOException {
    ...
    case TC_OBJECT:
      return checkResolve(readOrdinaryObject(unshared));
    ...
}
```

```java
private Object readOrdinaryObject(boolean unshared)
        throws IOException
{
    if (bin.readByte() != TC_OBJECT) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    desc.checkDeserialize();

    Class<?> cl = desc.forClass();
    if (cl == String.class || cl == Class.class
            || cl == ObjectStreamClass.class) {
        throw new InvalidClassException("invalid class descriptor");
    }

    Object obj;
    try {
      // 就是判断一下构造方法是否为空,只要有无参构造方法就会实例化
        obj = desc.isInstantiable() ? desc.newInstance() : null;
    } catch (Exception ex) {
        throw (IOException) new InvalidClassException(
            desc.forClass().getName(),
            "unable to create instance").initCause(ex);
    }

    passHandle = handles.assign(unshared ? unsharedMarker : obj);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(passHandle, resolveEx);
    }

    if (desc.isExternalizable()) {
        readExternalData((Externalizable) obj, desc);
    } else {
        readSerialData(obj, desc);
    }

    handles.finish(passHandle);

    if (obj != null &&
        handles.lookupException(passHandle) == null &&
        desc.hasReadResolveMethod()) // 找到了无参的readResolve()
    {
        Object rep = desc.invokeReadResolve(obj);
        if (unshared && rep.getClass().isArray()) {
            rep = cloneArray(rep);
        }
        if (rep != obj) {
            // Filter the replacement object
            if (rep != null) {
                if (rep.getClass().isArray()) {
                    filterCheck(rep.getClass(), Array.getLength(rep));
                } else {
                    filterCheck(rep.getClass(), -1);
                }
            }
          	// obj被重新赋值为反射调用readResolve方法得到的返回值
            handles.setObject(passHandle, obj = rep);
        }
    }

    return obj;
}
```

```java
boolean isInstantiable() {
    requireInitialized();
    return (cons != null); 
}
```

```java
boolean hasReadResolveMethod() {
    requireInitialized();
    return (readResolveMethod != null); 
}

// ObjectStreamClass(final Class<?> cl)方法中赋值
// 通过反射找到一个无参的readResolve()方法
readResolveMethod = getInheritableMethod(
                        cl, "readResolve", null, Object.class);
```

```java
Object invokeReadResolve(Object obj)
        throws IOException, UnsupportedOperationException
{
    requireInitialized();
     if (readResolveMethod != null) {
        try {
            return readResolveMethod.invoke(obj, (Object[]) null);
        } catch (InvocationTargetException ex) {
            Throwable th = ex.getTargetException();
            if (th instanceof ObjectStreamException) {
                throw (ObjectStreamException) th;
            } else {
                throwMiscException(th);
                throw new InternalError(th);  // never reached
            }
        } catch (IllegalAccessException ex) {
            // should not occur, as access checks have been suppressed
            throw new InternalError(ex);
        }
    } else {
        throw new UnsupportedOperationException();
    }
}
```

通过以上的源码分析可以看出，虽然增加 readResolve()方法返回实例，解决了单例被破坏的问题，但是，实际上还是实例化了两次，只不过第一次实例化的对象没有被返回而已。如果创建对象的动作发生频率增大，就意味着内存分配开销也就随之增大，登记式单例将解决这个问题。


#### 登记（注册）式单例
就是将每一个实例都登记到某一个地方，使用唯一的标识获取实例。

- 容器缓存

```java
public class Singleton {
    
    public static final Map<String,Object> container = new ConcurrentHashMap<>();
    
    private Singleton(){}

    public static Object getInstance(String className) {

        if (!container.containsKey(className)) {
            synchronized (Singleton.class) {
                if (!container.containsKey(className)) {
                    try {
                        container.put(className, Class.forName(className).newInstance());
                    } catch (InstantiationException|IllegalAccessException|ClassNotFoundException e) {
                        e.printStackTrace();
                    }
                }
            }
        }
        return container.get(className);
    }
}
```

- 枚举登记

```java
public enum Singleton {
    
  	// 枚举类构造方法默认私有化
  	// INSTANCE默认为static final
    INSTANCE; // Singleton.INSTANCE就是Singleton的唯一实例
  
}
```

通过工具jad反编译后

```java

public final class Singleton extends Enum
{

    public static Singleton[] values()
    {
        return (Singleton[])$VALUES.clone();
    }

    public static Singleton valueOf(String name)
    {
        return (Singleton)Enum.valueOf(com/example/demo/Singleton, name);
    }

    private Singleton(String s, int i)
    {
        super(s, i);
    }

    public static final Singleton INSTANCE;
    private static final Singleton $VALUES[];

    static 
    {
      	// 静态代码块中赋值，属于饿汉式
        INSTANCE = new Singleton("INSTANCE", 0);
        $VALUES = (new Singleton[] {
            INSTANCE
        });
    }
}
```

那么能否通过反射破解呢
```java
public T newInstance(Object ... initargs)
        throws InstantiationException, IllegalAccessException,
               IllegalArgumentException, InvocationTargetException
    {
        if (!override) {
            if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
                Class<?> caller = Reflection.getCallerClass();
                checkAccess(caller, clazz, null, modifiers);
            }
        }
        // 如果是枚举类型直接抛异常
        if ((clazz.getModifiers() & Modifier.ENUM) != 0)
            throw new IllegalArgumentException("Cannot reflectively create enum objects");
        ConstructorAccessor ca = constructorAccessor;   // read volatile
        if (ca == null) {
            ca = acquireConstructorAccessor();
        }
        @SuppressWarnings("unchecked")
        T inst = (T) ca.newInstance(initargs);
        return inst;
    }
```

能否通过序列化与反序列化破解呢

```java
private Object readObject0(boolean unshared) throws IOException {
    ...
    case TC_ENUM:
    return checkResolve(readEnum(unshared));
    ...
}
```

```java
private Enum<?> readEnum(boolean unshared) throws IOException {
    if (bin.readByte() != TC_ENUM) {
        throw new InternalError();
    }

    ObjectStreamClass desc = readClassDesc(false);
    if (!desc.isEnum()) {
        throw new InvalidClassException("non-enum class: " + desc);
    }

    int enumHandle = handles.assign(unshared ? unsharedMarker : null);
    ClassNotFoundException resolveEx = desc.getResolveException();
    if (resolveEx != null) {
        handles.markException(enumHandle, resolveEx);
    }

    String name = readString(false);
    Enum<?> result = null;
    Class<?> cl = desc.forClass();
    if (cl != null) {
        try {
            @SuppressWarnings("unchecked")
            // 通过 Class 对象和类名找到一个唯一的枚举对象,没有创建新的实例
            Enum<?> en = Enum.valueOf((Class)cl, name);
            result = en;
        } catch (IllegalArgumentException ex) {
            throw (IOException) new InvalidObjectException(
                "enum constant " + name + " does not exist in " +
                cl).initCause(ex);
        }
        if (!unshared) {
           handles.setObject(enumHandle, result);
        }
    }

    handles.finish(enumHandle);
    passHandle = enumHandle;
    return result;
}
```
#### 附：双重检查volatile关键字的必要性

singleton = new Singleton(); 创建了一个对象。这一行代码可以分解为如下的3行伪代码：

```c
memory = allocate(); // 1：分配对象的内存空间 
ctorInstance(memory); // 2：初始化对象 
instance = memory; // 3：设置instance指向刚分配的内存地址
```

上面3行伪代码中的2和3之间，可能会被重排序（在一些JIT编译器上，这种重排序是真实发生的）。2和3之间重排序之后的执行时序如下：

```c
memory = allocate(); // 1：分配对象的内存空间 
instance = memory; // 3：设置instance指向刚分配的内存地址 // 注意，此时对象还没有被初始化！ 
ctorInstance(memory); // 2：初始化对象
```

根据《The Java Language Specification,Java SE 7 Edition》（后文简称为Java语言规范），所有线程在执行Java程序时必须要遵守intra-thread semantics。intra-thread semantics保证重排序不会改变单线程内的程序执行结果。换句话说，intra-thread semantics允许那些在单线程内，不会改变单线程程序执行结果的重排序。上面3行伪代码的2和3之间虽然被重排序了，但这个重排序并不会违反intra-thread semantics。这个重排序在没有改变单线程程序执行结果的前提下，可以提高程序的执行性能。  如下图所示，只要保证2排在4的前面，即使2和3之间重排序了，也不会违反intra-thread semantics。
![](https://raw.githubusercontent.com/cwcaowei/gitbook/master/static/memoryreorder.png)
由于单线程内要遵守intra-thread semantics，从而能保证A线程的执行结果不会被改变。但是，当线程A和B按下图的时序执行时，B线程将看到一个还没有被初始化的对象。 回到本文的主题，DoubleCheckedLocking示例代码的第7行（instance=new Singleton();）如果发生重排序，另一个并发执行的线程B就有可能在第4行判断instance不为null（因为第4行处没有锁，线程B随时可能进来访问）。线程B接下来将访问instance所引用的对象，但此时这个对象可能还没有被A线程初始化！
![](https://raw.githubusercontent.com/cwcaowei/gitbook/master/static/memoryreorder1.png)
如下的执行时序，这里A2和A3虽然重排序了，但Java内存模型的intra-thread semantics将确保A2一定会排在 A4前面执行。因此，线程A的intra-thread semantics没有改变，但A2和A3的重排序，将导致线程 B在B1处判断出instance不为空，线程B接下来将访问instance引用的对象。此时，线程B将会访问到一个还未初始化的对象。
![](https://raw.githubusercontent.com/cwcaowei/gitbook/master/static/memoryreorder2.png)