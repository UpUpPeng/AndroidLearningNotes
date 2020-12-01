<!-- TOC -->

- [1. 泛型的概念](#1-泛型的概念)
- [2. 泛型的使用](#2-泛型的使用)
  - [2.1. 泛型类/接口](#21-泛型类接口)
  - [2.2. 泛型方法](#22-泛型方法)
  - [2.3. 通配符](#23-通配符)
  - [2.4. 上下边界限定](#24-上下边界限定)
- [3. 泛型的原理](#3-泛型的原理)
  - [3.1. 类型擦除](#31-类型擦除)
    - [3.1.1. 未指定上界的泛型类型会以`Object`类型替换](#311-未指定上界的泛型类型会以object类型替换)
    - [3.1.2. 已指定上界的泛型类型会以上界类型替换](#312-已指定上界的泛型类型会以上界类型替换)
  - [3.2. 绕过编译时泛型类型检查](#32-绕过编译时泛型类型检查)

<!-- /TOC -->

# 1. 泛型的概念

泛型，即“**参数化类型**”。就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（类型形参），然后在使用/调用时传入具体的类型（类型实参）。

- JDK1.5 之后引入。
- 让代码更通用更灵活。
- 核心目标是解决容器类型在编译时安全检查的问题。

在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别被称为泛型类、泛型接口、泛型方法。

---

# 2. 泛型的使用

> 类型参数（类型变量）用作占位符，可能有一个或多个，作用于整个类。根据惯例类型参数是单个大写字母：
> - E：元素，一般在线性结构集合中使用
> - K：键，一般在键值类型的集合中用于表示键
> - V：值，一般在键值类型的集合中用于表示值
> - N：数字
> - T：类型

## 2.1. 泛型类/接口

`class 类名称<类型参数1, 类型参数2 ...> {}`

`interface 接口名称<类型参数1, 类型参数2 ...> {}`

```java
// 泛型类
public class HashMap<K, V> extends AbstractMap<K, V> implements Map<K, V>, Cloneable, Serializable {}
// 泛型接口
public interface Map<K, V> {}
```

## 2.2. 泛型方法

`类型参数0（返回值类型） 方法名称<类型参数1 参数1, 类型参数2 参数2 ...> {}`

```java
public final V setValue(V newValue) {
    V oldValue = value;
    value = newValue;
    return oldValue;
}
```

**注意**：不能在静态方法、静态初块等静态内容中使用泛型的类型参数。

```
public class A<T> {
    public static void func(T t) {
        //报错，编译不通过
    }
}
```

## 2.3. 通配符

问号通配符 `?` **表示未知类型**。通配符可用于参数、字段、局部变量和返回类型。可以近似的理解为**泛型的泛型**。

- 通配符匹配出来的泛型类型只能读取，不能写入。因为不知道这个容器放什么类型的数据，所以只能读取不能添加。

- 最好不要在返回类型中使用通配符，因为确切知道方法返回的类型更安全。

```java
public static void main(String[] args) {
    List<String> name = new ArrayList<String>();
    List<Integer> age = new ArrayList<Integer>();
    List<Number> number = new ArrayList<Number>();

    name.add("icon");
    age.add(18);
    number.add(314);

    getData(name);   // icon
    getData(age);    // 18
    getData(number); // 314
}

// List<?>，在逻辑上是List<String>，List<Integer>等所有List<具体类型实参>的父类。
public static void getData(List<?> data) {
    System.out.println("data :" + data.get(0));
}
```

## 2.4. 上下边界限定

- 上界限定通配符：`<? extends E>`，表示只接受E类型及其子类型。
- 下界限定通配符：`<? super E>`，    表示只接受E类型及其父类型。

```java
public static void main(String[] args) {
    List<String> name = new ArrayList<String>();
    List<Integer> age = new ArrayList<Integer>();
    List<Number> number = new ArrayList<Number>();

    name.add("icon");
    age.add(18);
    number.add(314);

    getUperNumber(name);   // 编译时报错，String类型不是Number的子类
    getUperNumber(age);    // 18
    getUperNumber(number); // 314
}

public static void getData(List<?> data) {
    System.out.println("data :" + data.get(0));
}

// List<? extends Number>，在逻辑上是List<Number>，List<Integer>等类的父类。
public static void getUperNumber(List<? extends Number> data) {
    System.out.println("data :" + data.get(0));
}
```

# 3. 泛型的原理

Java 语言的泛型采用的是**擦除法**实现的**伪泛型**，泛型信息（类型变量、参数化类型）编译之后通通被除掉了。

## 3.1. 类型擦除

泛型信息只存在于编译前，编译后的字节码中是不包含泛型中的类型信息的。因此，编译器在编译时去掉类型参数，叫做**类型擦除**。

例如 `List<Integer>` 和 `List<String>` 等类型在编译之后都会变成 `List`。JVM看到的只是 `List`，而泛型信息对JVM来说是不可见的。

```java
Class c1 = new ArrayList<String>().getClass();
Class c2 = new ArrayList<Integer>().getClass();
System.out.println(c1 == c2);  // true
```

### 3.1.1. 未指定上界的泛型类型会以`Object`类型替换

```java
// 泛型类java源代码
public class Test<T> {
    private T test;
}
// 编译后的代码
public class Test{
    public Test(){}
    private Object test;  // 原来的泛型T被替换为Object
}
```

### 3.1.2. 已指定上界的泛型类型会以上界类型替换

```java
// 泛型类java源代码
public class Test<T extends String> {
    private T num;
}
// 编译后的代码
public class Test{
    public Test(){}
    private String test;  // 原来的泛型T被替换为上界类型String
}
```

## 3.2. 绕过编译时泛型类型检查

```java
List<Integer> list = new ArrayList<>();
list.add(123);		// 正常编译
list.add("string"); // 编译报错【不兼容的类型: java.lang.String无法转换为java.lang.Integer】
```

基于类型擦除，我们可以利用反射绕过这个限制。

```java
List<Integer> list = new ArrayList<>();
list.add(123);
try {
    // 由于List中的泛型参数没有设置上界，所以add方法可以add任何Object的子类型参数
    Method method = list.getClass().getDeclaredMethod("add", Object.class);
    method.invoke(list, "string");
    method.invoke(list, true);
    method.invoke(list, 45.6);
} catch (Exception e) {
    e.printStackTrace();
}
```