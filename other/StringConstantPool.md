```java
String s1 = "hello";
String s2 = "hello";
String s3 = "he" + "llo";
String s4 = "he" + new String("llo");
String s5 = new String("hello");
String s6 = s5.intern();
String s7 = "he";
String s8 = "llo";
String s9 = s7 + s8;
System.out.println(s1 == s2); // true
// 以字面量的形式创建String变量时，JVM会在编译期间就把该字面量"hello"放到字符串常量池中，后续再以字面量的形式创建"hello"变量时,直接返回该字面量在字符串常量池中的引用，所以s1==s2。
System.out.println(s1 == s3); // true
// "he" + "llo"在编译期间JVM会将其拼接为"hello"，如果常量池中没有，就放入字符串常量池中，否则返回该字面量在字符串常量池中的引用，并不会将"he"和"llo"放入字符串常量池中，所以s1==s3。
System.out.println(s1 == s4); // false
// new String("llo")创建了两个对象，"llo"存于字符串常量池，new String("llo")存于堆，"he" + new String("llo")两个对象的相加编译器不会优化，相加的结果存于堆，所以s1!=s4。
System.out.println(s1 == s9); // false
// s7 + s8两个对象的相加编译器不会优化，相加的结果存于堆，所以s1!=s9
System.out.println(s4 == s5); // false
// s4和s5是堆中两个不同的对象，所以s4!=s5。
```

```java
System.out.println(s1 == s6); // true
// intern()能使一个位于堆中的字符串在运行期间动态的加入到字符串常量池中（字符串常量池的内容是程序启动的时候就加载进内存了），如果字符串常量池中有该对象对应的字面量，就返回该字面量在字符串常量池中的引用，否则在字符串常量池中创建该对象对应的字面量，将该对象的引用作为该字面量在字符串常量池中的引用并返回。

String s10 = new String("hello") + new String("h");
String s11 = s10.intern();
String s12 = "helloh";
System.out.println(s13 == s14);
// 生成了5个对象：字符串常量池中的"hello"和"h",两个在堆中的对象new String("hello")和new String("h")以及它们相加后的对象，intern将"helloh"放入字符串常量池中，并将相加后的对象的引用作为字符串常量池中"helloh"的引用，所以s14和s13是同一个引用，所以s13==s14。
String s13 = new StringBuilder("test").append("this").toString();
System.out.println(s13 == s13.intern()); // true
// s13.intern()将s13的引用作为字面量"testthis"的引用存到了常量池
```
