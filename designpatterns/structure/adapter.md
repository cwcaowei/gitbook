#### 概念

又叫变压器模式，通过一个中间层将一个与客户端的期望不匹配的接口转换为匹配的另一个接口

#### 组成

以家用的220V交流电需要转化为手机充电用的5V直流电为例

- 目标角色（Target）：期望接口

  ```java
  public interface DC5 {

      int outputDC5V();

  }

- 源角色（Adaptee）：内容满足需求但不匹配的接口

  ```java
  public class AC220 {

      public int outputAC220V() {
          return 220;
      }

  }
  ```

- 适配器（Adapter）：将源角色转换为目标角色的实例

  - 类适配器

    让Adapter继承Adaptee并实现Target，在具体的实现方法中做转换

    ```java
    public class PowerAdapter extends AC220 implements DC5 {

      @Override
      public int output5V() {
          return super.outputAC220V() / 44;
      }

    }
    ```

  - 对象适配器

    让Adapter实现Target并内部持有Adaptee的引用，在Target接口规定的方法内转换Adaptee

    ```java
    public class PowerAdapter implements DC5 {

        private AC220 ac220;

        public PowerAdapter(AC220 ac220) {
            this.ac220 = ac220;
        }

        @Override
        public int output5V() {    
            return ac220.outputAC220V() / 44;
        }

    }
    ```

  - 接口适配器

    当不需要全部实现接口提供的方法时，可先设计一个抽象类（适配器）实现接口，并为该接口中每个方法提供一个默认实现/空实现，那么原先需要实现该接口的类改为继承抽象类同时可有选择的覆盖父类的某些方法来实现需求，匿名内部类是一种常见的形式

    ```java
    public interface InterfaceA {

        void metho1();

        void metho2();

        void metho3();

        void metho4();

        void metho5();
    }
    ```

    ```java
    public abstract class Adapter implements InterfaceA {

        @Override
        public void metho1() {}

        @Override
        public void metho2() {}

        @Override
        public void metho3() {}

        @Override
        public void metho4() {}

        @Override
        public void metho5() {}

    }
    ```

    ```java
    Adapter adapter = new Adapter() {
        @Override
        public void method1() {
            System.out.println("");
        }
    };
    adapter.method1();
    ```
