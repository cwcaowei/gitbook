#### 概念

又叫政策模式，用继承和多态解决在同一行为在不同场景下的不同实现，避免了if/else或switch所带来的复杂和臃肿

#### 适用场景

- 针对同一类型问题，有多种处理方式，且每一种都能独立解决问题
- 算法需要自由切换且只取其中一种的场景

#### 组成

- 抽象策略角色（IStrategy）：规定策略或算法的行为

  ```java
  public interface IPayStrategy {

      void pay();

  }
  ```
- 具体策略角色（ConcreteStrategy）：具体的策略或算法实现

  ```java
  public class WeChatPayStrategy implements IPayStrategy {

      @Override
      public void pay() {
          System.out.println("微信支付");
      }

  }
  ```

  ```java
  public class AliPayStrategy implements IPayStrategy {

      @Override
      public void pay() {
          System.out.println("支付宝支付");
      }

  }
  ```

  ```java
  public class BankPayStrategy implements IPayStrategy {

      @Override
      public void pay() {
          System.out.println("银行支付");
      }

  }
  ```


- 上下文角色（Context）：用来操作策略或算法，屏蔽客户端对策略或算法的直接访问，封装可能存在的变化，一般结合工厂与单例

  ```java
  public class PayFactory {

      private static Map<String, IPayStrategy> map = new HashMap<>();

      static {
          map.put("wechat", new WeChatPayStrategy());
          map.put("ali", new AliPayStrategy());
          map.put("bank", new BankStrategy());
      }

      public static IPayStrategy getStrategy(String key) {
          return map.get(key);
      }

  }
  ```

- 客户端：调用上下文角色

  ```java
  IPayStrategy aliPay = PayFactory.getStrategy("ali");
  aliPay.pay();
  ```
