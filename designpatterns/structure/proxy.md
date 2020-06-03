#### 组成

- 抽象主题：声明方法，由真实主题与代理实现具体逻辑，是接口/抽象类

  ```java
  public interface ISubject {
      void request();
  }
  ```

- 真实主题：被代理类，在实现抽象主题声明的方法里做了具体的逻辑处理

  ```java
  public class RealSubject implements ISubject {
      public void request() {
          System.out.println("real service is called.");
      }
  }
  ```

- 代理主题：代理类，持有真实主题的引用，在实现抽象主题声明的方法时，在真实主题的逻辑处理前后增加一些处理逻辑

  ```java
  public class Proxy implements ISubject {

      private ISubject subject;

      public Proxy(ISubject subject) {
          this.subject = subject;
      }

      public void request() {
          before();
          subject.request();
          after();
      }

      public void before() {
          System.out.println("called before request().");
      }

      public void after() {
          System.out.println("called after request().");
      }
  }
  ```

#### 静态代理

```java
Proxy proxy = new Proxy(new RealSubject());
proxy.request();
```

如上的实现方式便是静态代理，通常只代理一个明确的类

#### 动态代理

- JDK代理

  - 示例

    ```java
    import java.lang.reflect.InvocationHandler;
    import java.lang.reflect.Method;
    import java.lang.reflect.Proxy;

    public class JdkProxy implements InvocationHandler {

        private ISubject target;

        public ISubject getInstance(ISubject target) {
            this.target = target;
            Class<?> clazz = target.getClass();
            return (ISubject) Proxy.newProxyInstance(clazz.getClassLoader(), clazz.getInterfaces(),this);
        }

        public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
            before();
            Object result = method.invoke(this.target, args);
            after();
            return result;
        }

        public void before() {
            System.out.println("called before request().");
        }

        public void after() {
            System.out.println("called after request().");
        }
    }
    ```

    ```java
    JdkProxy proxy = new JdkProxy();
    ISubject subject = proxy.getInstance(new RealSubject());
    subject.request();
    ```

  - 原理

    通过字节码重组，重新生成对象(JDK有一个规范，classpath下$开头的.class文件都是自动生成的)来代替原始的被代理对象，新的对象继承了Proxy类并实现了ISubject接口，同时重写了request等方法，在静态块中用反射获取了被代理对象的所有方法且保存了它们的引用，在重写的方法中用反射调用被代理对象的方法
    - 获取被代理对象的引用，并通过反射获取其所有接口
    - 重新生成一个新的实现了被代理对象所有接口的类
    - 编译成.class文件并加载到JVM中

    ```java
    import java.lang.reflect.*;
    public final class $Proxy0 extends Proxy implements ISubject {

        public $Proxy0(InvocationHandler invocationhandler) {
            super(invocationhandler);
        }

        public final boolean equals(Object obj) {
            try {
                return ((Boolean)super.h.invoke(this, m1, new Object[] {obj})).booleanValue();
            }
            catch(Error _ex) { }
            catch(Throwable throwable) {
                throw new UndeclaredThrowableException(throwable);
            }
        }

        public final void request() {
            try {
                super.h.invoke(this, m3, null);
                return;
            }
            catch(Error _ex) { }
            catch(Throwable throwable) {
                throw new UndeclaredThrowableException(throwable);
            }
        }

        public final String toString() {
            try {
                return (String)super.h.invoke(this, m2, null);
            }
            catch(Error _ex) { }
            catch(Throwable throwable) {
                throw new UndeclaredThrowableException(throwable);
            }
        }

        public final int hashCode() {
            try {
                return ((Integer)super.h.invoke(this, m0, null)).intValue();
            }
            catch(Error _ex) { }
            catch(Throwable throwable) {
                throw new UndeclaredThrowableException(throwable);
            }
        }           

        private static Method m1;
        private static Method m3;
        private static Method m2;
        private static Method m0;
        static {
            try {
                m1 = Class.forName("java.lang.Object").getMethod("equals", new Class[] {Class.forName("java.lang.Object")});
                m3 = Class.forName("com.learn.designpatterns.proxy.RealSubject").getMethod("request", new Class[0]);
                m2 = Class.forName("java.lang.Object").getMethod("toString", new Class[0]);
                m0 = Class.forName("java.lang.Object").getMethod("hashCode", new Class[0]);
            }
            catch(NoSuchMethodException nosuchmethodexception) {
                throw new NoSuchMethodError(nosuchmethodexception.getMessage());
            }
            catch(ClassNotFoundException classnotfoundexception) {
                throw new NoClassDefFoundError(classnotfoundexception.getMessage());
            }
        }
    }
    ```

- Cglib代理

  被代理对象不需要实现接口,会忽略被代理对象final修饰的方法，

  - 示例

    ```java
    import net.sf.cglib.proxy.Enhancer;
    import net.sf.cglib.proxy.MethodInterceptor;
    import net.sf.cglib.proxy.MethodProxy;
    import java.lang.reflect.Method;

    public class CglibProxy implements MethodInterceptor {

        public Object getInstance(Class<?> clazz) throws Exception {
            Enhancer enhancer = new Enhancer();
            enhancer.setSuperclass(clazz);
            enhancer.setCallback(this);
            return enhancer.create();
        }

        public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable { //业务的增强
            before();
            Object obj = methodProxy.invokeSuper(o,objects); after();
            return obj;
        }

        public void before() {
            System.out.println("called before request().");
        }

        public void after() {
            System.out.println("called after request().");
        }
    }
    ```

    ```Java
    RealSubject subject = (RealSubject)new CglibProxy().getInstance(RealSubject.class);
    subject.request();
    ```

  - 原理

    在new CglibProxy前添加System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/Users/cw/desktop/cglib_proxy_class/");到cglib_proxy_class目录下可以看到cglib重新生成了3个对象，其中一个对象继承了被代理类即代理类，另外两个继承了FastClass类，这两个不是跟代理类一起生成的，而是在第一次执行MethodProxy的invoke()或 invokeSuper()方法时生成的。
    CGLib执行代理方法的效率之所以比JDK的高，是因为CGlib采用了 FastClass机制：为代理类和被代理类各生成一个类，这个类会为代理类或被代理类的方法分配一个index(int类型，供FastClass直接定位要调用的方法并直接进行调用)，省去了反射调用，所以调用效率比JDK代理通过反射调用高。

    ```java
    import java.lang.reflect.Method;
    import net.sf.cglib.core.ReflectUtils;
    import net.sf.cglib.core.Signature; import net.sf.cglib.proxy.*;

    public class RealSubject$$EnhancerByCGLIB$$3feeb52a extends RealSubject implements Factor {
        ...

        // 代理方法(methodProxy.invokeSuper()方法会调用)
        final void CGLIB$request$0() {
            super.request();
        }

        //被代理方法(methodProxy.invoke()方法会调用，这就是为什么在拦截器中调用 methodProxy.invoke 会发生死循环，一直在调用拦截器）
        public final void request() {
            MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
            if (this.CGLIB$CALLBACK_0 == null) {
                CGLIB$BIND_CALLBACKS(this);
                var10000 = this.CGLIB$CALLBACK_0;
            }

            if (var10000 != null) {
                var10000.intercept(this, CGLIB$request$0$Method, CGLIB$emptyArgs, CGLIB$request$0$Proxy);
            } else {
                super.request();
            }

        }
        ...
    }
    ```

#### CGLib和JDK动态代理对比

- JDK动态代理实现了被代理对象的接口，CGLib代理继承了被代理对象

- JDK动态代理和CGLib代理都在运行期生成字节码，JDK动态代理直接写Class 字节码，CGLib代理使用ASM框架写Class字节码，CGlib代理实现更复杂，生成代理类比JDK动态代理效率低

- JDK动态代理调用代理方法是通过反射机制调用的，CGLib代理是通过 FastClass机制直接调用方法的，CGLib代理的执行效率更高



#### 静态代理和动态代理的区别

- 对于静态代理，如果被代理类增加了新的方法，代理类需要同步增加，违反了开闭原则

- 动态代理采用在运行时动态生成代码的方式，没有对被代理类扩展的限制，遵循开闭原则

- 若动态代理要对目标类的增强逻辑进行扩展，结合策略模式，只需要新增策略类便可，无需修改代理类的代码
