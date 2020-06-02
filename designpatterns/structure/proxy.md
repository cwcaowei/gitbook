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

- Cglib代理

  被代理对象不需要实现接口

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

#### 静态代理和动态代理的区别

- 对于静态代理，如果被代理类增加了新的方法，代理类需要同步增加，违反了开闭原则

- 动态代理采用在运行时动态生成代码的方式，没有对被代理类扩展的限制，遵循开闭原则

- 若动态代理要对目标类的增强逻辑进行扩展，结合策略模式，只需要新增策略类便可，无需修改代理类的代码
