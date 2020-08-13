#### 概念

负责任务的调用或分配，是一种特殊的静态代理，不属于23种设计模式，经典的使用有双亲委派、Spring的DispatcherServlet

#### 组成

- 抽象任务角色（Task）：定义任务

  ```java
  public interface IEmployee {

      void doSomething();

  }
  ```

- 具体任务角色（Concrete）：实现具体任务并执行

  ```java
  public class Programmer implements IEmployee {

      @Override
      public void doSomething(String task) {
          System.out.println("我擅长编程，我开始做" + task);
      }

  }

  public class ProjectManager implements IEmployee {

      @Override
      public void doSomething(String task) {
          System.out.println("我擅长产品，我开始做" + task);
      }

  }
  ```

- 委派者角色（Delegate）：实现抽象任务角色定义的接口，持有具体角色的引用，在其中决定调用哪个具体任务角色

  ```java
  public class Leader implements IEmployee {

      private static Map<String, IEmployee> subordinates = new HashMap<>();

      static {
          subordinates.put("软件开发", new Programmer());
          subordinates.put("原型设计", new ProjectManager());
      }

      @Override
      public void doSomething(String task) {
          IEmployee subordinate = subordinates.get(task);
          if (subordinate != null) {
              subordinate.doSomething(task);
          }

      }

  }
  ```

  实际调用者只持有委派者的引用，给委派者分配任务，委派者再将任务分配下去

  ```java
  String task = "原型设计";
  new Leader().doSomething(task);
  ```
