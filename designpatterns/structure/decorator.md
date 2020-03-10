#### 概念

在不改变原有对象的基础上，将功能附加到对象上，透明且动态的扩展类的功能。


#### 组成

- 抽象组件（Component）：一个接口/抽象类，充当被装饰类的原始对象，规定了被装饰对象的行为。
```java
  public interface Cake {

      String getType();

      int getPrice();

  }
```

- 具体组件（Concrete Component）：实现/继承Component的一个具体对象，也是被装饰对象。
```java
  public class BaseCake implements Cake {

      @Override
      public String getType() {
          return "普通蛋糕";
      }

      @Override
      public int getPrice() {
          return 20;
      }

  }
```

- 抽象装饰器（Decorator）：Concrete Component的装饰器，一般是一个抽象类，有一个Component类型的成员变量，在构造函数中赋值，强制其子类按其构造方式接收一个Component，如果不需要实现许多装饰器，可以省略该类。
```java
  public abstract class CakeDecorator implements Cake {

      private Cake cake;

      public CakeDecorator(Cake cake) {
          this.cake = cake;
      }

      @Override
      public String getType() {
          return cake.getType();
      }

      @Override
      public int getPrice() {
          return cake.getPrice();
      }

  }
```


- 具体装饰器（Concrete Decorator）：继承Decorator类，扩展Component的功能。
```java
  public class FruitCakeDecorator extends CakeDecorator {

      public FruitCakeDecorator(Cake cake) {
          super(cake);
      }

      @Override
      public String getType() {
          return super.getType() + "水果";
      }

      @Override
      public int getPrice() {
          return super.getPrice() + 10;
      }

  }
```

  ```java
  public class IceCreamCakeDecorator extends CakeDecorator {

      public IceCreamCakeDecorator(Cake cake) {
          super(cake);
      }

      @Override
      public String getType() {
          return super.getType() + "冰激凌";
      }

      @Override
      public int getPrice() {
          return super.getPrice() + 15;
      }

  }
  ```


#### 调用示例

```java
Cake cake = new BaseCake();
System.err.println(cake.getType() + ":" + cake.getPrice()); // 普通蛋糕:20
cake = new IceCreamCakeDecorator(cake);
System.err.println(cake.getType() + ":" + cake.getPrice()); // 普通蛋糕+冰淇淋:35
cake = new FruitCakeDecorator(cake);
System.err.println(cake.getType() + ":" + cake.getPrice()); // 普通蛋糕+冰淇淋+水果:45
```

#### 与继承的区别

如果是继承，要满足以上场景需要冰淇淋蛋糕，冰淇淋水果蛋糕两个子类，如果这时候需要个水果蛋糕，就得加一个子类，对于普通蛋糕上可扩展项越多，子类将会成几何数增多，而装饰器与扩展项的关系则是1对1。
