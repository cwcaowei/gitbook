#### 概念

核心在于拷贝原型对象。当对象的构建过程比较耗时时，以一个已存在的对象为原型，基于内存二进制流进行拷贝，无需再经历耗时的对象初始化过程（调用构造函数），性能提升许多。


#### 适用场景

- 类初始化消耗资源较多
- new产生的一个对象需要非常繁琐的过程（new之后需要初始化一些数据）
- 构造函数复杂
- 循环体中产生大量对象时


#### 组成

- 抽象原型（Prototype）：规定拷贝方法

```java
public interface IPrototype<T> {
    T clone();
}
```

- 具体原型（Concrete Prototype）：被拷贝的对象

```java
@Data
public class ConcretePrototype implements IPrototype {

    private int age;
    private String name;

    @Override
    public ConcretePrototype clone() {
        ConcretePrototype concretePrototype = new ConcretePrototype();
        concretePrototype.setAge(this.age);
        concretePrototype.setName(this.name);
        return concretePrototype;
    }

}
```

- 客户端（Client）：通过拷贝的方式创建对象

```java
public static void main(String[] args) {
    // 创建原型对象
    ConcretePrototype prototype = new ConcretePrototype();
    prototype.setAge(18);
    prototype.setName("Tom");
    // 拷贝原型对象
    ConcretePrototype cloneType = prototype.clone();
}
```

#### Object中的clone()与Cloneable接口

以上是原型模式最基本的写法，实际上抽象原型不需要自己定义，所有类的父类Object中有一个native方法clone()，JDK提供了一个Cloneable接口，它的作用只有一个，就是在运行时通知虚拟机可以安全地在实现了此接口的类上使用clone方法。在jvm中，只有实现了这个接口的类才可以通过clone()被拷贝，否则在运行时会抛出CloneNotSupportedException异常。基于此，具体原型的实现如下：

```java
@Data
public class ConcretePrototype implements Cloneable {

    private int age;
    private String name;

    @Override
    public ConcretePrototype clone() {
        try {
            return (ConcretePrototype)super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
            return null;
        }
    }

}
```


#### 深克隆与浅克隆

- 浅克隆

  以上具体原型对象中的属性只是基本数据类型，但如果是引用类型呢，比如List，会发现在克隆对象的list中新增一个元素，原型对象里的list也会跟着新增，显然对于引用类型的对象，并未在内存中开辟一块新的空间，并拷贝数据，原型对象与克隆对象的引用类型的变量指向的是同一个内存地址，这就是浅克隆。或者说克隆对象的改变会导致原型对象的改变就是浅克隆。因此对基础数据类型及它们的封装类和String的克隆都是深克隆。

- 深克隆

  实现引用类型的对象的实际拷贝，原型对象与克隆对象完全独立。

  - 序列化

  ```java
  @Data
  public class ConcretePrototype implements Cloneable {

      private int age;
      private String name;

      @Override
      public ConcretePrototype clone() {
          try {
              ByteArrayOutputStream bos = new ByteArrayOutputStream();
              ObjectOutputStream oos = new ObjectOutputStream(bos);
              oos.writeObject(this);

              ByteArrayInputStream bis = new ByteArrayInputStream(bos.toByteArray());
              ObjectInputStream ois = new ObjectInputStream(bis);
              return (ConcretePrototype)ois.readObject();
          } catch (Exception e) {
              e.printStackTrace();
              return null;
          }
      }

  }
    ```

  - JSON

  - 手动复制


#### 原型模式与单例模式

原型模式与单例模式是互相对立的。
