根据[Google Java Style Guide](https://google.github.io/styleguide/javaguide.html) 、 [Android Code Style](http://source.android.com/source/code-style.html)、[Kotlin 语言编码规范](https://www.kotlincn.net/docs/reference/coding-conventions.html)，结合我公司实际场景，和Android特有规范定制。包含 Java/Kotlin 代码风格、规范及Android 编程建议。

核心还是减少沟通成本，提升我们的 Code Review 效率，让我们的代码更加易于维护。此外，一个一致的代码规范可以造成更少的 bug，也就意味着更节省时间和金钱。

---

<br/>

**【 目录 】**

<!-- TOC -->

- [1. 源文件基本规范](#1-源文件基本规范)
  - [1.1. 文件名](#11-文件名)
  - [1.2. 文件编码](#12-文件编码)
  - [1.3. 特殊字符](#13-特殊字符)
    - [1.3.1. 空格字符](#131-空格字符)
    - [1.3.2. 特殊字符转义](#132-特殊字符转义)
- [2. 源文件结构](#2-源文件结构)
  - [2.1. import语句](#21-import语句)
  - [2.2. 唯一顶级class](#22-唯一顶级class)
  - [2.3. 类成员顺序](#23-类成员顺序)
  - [2.4. 重载方法：不应该分开](#24-重载方法不应该分开)
- [3. 代码样式规范](#3-代码样式规范)
  - [3.1. 使用标准大括号样式](#31-使用标准大括号样式)
  - [3.2. 编写简短方法](#32-编写简短方法)
  - [3.3. 类成员的顺序](#33-类成员的顺序)
  - [3.4. 函数参数的排序](#34-函数参数的排序)
  - [3.5. 字符串常量的命名和值](#35-字符串常量的命名和值)
  - [3.6. 行长限制](#36-行长限制)
    - [3.6.1. 操作符的换行](#361-操作符的换行)
    - [3.6.2. 函数链的换行](#362-函数链的换行)
    - [3.6.3. 多参数的换行](#363-多参数的换行)
    - [3.6.4. 控制流语句的换行](#364-控制流语句的换行)
    - [3.6.5. RxJava 链式的换行](#365-rxjava-链式的换行)
  - [3.7. 空格代替 tab](#37-空格代替-tab)
  - [3.8. 横向空白](#38-横向空白)
    - [3.8.1. 留空格](#381-留空格)
    - [3.8.2. 不留空格](#382-不留空格)
  - [3.9. Kotlin 规范](#39-kotlin-规范)
    - [3.9.1. 空格](#391-空格)
    - [3.9.2. 分号](#392-分号)
    - [3.9.3. 类头格式化](#393-类头格式化)
    - [3.9.4. 函数格式化](#394-函数格式化)
    - [3.9.5. 换行](#395-换行)
    - [3.9.6. 修饰符顺序](#396-修饰符顺序)
- [4. 命名](#4-命名)
  - [4.1. 包名](#41-包名)
  - [4.2. 类名](#42-类名)
  - [4.3. 方法名](#43-方法名)
  - [4.4. 常量名](#44-常量名)
  - [4.5. 变量名](#45-变量名)
  - [4.6. 资源名](#46-资源名)
    - [4.6.1. 动画（anim/ 和 animator/）](#461-动画anim-和-animator)
    - [4.6.2. 颜色（color/）](#462-颜色color)
    - [4.6.3. 图片（drawable/ 和 mipmap/）](#463-图片drawable-和-mipmap)
    - [4.6.4. 布局（layout/）](#464-布局layout)
    - [4.6.5. 布局资源 id](#465-布局资源-id)
    - [4.6.6. 菜单（menu/）](#466-菜单menu)
    - [4.6.7. colors.xml](#467-colorsxml)
    - [4.6.8. strings.xml](#468-stringsxml)
    - [4.6.9. styles.xml](#469-stylesxml)
- [5. 编程实践](#5-编程实践)
  - [5.1. Kotlin 优先使用不可变数据](#51-kotlin-优先使用不可变数据)
  - [5.2. Kotlin lambda 单个参数隐式名称](#52-kotlin-lambda-单个参数隐式名称)
  - [5.3. Kotlin 默认参数值](#53-kotlin-默认参数值)
  - [5.4. @override](#54-override)
  - [5.5. 异常捕获不应该被忽略](#55-异常捕获不应该被忽略)
  - [5.6. 静态成员的访问](#56-静态成员的访问)
  - [5.7. 控制修饰符的级别](#57-控制修饰符的级别)
  - [5.8. 使用合适级别的Log信息](#58-使用合适级别的log信息)
  - [5.9. 明确方法功能](#59-明确方法功能)
  - [5.10. 函数参数合法性检查](#510-函数参数合法性检查)
  - [5.11. 类功能明确实现精确](#511-类功能明确实现精确)
  - [5.12. 数据库、IO操作](#512-数据库io操作)
  - [5.13. 抛出异常](#513-抛出异常)
    - [5.13.1. 异常信息](#5131-异常信息)
    - [5.13.2. 异常使用](#5132-异常使用)
  - [5.14. 代码的调试Log](#514-代码的调试log)
  - [5.15. 魔鬼数字](#515-魔鬼数字)
  - [5.16. 不要使用难懂的语句](#516-不要使用难懂的语句)
  - [5.17. 重复代码段函数实现](#517-重复代码段函数实现)
- [6. Android 的注释规范](#6-android-的注释规范)
  - [6.1. 类注释](#61-类注释)
  - [6.2. 方法注释](#62-方法注释)
  - [6.3. 块注释](#63-块注释)
  - [6.4. 全局变量的注释](#64-全局变量的注释)
  - [6.5. 其他一些注释](#65-其他一些注释)
  - [6.6. 注释必须遵守的规范](#66-注释必须遵守的规范)

<!-- /TOC -->

---

<br/>

# 1. 源文件基本规范

## 1.1. 文件名

源码文件名由它所包含的顶级class的类名（大小写敏感），加上.java后缀组成。（除了package-info.java文件）。

## 1.2. 文件编码

源码文件使用UTF-8编码。

## 1.3. 特殊字符

### 1.3.1. 空格字符

除了换行符外，ASCII水平空白字符（0x20）是源码文件中唯一支持的空格字符。这意味着：

- 其他空白字符将被转义。
- Tab字符不被用作缩进控制。

可以在AS中配置Tab自动替换为空格，路径为：`Code Style->Java->Tabs and Indents->Use tab character`（不勾选）

### 1.3.2. 特殊字符转义

特殊字符使用类似：\b, \t, \n, \f, \r, \', \\，而不是八进制数（例如 \012）或Unicode码（例如 \u000a）。

<br/>

# 2. 源文件结构

源码文件按照先后顺序，由以下几部分组成：

- License或者copyright声明信息。（如果需要声明）
- 包声明语句。
- import语句。
- class类声明（每个源码文件只能有唯一一个顶级class）。

每个部分之间应该只有一行空行作为间隔。

## 2.1. import语句

没有行长度的限制。（后面会有行长度规定，一般为100）

## 2.2. 唯一顶级class

每个源码文件中只能有一个顶级class。package-info.java文件除外。

## 2.3. 类成员顺序

类成员的顺序对代码的易读性有很大影响，但是没有一个统一正确的标准。不同的类可能有不同的排序方式。重要的是，每个class都要按照一定的逻辑规律排序。当被问及时，能够解释清楚为什么这样排序。

## 2.4. 重载方法：不应该分开

当一个类有多个构造函数，或者多个同名成员方法时，这些函数应该写在一起，不应该被其他成员分开。

<br/>

# 3. 代码样式规范

## 3.1. 使用标准大括号样式

左大括号不单独占一行，与其前面的代码位于同一行：

```java
class MyClass {
    int func() {
        if (something) {
            // ...
        } else if (somethingElse) {
            // ...
        } else {
            // ...
        }
    }
}
```

我们需要在条件语句周围添加大括号。例外情况：如果整个条件语句（条件和主体）适合放在同一行，那么您可以（但不是必须）将其全部放在一行上。例如，我们接受以下样式：

```java
if (condition) {
    body();
}
```

同样也接受以下样式：

```java
if (condition) body();
```

但不接受以下样式：

```java
if (condition)
    body();  // 不良!
```

## 3.2. 编写简短方法

在可行的情况下，尽量编写短小精炼的方法。我们了解，有些情况下较长的方法是恰当的，因此对方法的代码长度没有做出硬性限制。如果某个方法的代码超出 40 行，请考虑是否可以在不破坏程序结构的前提下对其拆解。

## 3.3. 类成员的顺序

这并没有唯一的正确解决方案，但如果都使用一致的顺序将会提高代码的可读性，推荐使用如下排序：

1. 常量（Kotlin 伴生对象放在开头）
2. 字段
3. 构造函数
4. 重写函数和回调
5. 公有函数
6. 私有函数
7. 内部类或接口

例如：

```java
public class MainActivity extends Activity {

    private static final String TAG = MainActivity.class.getSimpleName();

    private String mTitle;
    private TextView mTextViewTitle;

    @Override
    public void onCreate() {
        ...
    }

    public void setTitle(String title) {
    	mTitle = title;
    }

    private void setUpView() {
        ...
    }

    static class AnInnerClass {

    }
}
```

如果类继承于 Android 组件（例如 `Activity` 或 `Fragment`），那么把重写函数按照他们的生命周期进行排序是一个非常好的习惯，例如，`Activity` 实现了 `onCreate()`、`onDestroy()`、`onPause()`、`onResume()`，它的正确排序如下所示：

```java
public class MainActivity extends Activity {
    //Order matches Activity lifecycle
    @Override
    public void onCreate() {}

    @Override
    public void onResume() {}

    @Override
    public void onPause() {}

    @Override
    public void onDestroy() {}
}
```

## 3.4. 函数参数的排序

在 Android 开发过程中，`Context` 在函数参数中是再常见不过的了，我们最好把 `Context` 作为其第一个参数。

正相反，我们把回调接口应该作为其最后一个参数。

例如：

```java
// Context always goes first
public User loadUser(Context context, int userId);

// Callbacks always go last
public void loadUserAsync(Context context, int userId, UserCallback callback);
```

## 3.5. 字符串常量的命名和值

Android SDK 中的很多类都用到了键值对函数，比如 `SharedPreferences`、`Bundle`、`Intent`，所以，即便是一个小应用，我们最终也不得不编写大量的字符串常量。

当时用到这些类的时候，我们 **必须** 将它们的键定义为 `static final` 字段，并遵循以下指示作为前缀。

| 类                 | 字段名前缀 |
| ------------------ | ---------- |
| SharedPreferences  | PREF_      |
| Bundle             | BUNDLE_    |
| Fragment Arguments | RGUMENT_   |
| Intent Extra       | EXTRA_     |
| Intent Action      | ACTION_    |

说明：虽然 `Fragment.getArguments()` 得到的也是 `Bundle` ，但因为这是 `Bundle` 的常用用法，所以特意为此定义一个不同的前缀。

例如：

```java
// 注意：字段的值与名称相同以避免重复问题
static final String PREF_EMAIL = "PREF_EMAIL";
static final String BUNDLE_AGE = "BUNDLE_AGE";
static final String ARGUMENT_USER_ID = "ARGUMENT_USER_ID";

// 与意图相关的项使用完整的包名作为值的前缀
static final String EXTRA_SURNAME = "com.myapp.extras.EXTRA_SURNAME";
static final String ACTION_OPEN_USER = "com.myapp.action.ACTION_OPEN_USER";
```

## 3.6. 行长限制

代码中每一行文本的长度都应该不超过 160 个字符。虽然关于此规则存在很多争论，但最终决定仍是以 160 个字符为上限，如果行长超过了 160（AS 窗口右侧的竖线就是设置的行宽末尾 ），我们通常有两种方法来缩减行长。

- 提取一个局部变量或方法（最好）。
- 使用换行符将一行换成多行。

不过存在以下例外情况：

- 如果备注行包含长度超过 160 个字符的示例命令或文字网址，那么为了便于剪切和粘贴，该行可以超过 160 个字符。
- 导入语句行可以超出此限制，因为用户很少会看到它们（这也简化了工具编写流程）。

### 3.6.1. 操作符的换行

除赋值操作符之外，我们把换行符放在操作符之前，例如：

```java
int longName = anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne
        + theFinalOne;
```

赋值操作符的换行我们放在其后，例如：

```java
int longName =
        anotherVeryLongVariable + anEvenLongerOne - thisRidiculousLongOne + theFinalOne;
```

### 3.6.2. 函数链的换行

当同一行中调用多个函数时（比如使用构建器时），对每个函数的调用应该在新的一行中，我们把换行符插入在 `.` 之前。

例如：

```java
Picasso.with(context).load("https://blankj.com/images/avatar.jpg").into(ivAvatar);
```

我们应该使用如下规则：

```java
Picasso.with(context)
        .load("https://blankj.com/images/avatar.jpg")
        .into(ivAvatar);
```

### 3.6.3. 多参数的换行

当一个方法有很多参数或者参数很长的时候，我们应该在每个 `,` 后面进行换行。

比如：

```java
loadPicture(context, "https://blankj.com/images/avatar.jpg", ivAvatar, "Avatar of the user", clickListener);
```

我们应该使用如下规则：

```java
loadPicture(context,
        "https://blankj.com/images/avatar.jpg",
        ivAvatar,
        "Avatar of the user",
        clickListener);
```

### 3.6.4. 控制流语句的换行

如果 `if` 语句的条件有多行，那么在语句体外边总是使用大括号。 将该条件的每个后续行相对于条件语句起始处缩进 4 个空格。 将该条件的右圆括号与左花括号放在单独一行：

```java
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module);
}
```

将 `else`、 `catch`、 `finally` 关键字以及 do/while 循环的 `while` 关键字与之前的花括号放在相同的行上：

```java
if (condition) {
    // 主体
} else {
    // else 部分
}

try {
    // 主体
} finally {
    // 清理
}
```

### 3.6.5. RxJava 链式的换行

RxJava 的每个操作符都需要换新行，并且把换行符插入在 `.` 之前。

例如：

```java
public Observable<Location> syncLocations() {
    return mDatabaseHelper.getAllLocations()
            .concatMap(new Func1<Location, Observable<? extends Location>>() {
                @Override
                 public Observable<? extends Location> call(Location location) {
                     return mRetrofitService.getLocation(location.id);
                 }
            })
            .retry(new Func2<Integer, Throwable, Boolean>() {
                 @Override
                 public Boolean call(Integer numRetries, Throwable throwable) {
                     return throwable instanceof RetrofitError;
                 }
            });
}
```

## 3.7. 空格代替 tab

使用 4 个空格缩进。不要使用 tab。

## 3.8. 横向空白

### 3.8.1. 留空格

- 在二元操作符左右留空格（`a + b`） 。
- 在控制流关键字（`if`、 `when`、 `for` 以及 `while`）与相应的左括号之间留空格。
- 在 `//` 之后留一个空格：`// 这是一条注释`

### 3.8.2. 不留空格

- 不在一元运算符左右留空格（`a++`）。
- 不在主构造函数声明、方法声明或者方法调用的左括号之前留空格。
- 不在 `(`、 `[` 之后或者 `]`、 `)` 之前留空格。
- 不在`.` 左右留空格。
- 不在用于指定类型参数的尖括号前后留空格（`class Map<K, V> { …… }`）

## 3.9. Kotlin 规范

> https://www.kotlincn.net/docs/reference/coding-conventions.html

### 3.9.1. 空格

- 在 `:` 之后留一个空格。
- 不在“range to”操作符（`0..i`）左右留空格。
- 不在 `?.` 左右留空格。
- 不在用于标记可空类型的 `?` 前留空格（`String?`）

### 3.9.2. 分号

在 Kotlin 中，分号是可选的，**推荐不写分号**，因此换行很重要。

```kotlin
foo();	// 不良
foo()	// 良好
```

### 3.9.3. 类头格式化

具有少数主构造函数参数的类可以写成一行：

```kotlin
class Person(id: Int, name: String)
```

具有较长类头的类应该格式化，以使每个主构造函数参数都在带有缩进的独立的行中。 另外，右括号应该位于一个新行上。如果使用了继承，那么超类的构造函数调用或者所实现接口的列表应该与右括号位于同一行：

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name) { /*……*/ }
```

对于多个接口，应该将超类构造函数调用放在首位，然后将每个接口应放在不同的行中：

```kotlin
class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker { /*……*/ }
```

对于具有很长超类型列表的类，在冒号后面换行，并横向对齐所有超类型名：

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {

    fun foo() { /*...*/ }
}
```

为了将类头与类体分隔清楚，当类头很长时，可以在类头后放一空行 （如上例所示）或者将左花括号放在独立行上：

```kotlin
class MyFavouriteVeryLongClassHolder :
    MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne 
{
    fun foo() { /*...*/ }
}
```

构造函数参数使用常规缩进（4 个空格）。

> 理由：这确保了在主构造函数中声明的属性与 在类体中声明的属性具有相同的缩进。

### 3.9.4. 函数格式化

如果函数签名不适合单行，请使用以下语法：

```kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType,
): ReturnType {
    // 函数体
}
```

函数参数使用常规缩进（4 个空格）。

> 理由：与构造函数参数一致

对于由单个表达式构成的函数体，优先使用表达式形式。

```kotlin
fun foo(): Int {     // 不良
    return 1 
}
fun foo() = 1        // 良好
```

### 3.9.5. 换行

在 `when` 语句中，如果一个分支不止一行，可以考虑用空行将其与相邻的分支块分开：

```kotlin
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ……
        }
    }
}
```

将短分支放在与条件相同的行上，无需花括号。

```kotlin
when (foo) {
    true -> bar()		// 良好
    false -> { baz() }	// 不良
}
```

当对链式调用换行时，将 `.` 字符或者 `?.` 操作符放在下一行，并带有单倍缩进：

```kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```

调用链的第一个调用通常在换行之前，当然如果能让代码更有意义也可以忽略这点。

### 3.9.6. 修饰符顺序

如果一个声明有多个修饰符，请始终按照以下顺序安放：

```kotlin
public / protected / private / internal
expect / actual
final / open / abstract / sealed / const
external
override
lateinit
tailrec
vararg
suspend
inner
enum / annotation / fun // 在 `fun interface` 中是修饰符
companion
inline
infix
operator
data
```

<br/>

# 4. 命名

## 4.1. 包名

Android 里面有 package 的概念，所以需要约定一下包名命名规范。

包名全部小写，不允许出现中文、大写字母或者下划线，前面为子模块命名，再根据 PBF 方式进行命名 `org.example.project` 。

通常不鼓励使用多个词的名称，但是如果确实需要使用多个词，可以将它们连接在一起或使用驼峰风格 `org.example.myProject` 。

## 4.2. 类名

类名都以 `UpperCamelCase` 风格编写。

类名通常是名词或名词短语，接口名称有时可能是形容词或形容词短语。现在还没有特定的规则或行之有效的约定来命名注解类型。

名词，采用大驼峰命名法，尽量避免缩写，除非该缩写是众所周知的， 比如 HTML、URL，如果类名称中包含单词缩写，则单词缩写的每个字母均应大写。

| 类                   | 描述                              | 例如                                                         |
| -------------------- | --------------------------------- | ------------------------------------------------------------ |
| Activity 类          | 模块名 + `Activity`               | 闪屏页类 `SplashActivity`                                    |
| Fragment 类          | 模块名 + `Fragment`               | 主页类 `HomeFragment`                                        |
| Service 类           | 模块名 + `Service`                | 时间服务 `TimeService`                                       |
| BroadcastReceiver 类 | 功能名 + `Receiver`               | 推送接收 `JPushReceiver`                                     |
| ContentProvider 类   | 功能名 + `Provider`               | `ShareProvider`                                              |
| 自定义 View          | 功能名 + View/ViewGroup(组件名称) | `ShapeButton`                                                |
| Dialog 对话框        | 功能名 + `Dialog`                 | `ImagePickerDialog`                                          |
| Adapter 类           | 模块名 + `Adapter`                | 课程详情适配器 `LessonDetailAdapter`                         |
| 解析类               | 功能名 + `Parser`                 | 首页解析类 `HomePosterParser`                                |
| 工具方法类           | 功能名 + `Utils` 或 `Manager`     | 线程池管理类：`ThreadPoolManager` 日志工具类：`LogUtils`（`Logger` 也可） 打印工具类：`PrinterUtils` |
| 数据库类             | 功能名 + `DBHelper`               | 新闻数据库：`NewsDBHelper`                                   |
| 自定义的共享基础类   | `Base` + 基础                     | `BaseActivity`, `BaseFragment`                               |
| 抽象类               | `Base` / `Abstract` 开头          | `AbstractLogin`                                              |
| 异常类               | `Exception` 结尾                  | `LoginException`                                             |
| 接口                 | `able` / `ible` 结尾 / I 开头     | `Runnable`, `Accessible` ，`ILoginView`                      |

测试类的命名以它要测试的类的名称开始，以 Test 结束。例如：`HashTest` 或 `HashIntegrationTest`。

接口（interface）：命名规则与类一样采用大驼峰命名法，多以 able 或 ible 结尾，如 `interface Runnable`、`interface Accessible`。

>  注意：如果项目采用 MVP，所有 Model、View、Presenter 的接口都以 I 为前缀，不加后缀，其他的接口采用上述命名规则。

## 4.3. 方法名

方法命名，采用以小写字母开头的大小写字符间隔的方式（lowerCamelCase）。

方法命名一般使用动词或者动词短语。

| 方法                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| `initXX()`                  | 初始化相关方法，使用 init 为前缀标识，如初始化布局 `initView()` |
| `isXX()`, `checkXX()`       | 方法返回值为 boolean 型的请使用 is/check 为前缀标识          |
| `getXX()`                   | 返回某个值的方法，使用 get 为前缀标识                        |
| `setXX()`                   | 设置某个属性值                                               |
| `handleXX()`, `processXX()` | 对数据进行处理的方法                                         |
| `displayXX()`, `showXX()`   | 弹出提示框和提示信息，使用 display/show 为前缀标识           |
| `updateXX()`                | 更新数据                                                     |
| `saveXX()`, `insertXX()`    | 保存或插入数据                                               |
| `resetXX()`                 | 重置数据                                                     |
| `clearXX()`                 | 清除数据                                                     |
| `removeXX()`, `deleteXX()`  | 移除数据或者视图等，如 `removeView()`                        |
| `drawXX()`                  | 绘制数据或效果相关的，使用 draw 前缀标识                     |

## 4.4. 常量名

常量命名，全部使用大写字符，词与词之间用下划线隔开。（CONSTANCE_CASE）。



## 4.5. 变量名

这里的变量为广义的变量，包括了常量、局部变量、全局变量等，它们的基础规则是：

- 类型需要是名词 / 名词短语；
- 采用 `lowerCamelCase` 风格；

在具体的变量命名时，会根据该变量的类型不同而附加额外的命名规则：

| 类型       | 说明                                                       | 例如                                                       |
| ---------- | ---------------------------------------------------------- | ---------------------------------------------------------- |
| 常量       | 大写 & 下划线隔开，Kotlin 一定要 const val                 | `const val TYPE_NORMAL = 1` `static final TYPE_NORMAL = 1` |
| 临时变量名 | 整型：`i`、`j`、`k`、`m`、`n` ，字符型一般用 `c`、`d`、`e` | `for(int i = 0;i < len; i++)`                              |
| 其他变量   | `lowerCamelCase` 风格即可，私有变量也不要使用 `m` 开头     | `private int tmp;`                                         |
| Kotlin     | 只读变量使用 `val`，可变变量使用 `var`，尽可能使用 `val`   | `var tmp = 0` `val defaultIndex = 0`                       |

## 4.6. 资源名

资源文件命名为全部小写，采用下划线命名法。

### 4.6.1. 动画（anim/ 和 animator/）

安卓主要包含属性动画和视图动画，其视图动画包括补间动画和逐帧动画。属性动画文件需要放在 `res/animator/` 目录下，视图动画文件需放在 `res/anim/` 目录下。命名规则：`{模块名_}逻辑名称`。

说明：`{}` 中的内容为可选，`逻辑名称` 可由多个单词加下划线组成。例如：`refresh_progress.xml`、`market_cart_add.xml`、`market_cart_remove.xml`。

如果是普通的补间动画或者属性动画，可采用：`动画类型_方向` 的命名方式。

例如：

| 名称                | 说明           |
| ------------------- | -------------- |
| `fade_in`           | 淡入           |
| `fade_out`          | 淡出           |
| `push_down_in`      | 从下方推入     |
| `push_down_out`     | 从下方推出     |
| `push_left`         | 推向左方       |
| `slide_in_from_top` | 从头部滑动进入 |
| `zoom_enter`        | 变形进入       |
| `slide_in`          | 滑动进入       |
| `shrink_to_middle`  | 中间缩小       |

### 4.6.2. 颜色（color/）

`color/` 是专门用于存放颜色相关资源的文件夹。命名规则：`类型{_模块名}_逻辑名称`。

说明：`{}` 中的内容为可选。例如：`sel_btn_font.xml`。

颜色资源也可以放于 `res/drawable/` 目录，引用时则用 `@drawable` 来引用，但不推荐这么做，最好还是把两者分开。

### 4.6.3. 图片（drawable/ 和 mipmap/）

`res/drawable/` 目录下放的是位图文件（.png、.9.png、.jpg、.gif）或编译为可绘制对象资源子类型的 XML 文件，而 `res/mipmap/` 目录下放的是不同密度的启动图标，所以 `res/mipmap/` 只用于存放启动图标，其余图片资源文件都应该放到 `res/drawable/` 目录下。

命名规则：`类型{_模块名}_逻辑名称`、`类型{_模块名}_颜色`。

说明：`{}` 中的内容为可选；`类型` 可以是可绘制对象资源类型，也可以是控件类型最后可加后缀 `_small` 表示小图，`_big` 表示大图。

例如：

| 名称                      | 说明                                        |
| ------------------------- | ------------------------------------------- |
| `btn_main_about.png`      | 主页关于按键 `类型_模块名_逻辑名称`         |
| `btn_back.png`            | 返回按键 `类型_逻辑名称`                    |
| `divider_maket_white.png` | 商城白色分割线 `类型_模块名_颜色`           |
| `ic_edit.png`             | 编辑图标 `类型_逻辑名称`                    |
| `bg_main.png`             | 主页背景 `类型_逻辑名称`                    |
| `btn_red.png`             | 红色按键 `类型_颜色`                        |
| `btn_red_big.png`         | 红色大按键 `类型_颜色`                      |
| `ic_avatar_small.png`     | 小头像图标 `类型_逻辑名称`                  |
| `bg_input.png`            | 输入框背景 `类型_逻辑名称`                  |
| `divider_white.png`       | 白色分割线 `类型_颜色`                      |
| `bg_main_head.png`        | 主页头部背景 `类型_模块名_逻辑名称`         |
| `def_search_cell.png`     | 搜索页面默认单元图片 `类型_模块名_逻辑名称` |
| `ic_more_help.png`        | 更多帮助图标 `类型_逻辑名称`                |
| `divider_list_line.png`   | 列表分割线 `类型_逻辑名称`                  |
| `sel_search_ok.xml`       | 搜索界面确认选择器 `类型_模块名_逻辑名称`   |
| `shape_music_ring.xml`    | 音乐界面环形形状 `类型_模块名_逻辑名称`     |

如果有多种形态，如按钮选择器：`sel_btn_xx.xml`，采用如下命名：

| 名称                    | 说明                                |
| ----------------------- | ----------------------------------- |
| `sel_btn_xx`            | 作用在 `btn_xx` 上的 `selector`     |
| `btn_xx_normal`         | 默认状态效果                        |
| `btn_xx_pressed`        | `state_pressed` 点击效果            |
| `btn_xx_focused`        | `state_focused` 聚焦效果            |
| `btn_xx_disabled`       | `state_enabled` 不可用效果          |
| `btn_xx_checked`        | `state_checked` 选中效果            |
| `btn_xx_selected`       | `state_selected` 选中效果           |
| `btn_xx_hovered`        | `state_hovered` 悬停效果            |
| `btn_xx_checkable`      | `state_checkable` 可选效果          |
| `btn_xx_activated`      | `state_activated` 激活效果          |
| `btn_xx_window_focused` | `state_window_focused` 窗口聚焦效果 |

注意：使用 Android Studio 的插件 SelectorChapek 可以快速生成 selector，前提是命名要规范。

### 4.6.4. 布局（layout/）

命名规则：`类型_模块名`、`{模块名_}类型_逻辑名称`。(也采用 PBF，方便查看，尤其在大项目中)

说明：`{}` 中的内容为可选。

例如：

| 类型               | 名称                 | 说明                                    |
| ------------------ | -------------------- | --------------------------------------- |
| `Activity`         | `main_activity.xml`  | 主窗体 `模块名_类型`                    |
| `Fragment`         | `music_fragment.xml` | 音乐片段 `模块名_类型`                  |
| `Dialog`           | `loading_dialog.xml` | 加载对话框 `逻辑名称_类型`              |
| `PopupWindow`      | `info_ppw.xml`       | 信息弹窗（PopupWindow） `逻辑名称_类型` |
| `adapter` 的列表项 | `main_song_item.xml` | 主页歌曲列表项 `模块名_类型_逻辑名称`   |

### 4.6.5. 布局资源 id

命名规则：`{模块名_}_逻辑名_view 缩写（功能）`，例如： `main_search_btn`、`back_btn`。此外，采用 Kotlinx 直接获取布局文件的时候，id 命名采用驼峰样式。

说明：`{}` 中的内容为可选。参考 GoogleSamples Demo：https://github.com/android/architecture-samples

例如：

| 类型               | 规范       | 命名示例          |
| ------------------ | ---------- | ----------------- |
| `TextView`         | `xxx_text` | `user_login_text` |
| `EditText`         | `xxx_edit` | `user_login_edit` |
| `ImageView`        | `xxx_iv`   | `user_login_iv`   |
| `Button`           | `xxx_btn`  | `user_login_btn`  |
| `CheckBox`         | `xxx_cb`   | `user_login_cb`   |
| `GridView`         | `xxx_gv`   | `user_login_gv`   |
| `ListView`         | `xxx_lv`   | `user_login_lv`   |
| `RecyclerView`     | `xxx_rv`   | `user_login_rv`   |
| `RadioButton`      | `xxx_rb`   | `user_login_rb`   |
| `LinearLayout`     | `xxx_ll`   | `user_login_ll`   |
| `RelativeLayout`   | `xxx_rl`   | `user_login_rl`   |
| `FrameLayout`      | `xxx_fl`   | `user_login_fl`   |
| `GridLayout`       | `xxx_gl`   | `user_login_gl`   |
| `ConstraintLayout` | `xxx_cl`   | `user_login_cl`   |

### 4.6.6. 菜单（menu/）

菜单相关的资源文件应放在该目录下。命名规则：`{模块名_}逻辑名称`

说明：`{}` 中的内容为可选。例如：`main_drawer.xml`、`navigation.xml`。

### 4.6.7. colors.xml

`<color>` 的 `name` 命名使用下划线命名法，在你的 `colors.xml` 文件中应该只是映射颜色的名称一个 ARGB 值，而没有其它的。不要使用它为不同的按钮来定义 ARGB 值。

例如，不要像下面这样做：

```xml
<resources>
      <color name="button_foreground">#FFFFFF</color>
      <color name="button_background">#2A91BD</color>
      <color name="comment_background_inactive">#5F5F5F</color>
      <color name="comment_background_active">#939393</color>
      <color name="comment_foreground">#FFFFFF</color>
      <color name="comment_foreground_important">#FF9D2F</color>
      ...
      <color name="comment_shadow">#323232</color>
```

使用这种格式，会非常容易重复定义 ARGB 值，而且如果应用要改变基色的话会非常困难。同时，这些定义是跟一些环境关联起来的，如 `button` 或或者`comment`，应该放到一个按钮风格中，而不是在 `colors.xml` 文件中。

相反，应该这样做：

```xml
<resources>
      <!-- grayscale -->
      <color name="white">#FFFFFF</color>
      <color name="gray_light">#DBDBDB</color>
      <color name="gray">#939393</color>
      <color name="gray_dark">#5F5F5F</color>
      <color name="black">#323232</color>

      <!-- basic colors -->
      <color name="green">#27D34D</color>
      <color name="blue">#2A91BD</color>
      <color name="orange">#FF9D2F</color>
      <color name="red">#FF432F</color>
  </resources>
```

向应用设计者那里要这个调色板，名称不需要跟 `"green"`、`"blue"` 等等相同。`"brand_primary"`、`"brand_secondary"`、`"brand_negative"` 这样的名字也是完全可以接受的。像这样规范的颜色很容易修改或重构，会使应用一共使用了多少种不同的颜色变得非常清晰。通常一个具有审美价值的 UI 来说，减少使用颜色的种类是非常重要的。

注意：如果某些颜色和主题有关，那就单独写一个 `colors_theme.xml`。

### 4.6.8. strings.xml

`<string>` 的 `name` 命名使用下划线命名法，采用以下规则：`{模块名_}逻辑名称`，这样方便同一个界面的所有 `string` 都放到一起，方便查找。

| 名称                | 说明           |
| ------------------- | -------------- |
| `main_menu_about`   | 主菜单按键文字 |
| `friend_title`      | 好友模块标题栏 |
| `friend_dialog_del` | 好友删除提示   |
| `login_check_email` | 登录验证       |
| `dialog_title`      | 弹出框标题     |
| `button_ok`         | 确认键         |
| `loading`           | 加载文字       |

### 4.6.9. styles.xml

`<style>` 的 name 命名使用大驼峰命名法，几乎每个项目都需要适当的使用 styles.xml 文件，因为对于一个视图来说，有一个重复的外观是很常见的，将所有的外观细节属性（colors、padding、font）放在 styles.xml 文件中。 在应用中对于大多数文本内容，最起码你应该有一个通用的 styles.xml 文件，例如：

```xml
<style name="ContentText">
    <item name="android:textSize">@dimen/font_normal</item>
    <item name="android:textColor">@color/basic_black</item>
</style>
```

应用到 `TextView` 中：

```xml
<TextView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/price"
    style="@style/ContentText"/>
```

或许你需要为按钮控件做同样的事情，不要停止在那里，将一组相关的和重复 `android:xxxx` 的属性放到一个通用的 `<style>` 中。

<br/>

# 5. 编程实践

## 5.1. Kotlin 优先使用不可变数据

kotlin 语言惯性，优先使用不可变数据，初始化后未修改的局部变量与属性，总是将其声明为 `val` 而不是 `var` 。

```kotlin
var a = 0;	// 不良
val b = 0;	// 良好
```

总是使用不可变集合接口（`Collection`、`List`、 `Set`、 `Map`）来声明无需改变的集合。使用工厂函数创建集合实例时，尽可能选用返回不可变集合类型的函数：

```kotlin
// 不良：使用可变集合类型作为无需改变的值
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { …… }

// 良好：使用不可变集合类型
fun validateValue(actualValue: String, allowedValues: Set<String>) { …… }

// 不良：arrayListOf() 返回 ArrayList<T>，这是一个可变集合类型
val allowedValues = arrayListOf("a", "b", "c")

// 良好：listOf() 返回 List<T>
val allowedValues = listOf("a", "b", "c")
```

## 5.2. Kotlin lambda 单个参数隐式名称

一个 lambda 表达式只有一个参数是很常见的。如果编译器自己可以识别出签名，也可以不用声明唯一的参数并忽略 `->`。 该参数会隐式声明为 `it`：

```kotlin
ints.filter { it > 0 } // 单个参数的隐式名称
```

而在有参数的嵌套 lambda 表达式中，始终应该显式声明参数。

## 5.3. Kotlin 默认参数值

优先声明带有默认参数的函数而不是声明重载函数。

```kotlin
// 不良
fun foo() = foo("a")
fun foo(a: String) { /*……*/ }

// 良好
fun foo(a: String = "a") { /*……*/ }
```

## 5.4. @override

`@override` annotations 只要是符合语法的，都应该使用。（所有覆写的地方都应该声明）

## 5.5. 异常捕获不应该被忽略

一般情况下，`catch` 住的异常不应该被忽略，而是都需要做适当的处理。例如将错误日志打印出来，或者如果认为这种异常不会发生，则应该作为断言异常重新抛出。

如果这个 `catch` 住的异常确实不需要任何处理，也应该通过注释做出说明。例如：

```java
try {
	int i = Integer.parseInt(response);
	return handleNumericResponse(i);
} catch (NumberFormatException ok) {
	// it's not numeric; that's fine, just continue
}
return handleTextResponse(response);
```

## 5.6. 静态成员的访问

当一个静态成员被访问时，应该通过 class 名去访问，而不应该使用这个 class 的具体实例对象。例如：

```java
Foo.aStaticMethod();		// 良好
new Foo().aStaticMethod();	// 不良
somethingThatYieldsAFoo().aStaticMethod();	// very bad
```

## 5.7. 控制修饰符的级别

不是必须使用 `public` 属性的，请使用 `protected` ，不是必须使用 `protected` ，请使用 `private`。

## 5.8. 使用合适级别的Log信息

- ERROR：发生致命错误
- WARNNING：发生严重的意外
- INFORMATION：大多数人感兴趣的信息
- DEBUG：调试使用信息
- VERBOSE：其它任何情况

并且给出 Log 信息的开关，可以一键关闭 Log 输出。

## 5.9. 明确方法功能

精确地实现方法设计，一个函数仅完成一件功能，即使简单功能也应该编写方法实现。

## 5.10. 函数参数合法性检查

应明确规定对接口方法参数的合法性检查应由方法的调用者负责还是由接口方法本身负责，缺省是由方法本身负责。

## 5.11. 类功能明确实现精确

明确类的功能，精确（而不是近似）地实现类的设计。一个类仅实现一组相近的功能。

## 5.12. 数据库、IO操作

数据库操作、IO操作等需要使用结束 `close()` 的对象必须在 `try -catch-finally` 的 `finally` 中 `close()` 。

## 5.13. 抛出异常

### 5.13.1. 异常信息

自己抛出的异常必须要填写详细的描述信息。例如：

```java
throw new IOException("这里的异常写清楚！");
```

### 5.13.2. 异常使用

异常分为运行时异常和非运行时异常，运行时异常一般为程序员自己的错误导致，自己抛出时对应的方法需要声明 `throws` 字句，否则编译失败，非运行时异常一般为运行环境异常，比如受磁盘空间影响等，自己抛出时不需要声明 `throws` 字句。`error` 不需要处理。

## 5.14. 代码的调试Log

不要使用 `System.out` 和 `System.err` 进行打印，应该使用一个包含统一开关的测试类进行统一打印。

## 5.15. 魔鬼数字

魔鬼数字尽量不要出现在代码中，必须用有意义的静态变量来代替且必须给静态变量写详细的注释。

## 5.16. 不要使用难懂的语句

难懂的高技巧语句不等于高效率的程序，实际上程序的效率关键在于算法。

## 5.17. 重复代码段函数实现

如果多段代码重复做同一件事情，那么在方法的划分上可能存在问题。若此段代码各语句之间有实质性关联并且是完成同一件功能的，那么可考虑把此段代码构造成一个新的方法。

<br/>

# 6. Android 的注释规范

## 6.1. 类注释

每个类完成后应该有作者姓名和联系方式的注释，对自己的代码负责。

```java
/**
 * <pre>
 *     author : nanchen
 *     e-mail : xxx@xx
 *     time   : 2021/02/25
 *     desc   : xxxx 描述
 *     version: 1.0
 * </pre>
 */
public class WelcomeActivity {
    ...
}
```

具体可以在 AS 中自己配制，进入 Settings -> Editor -> File and Code Templates -> Includes -> File Header，输入

```java
/**
 * <pre>
 *     author : ${USER}
 *     e-mail : xxx@xx
 *     time   : ${YEAR}/${MONTH}/${DAY}
 *     desc   :
 *     version: 1.0
 * </pre>
 */
```

这样便可在每次新建类的时候自动加上该头注释。

## 6.2. 方法注释

每一个成员方法（包括自定义成员方法、覆盖方法、属性方法）的方法头都必须做方法头注释，在方法前一行输入 `/** + 回车` 或者设置 `Fix doc comment`（Settings -> Keymap -> Fix doc comment）快捷键，AS 便会帮你生成模板，我们只需要补全参数即可，如下所示。**`@param` , `@return` , `@throws` , `@deprecated` 这 4 种标记出现的时候，描述都不能为空。当描述无法在一行中容纳，连续行至少需要再缩进 4 个空格。**

```java
/**
 * Report an accessibility action to this view's parents for delegated processing.
 *
 * <p>Implementations of {@link #performAccessibilityAction(int, Bundle)} may internally
 * call this method to delegate an accessibility action to a supporting parent. If the parent
 * returns true from its
 * {@link ViewParent#onNestedPrePerformAccessibilityAction(View, int, android.os.Bundle)}
 * method this method will return true to signify that the action was consumed.</p>
 *
 * <p>This method is useful for implementing nested scrolling child views. If
 * {@link #isNestedScrollingEnabled()} returns true and the action is a scrolling action
 * a custom view implementation may invoke this method to allow a parent to consume the
 * scroll first. If this method returns true the custom view should skip its own scrolling
 * behavior.</p>
 *
 * @param action Accessibility action to delegate
 * @param arguments Optional action arguments
 * @return true if the action was consumed by a parent
 */
public boolean dispatchNestedPrePerformAccessibilityAction(int action, Bundle arguments) {
    for (ViewParent p = getParent(); p != null; p = p.getParent()) {
        if (p.onNestedPrePerformAccessibilityAction(this, action, arguments)) {
            return true;
        }
    }
    return false;
}
```

## 6.3. 块注释

块注释与其周围的代码在同一缩进级别。它们可以是 `/* ... */` 风格，也可以是 `// ...` 风格（**`//` 后最好带一个空格**）。对于多行的 `/* ... */` 注释，后续行必须从 `*` 开始， 并且与前一行的 `*` 对齐。以下示例注释都是 OK 的。

```java
/*
 * This is okay.
 */

// And so
// is this.

/* Or you can
 * even do this. */
```

注释不要封闭在由星号或其它字符绘制的框架里。

在写多行注释时，如果你希望在必要时能重新换行（即注释像段落风格一样），那么使用 `/* ... */`。

## 6.4. 全局变量的注释

全局变量的注释样式如下（注意注释之间有空格）：

```java
/**
 * The next available accessibility id.
 */
private static int nextAccessibilityViewId;

/**
 * The animation currently associated with this view.
 */
protected Animation currentAnimation = null;
```

## 6.5. 其他一些注释

AS 已帮你集成了一些注释模板，我们只需要直接使用即可，在代码中输入 `TODO`、`FIXME` 等这些注释模板，回车后便会出现如下注释。

```java
// TODO: 17/3/14 需要实现，但目前还未实现的功能的说明
// FIXME: 17/3/14 需要修正，甚至代码是错误的，不能工作，需要修复的说明
```

## 6.6. 注释必须遵守的规范

- 不言自明的方法不要加注释。比如 `Item getItem(int index)` 是一段自说明的代码，我们可以直接从方法的命名就能知道它是干嘛的，所以不需要增加注释。
- 提测的代码不应该有 TODO 注释。

