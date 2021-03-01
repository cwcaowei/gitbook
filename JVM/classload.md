#### 类加载的流程

- 装载

  - 通过类的全限定名获取它的二进制字节流（字节码增强就是在这一阶段做）

  - 将二进制字节流代表的静态存储结构转化为方法区的运行时数据结构

  - 在堆中生成该类对应的Class对象，作为方法区数据的访问入口

- 验证

  验证Class对象中的字节流符合jvm要求，不会对jvm安全产生危害，可通过-Xverify:none关闭验证。

- 准备

  为类变量（静态变量）在方法区中分配内存并设置默认初始值，int、long、short、char、byte、boolean、float、double、reference默认值分别为0、0L、（short）0、'\u0000'、（byte）0、false、0.0f、0.0d、null。这里的类变量不包含用final修饰的static变量，因为这样的变量称为ConstantValue属性（仅限基本类型和String），在编译时就已分配好，在准备阶段会设置代码中给它指定的值，而非基本类型和String的以及没有final修饰的static的则在类构造器中被设置代码指定的值，其他的则在实例构造器中被赋值。比如
  ```java
  private static final int a = 2;
  private static final int b = 2;
  ```
  在准备阶段，a就被赋值为了2，而b只是赋值为默认的0，b得等到初始化阶段才会被赋值为2。

- 解析

  将类常量池中的符号引用转化为直接引用。

- 初始化

  调用类构造器，只有主动引用会触发该阶段，被动引用不会触发该阶段。

  - 主动引用

    - new创建实例

    - 访问非final的静态变量或者对非final的静态变量赋值

    - 调用静态方法

    - 反射，如Class.foeName("com...mysql.Driver")

    - 初始化子类，则父类也会被初始化

    - jvm启动时被标为启动类的类，如xxApplication

  - 被动引用

    - 引用父类的静态字段，只会引起父类的初始化，不会引起子类的初始化

    - 定义类数组，不会引起初始化

    - 引用类的static final变量，不会引起初始化

#### 类加载器

类加载器共有4种类型，一个类在同一个类加载器中具有唯一性，但不同类加载器中是可以有同名类的，这样的类无法通过instanceof、equals的校验。

- BootstrapClassLoader

  又称根加载器，加载java.*.* 这些类

- ExtensionClassLoader

  加载javax.*.* 这些类

- AppClassLoader

  又称系统类加载器，加载开发写的代码和第三方jar包里类，可以通过ClassLoader.getSystemClassLoader()获取


![](/assets/jvm/classloader.png)


#### 双亲委派

  检查类是否已加载，如果没有则委托父类加载器加载，父类加载器再检查类是否已加载，如果没有则委托父类的父类加载器加载，这样一直往上，如果最顶层的加载器在类路径范围内找不到该类时，就把类加载的任务扔回给子类加载器，这样一直往下，最终可能又回到最开始的类加载器。比如AppClassLoader先委托ExtensionClassLoader，ExtensionClassLoader又委托BootstrapClassLoader，BootstrapClassLoader一找没找到，就会打回给ExtensionClassLoader，ExtensionClassLoader再一找，没找到，又会打回给AppClassLoader，AppClassLoader找到了就返回对应的Class，没找到就抛异常，不会往下找子类加载器去加载。

  ```java
  protected Class<?> loadClass(String name, boolean resolve)
        throws ClassNotFoundException
  {
      synchronized (getClassLoadingLock(name)) {
          // First, check if the class has already been loaded
          Class<?> c = findLoadedClass(name);
          if (c == null) {
              long t0 = System.nanoTime();
              try {
                  if (parent != null) {
                      c = parent.loadClass(name, false);
                  } else {
                      c = findBootstrapClassOrNull(name);
                  }
              } catch (ClassNotFoundException e) {
                  // ClassNotFoundException thrown if class not found
                  // from the non-null parent class loader
              }

              if (c == null) {
                  // If still not found, then invoke findClass in order
                  // to find the class.
                  long t1 = System.nanoTime();
                  c = findClass(name);

                  // this is the defining class loader; record the stats
                  sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                  sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                  sun.misc.PerfCounter.getFindClasses().increment();
              }
          }
          if (resolve) {
              resolveClass(c);
          }
          return c;
      }
  }
  ```

#### 打破双亲委派

- SPI

  JDK在核心类库rt.jar中定义接口及调用逻辑，开发者在META-INF/serices目录下提供实现。

- 自定义类加载器并重写loadClass方法，比如tomcat

#### 自定义类加载器

- 主要是继承ClassLoader，并重写findClass方法，在findClass方法里调用defineCass方法，findClass方法不会破坏双亲委派

- 要加载的类不能放在类路径下，否则根据双亲委派，该类会先被AppClassLoader加载

#### 延迟加载

JVM并不是一次性加载所有类的，它是按需加载，也就是延迟加载。程序在运行的过程中会逐渐遇到很多不认识的新类，这时候就会通过ClassLoader来加载这些类。加载完成后就会将Class对象存在ClassLoader里面，下次就不需要重新加载了。
