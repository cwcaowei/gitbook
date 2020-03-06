#### 适用场景

- 创建对象需要很多步骤，但是步骤的顺序不一定固定
- 对象有非常复杂的内部结构（很多属性）
- 对象的很多参数都有默认值


#### 组成

- 产品（Product）：要创建的对象

```java
@Data
public class User {

    private String name;
    private Integer age;
    private String email;
    private String phone;
    private String role;
    private String idCard;
    private String address;
    private String company;

}

```
- 建造者抽象（Builder）：建造者的抽象类，规范产品对象的各个组成部分的建造，由子类实现具体的建造过程

```java
public interface IBuilder<T> {

    T build();

}
```

- 建造者（Concrete Builder）：具体的Builder类，根据不同的业务逻辑，具体化对象的各个组成部分的创建

```java
public class UserBuilder implements IBuilder<User> {

    private User user = new User();

    public UserBuilder name(String name) {
        user.setName(name);
        return this;
    }

    public UserBuilder age(Integer age) {
        user.setAge(age);
        return this;
    }

    ...

    public User build() {
        return user;
    }
}
```

- 调用者（Director）：调用具体的建造者，负责对象各部分完整创建或按某种顺序创建

```java
User user = new UserBuilder()
                    .name("zhangsan")
                    .age(20)
                    .role("admin")
                    .company("ALI")
                    .email("test@test.com.cn")
                    .phone("1811111111")
                    .idCard("32123213123123123")
                    .address("某某路2号")
                    .build();
```
