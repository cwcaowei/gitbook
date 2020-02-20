#### 简单工厂模式

又称静态工厂方法模式，工厂类封装new对象的细节，使用者通过传参给工厂类来获得相应对象。

- 组成：
	- 工厂类角色（一个）：这是本模式的核心，含有一定的商业逻辑和判断逻辑，用来创建产品。
	- 抽象产品角色（一个）：它一般是具体产品继承的父类或者实现的接口。 
	- 具体产品角色（多个）：工厂类所创建的对象就是此角色的实例，在java中由一个具体类实现。 

- 设计原则分析
每增加一种产品，创建一个具体产品角色，产品层面不违背开闭原则，但需要修改工厂类中创建产品的逻辑，工厂层面违背了开闭原则。且随着产品越来越多，工厂职责越来越重，违背了单一职责原则。

如：

```java
// 使用者
Logger LOGGER = LoggerFactory.getLogger(WebLogController.class);

public static Logger getLogger(Class<?> clazz) {
    Logger logger = getLogger(clazz.getName());
    if (DETECT_LOGGER_NAME_MISMATCH) {
      Class<?> autoComputedCallingClass = Util.getCallingClass();
      if (autoComputedCallingClass != null && nonMatchingClasses(clazz, autoComputedCallingClass)) {
        Util.report(String.format("Detected logger name mismatch. Given name: \"%s\"; computed name: \"%s\".", logger.getName(), autoComputedCallingClass.getName()));
        Util.report("See " + LOGGER_NAME_MISMATCH_URL + " for an explanation");
      }
    }
    return logger;
}

public static Logger getLogger(String name) {
    ILoggerFactory iLoggerFactory = getILoggerFactory();
    return iLoggerFactory.getLogger(name);
}
```

```java
// 使用者
Calendar calendar = Calendar.getInstance();

public static Calendar getInstance() {
        return createCalendar(TimeZone.getDefault(), Locale.getDefault(Locale.Category.FORMAT));
}	

private static Calendar createCalendar(TimeZone zone, Locale aLocale) {
    CalendarProvider provider =
      LocaleProviderAdapter.getAdapter(CalendarProvider.class, aLocale)
      .getCalendarProvider();
    if (provider != null) {
      try {
        return provider.getInstance(zone, aLocale);
      } catch (IllegalArgumentException iae) {
        // fall back to the default instantiation
      }
    }

    Calendar cal = null;

    if (aLocale.hasExtensions()) {
      String caltype = aLocale.getUnicodeLocaleType("ca");
      if (caltype != null) {
        switch (caltype) {
          case "buddhist":
            cal = new BuddhistCalendar(zone, aLocale);
            break;
          case "japanese":
            cal = new JapaneseImperialCalendar(zone, aLocale);
            break;
          case "gregory":
            cal = new GregorianCalendar(zone, aLocale);
            break;
        }
      }
    }
    if (cal == null) {
      // If no known calendar type is explicitly specified,
      // perform the traditional way to create a Calendar:
      // create a BuddhistCalendar for th_TH locale,
      // a JapaneseImperialCalendar for ja_JP_JP locale, or
      // a GregorianCalendar for any other locales.
      // NOTE: The language, country and variant strings are interned.
      if (aLocale.getLanguage() == "th" && aLocale.getCountry() == "TH") {
        cal = new BuddhistCalendar(zone, aLocale);
      } else if (aLocale.getVariant() == "JP" && aLocale.getLanguage() == "ja"
                 && aLocale.getCountry() == "JP") {
        cal = new JapaneseImperialCalendar(zone, aLocale);
      } else {
        cal = new GregorianCalendar(zone, aLocale);
      }
    }
    return cal;
}
```



#### 工厂方法模式

工厂方法模式简单来讲就是简单工厂模式里集中在工厂类上的压力由工厂方法模式里不同的工厂子类来分担。 

- 组成
	- 抽象工厂角色（一个）： 这是本模式的核心，是具体工厂角色必须实现的接口或者必须继承的父类，在java中它由抽象类或者接口来实现。 
	- 具体工厂角色（多个）：它含有和具体业务逻辑有关的代码，由应用程序调用以创建对应的具体产品的对象。 
	- 抽象产品角色（一个）：它是具体产品继承的父类或者是实现的接口，在java中一般由抽象类或者接口来实现。 
	- 具体产品角色（多个）：具体工厂角色所创建的对象就是此角色的实例，在java中由具体的类来实现。 

- 设计原则分析
  当有新的具体产品产生时，只要按照抽象产品角色、抽象工厂角色提供的规则来生成具体产品角色和具体工厂角色，而不必去修改任何已有的代码。可以看出产品和工厂层面都是符合开闭原则的。每个具体工厂角色只负责创建对应的具体产品角色，不违背单一职责原则。

  

#### 抽象工厂模式

多个抽象产品角色，每个抽象产品角色有多个具体产品角色的情况下，由一个具体工厂角色负责创建一系列相关或相互依赖的具体产品角色，而无需创建多个具体工厂角色。

- 组成
  - 抽象工厂角色（一个）： 这是本模式的核心，是具体工厂角色必须实现的接口或者必须继承的父类，在java中它由抽象类或者接口来实现。 
  - 具体工厂角色（多个）：它含有和具体业务逻辑有关的代码，由应用程序调用以创建对应的具体产品的对象。 
  - 抽象产品角色（多个）：它是具体产品继承的父类或者是实现的接口，在java中一般由抽象类或者接口来实现。 
  - 具体产品角色（多个）：具体工厂角色所创建的对象就是此角色的实例，在java中由具体的类来实现。 

比如空调这个抽象产品分为美的空调，海尔空调，格力空调，冰箱这个抽象产品分为美的冰箱，海尔冰箱，格力冰箱，洗衣机这个抽象产品分为美的洗衣机，海尔洗衣机，格力洗衣机，像美的空调，美的冰箱，美的洗衣机都属于美的品牌，那么就由一个美的工厂来负责生产这三个产品，而不需要美的空调工厂，美的冰箱工厂，美的洗衣机工厂3个工厂来处理。

- 设计原则分析

  如果加入了一个新的抽象产品，那么从抽象工厂角色到具体工厂角色都要调整，违背了开闭原则。
