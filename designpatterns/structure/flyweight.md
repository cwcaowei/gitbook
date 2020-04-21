#### 概念

又称为轻量级模式，是对象池的一种实现。把对象的状态分为内部状态和外部状态，内部状态是不变的（比如数据库连接对象中的用户名、密码、地址、驱动），外部状态是变化的（比如数据库连接对象的状态是正在使用、已回收等待再次使用），通过共享不变的部分，达到减少对象数量从而节约内存的目的。

#### 组成

- 抽象享元：定义行为的一个接口或抽象类
- 具体享元：实现抽象享元的具体行为
- 享元工厂：维护享元对象池

#### 应用场景

- 系统底层开发，以便解决性能问题
- 系统中有大量相似对象，需要缓冲池

#### 在源码中的应用

- String

  参考[String常量池](https://bukeyan.com.cn/other/StringConstantPool.html)

- Integer

  valueOf()做了一个判断，如果入参的值在-128至127之间，则从缓存中取值，否则新建对象，因为这个数据范围内数字是使用最频繁的，为了节省频繁创建对象带来的内存消耗，这里用享元模式来提高性能，同理，Long也是。
  ```java
  private static class IntegerCache {
        static final int low = -128;
        static final int high;
        static final Integer cache[];

        static {
            // high value may be configured by property
            int h = 127;
            String integerCacheHighPropValue =
                sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
            if (integerCacheHighPropValue != null) {
                try {
                    int i = parseInt(integerCacheHighPropValue);
                    i = Math.max(i, 127);
                    // Maximum array size is Integer.MAX_VALUE
                    h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
                } catch( NumberFormatException nfe) {
                    // If the property cannot be parsed into an int, ignore it.
                }
            }
            high = h;

            cache = new Integer[(high - low) + 1];
            int j = low;
            for(int k = 0; k < cache.length; k++)
                cache[k] = new Integer(j++);

            // range [-128, 127] must be interned (JLS7 5.1.7)
            assert IntegerCache.high >= 127;
        }

      private IntegerCache() {}
  }

  public static Integer valueOf(int i) {
      if (i >= IntegerCache.low && i <= IntegerCache.high)
          return IntegerCache.cache[i + (-IntegerCache.low)];
      return new Integer(i);
  }
  ```
