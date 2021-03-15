# 1. 类与对象

## 1.1. C++ 类定义

定义一个类，本质上是定义一个数据类型，包括了成员和操作。

```c++
class 类名
{
	访问修饰符： private/public/protected
		数据类型 成员变量名;
    	返回值类型 成员函数名(){}
}
```

示例代码：

```c++
#include <iostream>
#include <string>
using namespace std;
// 定义一个User类
class User {
private:	// 以下代码的访问权限为private
    string name;	// 成员变量1
    int age;		// 成员变量2
public:		// 以下代码的访问权限为public
    // 构造函数，对两个成员赋初始值
    User(const string &name, int age) {
        this->name = name;
        this->age = age;
    }
	// 获取成员变量的值
    const string &getName() const {
        return name;
    }
	// 获取成员变量的值
    int getAge() const {
        return age;
    }
};

int main() {
    // 实例化一个User对象，并用一个指针指向该堆内存地址
    User *user = new User("小明", 18);
    // 获取User对象的两个成员变量的值
    cout << user->getName() << " " << user->getAge() << endl;
    cout << sizeof(User) << endl;
    // 释放User对象所占用的堆内存
    delete user;
    return 0;
}
```

`const string &getName() const {}` 函数中，两个 `const` 的含义：

- 第一个 `const` 表示返回值是一个常量。
- 第二个 `const` 表示该函数不会也不允许修改成员变量，该函数称为**常量函数**。

### 1.1.1. 类访问修饰符

类访问修饰符用来封装数据，防止函数直接访问内部成员。

| 修饰符    | 作用                                                         |
| --------- | ------------------------------------------------------------ |
| public    | **公有成员**在类的外部是可访问的，可以直接读写公有成员变量的值 |
| protected | **受保护成员**在派生类（子类）被访问，具有public相同的效果；在其他类中被访问，具有private相同的效果 |
| private   | **私有成员**在类的外部是不可访问的，不可以直接读写公有成员变量的值 |

| 访问修饰符 | 本类 | 派生类 | 其他类 |
| :--------- | :--- | :----- | :----- |
| public     | ⭕️    | ⭕️      | ⭕️      |
| protected  | ⭕️    | ⭕️      | ❌      |
| private    | ⭕️    | ❌      | ❌      |

### 1.1.2. 构造函数 & 析构函数

**构造函数**是类的一种特殊的成员函数，它会在每次创建类的新对象时执行。

构造函数的名称与类的名称是完全相同的，并且不会返回任何类型，也不会返回 void，可用于为某些成员变量设置初始值。

```c++
// 默认构造函数没有参数，也没有函数体。仅用来创建对象
User(){}
// 在构造函数中传参数，可以对成员进行初始化
User(const string &name, int age) {
    this->name = name;
    this->age = age;
}
```

**析构函数**是类的一种特殊的成员函数，它会在每次删除所创建的对象时执行。

析构函数的名称与类的名称是完全相同的，只是在前面加了个波浪号 `~` 作为前缀，它不会返回任何值，也不能带有任何参数。析构函数有助于在跳出程序（比如关闭文件、释放内存等）前释放资源。

```c++
// 在析构函数中释放资源
~User() {
	delete obj;
}
```

### 1.1.3. 静态成员

使用 `static` 关键字定义静态成员，此时全部的类对象，都共享这同一个成员。

```c++
class Test {
public:
    // 声明Test类的静态成员
    static int i;
};
// 初始化Test类的静态成员
int Test::i = 10;

int main() {
    // 输出Test类的静态成员
    cout << " test1.i: " << Test::i << endl;	// 【10】
    return 0;
}
```

## 1.2. C++ 函数

### 1.2.1. 拷贝构造函数

**拷贝构造函数**是一种特殊的构造函数，使用同一类中之前创建的对象来初始化新创建的对象。通常用于：

- 通过使用另一个同类型的对象来初始化新创建的对象。
- 复制对象把它作为参数传递给函数。
- 复制对象，并从函数返回这个对象。

```c++
User(const User &user) {
    this->name = user.name;	// 拷贝里面的值并赋值给成员
    this->age = user.age;	// 拷贝里面的值并赋值给成员
}
```

编译器会默认生成一个拷贝构造函数。但如果类有指针成员变量，并有动态内存分配，则它必须指定一个拷贝构造函数。

### 1.2.2. 友元函数

类的友元函数是定义在类外部，但有权访问类的所有私有成员和保护成员。友元函数需事先在类中声明，但是友元函数并不是成员函数。

```c++
#include <iostream>
#include <string>
using namespace std;

class User {

private:
    string name;
    int age;
public:
    User(const string &name, int age) {
        this->name = name;
        this->age = age;
    }
    // 声明一个友元函数
    friend void setAge(User *user, int age);
   	// 声明一个友元类
    friend class Admin;
};

// 定义友元函数
void setAge(User *user, int age) {
    // 直接访问User类的私有成员
    user->age = age;
}

// 定义友元类
class Admin {
public:
    void setName(User *user, const string &name) {
        // 直接访问User类的私有成员
        user->name = name;
    }
};

int main() {
    User *user = new User("小明", 18);
    cout << user->getName() << " " << user->getAge() << endl;	// 【小明 18】
    
    // 通过友元函数访问私有成员
    setAge(user,20);
    // 通过友元类访问私有成员
    Admin *admin = new Admin();
    admin->setName(user, "小刚");
    
    cout << user->getName() << " " << user->getAge() << endl;	// 【小刚 20】
    
    delete user;
    delete admin;
    return 0;
}
```

### 1.2.3. 内联函数

如果一个函数是内联的，那么在编译时，编译器会把该函数的代码副本放置在每个调用该函数的地方。

内联函数在函数名前面加关键字 `inline`，且在调用之前就需要定义。如果函数体超过一行，编译器则会忽略 inline 修饰符。

```c++
inline int Max(int x, int y) {
    return (x > y) ? x : y;
}

int main() {
    cout << "Max (10,20): " << Max(10, 20) << endl;	// 【20】
    return 0;
}
```

**【优缺点】**

- 减少函数调用的开销。
- 调试信息通常比宏定义更详细，且更安全。
- 可能增大编译时间，以及编译后的体积。

---

<br/>

# 2. 函数重载和运算符重载

## 2.1. 函数重载

在同一个作用域内，可以声明几个同名函数但不同形式参数（指参数的个数、类型或者顺序）的函数，而且不能仅通过返回类型的不同来重载函数。

```c++
#include <iostream>
using namespace std;
 
class printData {
public:
    void print(int i) {
        cout << "整数为: " << i << endl;
    }

    void print(double f) {
        cout << "浮点数为: " << f << endl;
    }
};
 
int main(void) {
   printData pd;
 
   // 输出整数
   pd.print(5);			// 【整数为: 5】
   // 输出浮点数
   pd.print(500.263);	// 【浮点数为: 500.263】
 
   return 0;
}
```

## 2.2. 运算符重载

重载的运算符是带有特殊名称的函数，函数名是由关键字 `operator` 和其后要重载的运算符符号构成的。与其他函数一样，重载运算符有一个返回类型和一个参数列表。

```c++
// 重载+运算符，用于把两个User对象相加
User operator+(const User &user) {
    User userResult;
    userResult.name = this->name + user.name;
    userResult.age = this->age + user.age;
    return userResult;
}

int main() {
    User *user1 = new User("小明", 18);
    User *user2 = new User("小刚", 20);
    // 利用重载后的+运算符，把两个对象相加
    User user3 = *user1 + *user2;
    cout << user3.getName() << " " << user3.getAge() << endl;	// 【小明小刚 38】

    return 0;
}
```

### 2.2.1. 可重载运算符

| 运算符 | 运算 |
| -------------- | ------------------------------------------------------------ |
| 双目算术运算符 | `+` 加，`-` 减，`*` 乘，`/` 除，`%` 取模     |
| 关系运算符     | `==` 等于，`!=` 不等于，`<` 小于，`>` 大于，`<=` 小于等于，`>=` 大于等于 |
| 逻辑运算符     | `||` 逻辑或，`&&` 逻辑与，`!` 逻辑非                 |
| 单目运算符     | `+` 正，`-` 负，`*` 指针，`&` 取地址               |
| 自增自减运算符 | `++` 自增，`--` 自减                                     |
| 位运算符       | `|` 按位或，`&` 按位与，`~` 按位取反，`^` 按位异或，`<<` 左移，`>>` 右移 |
| 赋值运算符     | `=` ，`+=` ，`-=` ，`*=` ，`/=` ，`%=` ，`&=` ，`|=` ，`^=` ，`<<=` ，`>>=` |
| 空间申请与释放 | `new` ，`delete` ，`new[]` ，`delete[]`                     |
| 其他运算符     | `()` 函数调用，`->` 成员访问，`,` 逗号，`[]` 下标 |

### 2.2.2. 不可重载运算符

| 运算符             | 实例       |
| ------------------ | ---------- |
| 成员访问运算符     | `.`        |
| 成员指针访问运算符 | `.` ，`->` |
| 域运算符           | `::`       |
| 长度运算符         | `sizeof`   |
| 条件运算符         | `?:`       |
| 预处理符号         | `#`        |

---

<br/>

# 3. RVO 与 NRVO

**RVO（return value optimization）** 和 **NRVO（named return value optimization）** ，是一种编译器优化技术。为了减少返回非引用对象在调用处创建的临时对象，同时减少**拷贝构造**次数以及**析构**次数。

先来看一个现象：

```c++
#include <iostream>
#include <string>

using namespace std;

class Test {
public:
    string v;

    Test add(const Test &s) const {
        Test tmp;
        tmp.v = this->v + s.v;
        return tmp;
    }

};

int main() {

    Test t1;
    t1.v = "Hello";
    Test t2;
    t2.v = "World";

    Test t3 = t1.add(t2);
    cout << t3.v << endl;

    return 0;
}
```

运行结果：

> HelloWorld

⭐️ **【注意】**

`Test tmp;` 是一个局部变量，在函数运行结束后，就会被回收，导致 `Test t3` 指向的内容异常。但事实上运行结果正确，原因如下：

- 在函数返回时，会把栈中的 `tmp` 对象通过拷贝构造函数**拷贝**给一个**临时对象**（我们不可见），再调用析构函数回收 `tmp` 对象。
- `Test t3 = t1.add(t2);` 中，在 `=` 号赋值时， 会把该**临时对象**，再次通过拷贝构造函数**拷贝**给 `t3` ，使得结果被正确保留。

综上，编译器通过**两次拷贝**，把原本位于栈中应当销毁的局部变量保留了下来，使得运算结果正确。

但是，不同的编译器，对拷贝次数有不同的优化。

- VS 环境中，debug 模式下会执行1次拷贝（**RVO**），release 环境下会执行0次拷贝（**NRVO**）。
- Clion 环境中，会执行0次拷贝（**NRVO**）。
- Xcode 环境中，会执行0次拷贝（**NRVO**）。

## 3.1. RVO

RVO 优化会在编译阶段，把函数的返回值对象作为参数传递进来，并把函数返回值改为 **void** 。

原函数和调用方式：

```c++
// 函数有一个参数，有返回值
Test add(const Test &s) const {
    Test tmp;
    tmp.v = this->v + s.v;
    return tmp;
}
// 调用方式
Test t3 = t1.add(t2);
```

经过 RVO 优化后的伪代码（实际更复杂，伪代码便于理解）：

```c++
// 返回值被作为参数传递进来，函数也就不需要返回值了
void add(Test &result, const Test &t) const {
    Test tmp;
    tmp.v = this->v + t.v;
    result = tmp;
}
// 调用方式
Test t3;
t1.add(t3, t2);
```

RVO 优化后，只需要1次拷贝（`tmp` 对象拷贝到 `result`）就能实现。

## 3.2. NRVO

NRVO 会更进一步，直接连都不创建了。

经过 NRVO 优化后的伪代码（实际更复杂，伪代码便于理解）：

```c++
// 直接在返回参数里面修改，不用创建临时变量，函数也不需要返回值
void add(Test &result, const Test &t1, const Test &t2) const {
    result.v = t1.v + t2.v;
}
// 调用方式
Test t3;
t1.add(t3, t1, t2);
```

RVO 优化后，不需要拷贝就能实现。

---

<br/>

# 4. 封装

封装是面向对象编程中的把数据和操作数据的函数绑定在一起的一个概念，这样能避免受到外界的干扰和误用，从而确保了安全。

```c++
#include <iostream>
using namespace std;

class Adder {
public:
    // 构造函数
    Adder(int i = 0) {
        total = i;
    }

    // 对外的接口
    void addNum(int number) {
        total += number;
    }

    // 对外的接口
    int getTotal() {
        return total;
    };
private:
    // 对外隐藏的数据
    int total;
};

int main() {
    Adder a;

    a.addNum(10);
    a.addNum(20);
    a.addNum(30);

    cout << "Total " << a.getTotal() << endl;
    return 0;
}
```

---

<br/>

# 5. 继承

继承是依据一个类来定义一个新的类，提高代码复用性和执行效率。

作为依据的已有类称为**基类（父类）**，新创建的类称为**派生类（子类）**。

![](https://picture-1251081707.cos.ap-shanghai.myqcloud.com/20210309-113825-7a9cd7bdbdf420436101a7e44263d26d.png)

```c++
// 基类
class Animal {
    // eat() 函数
    // sleep() 函数
};


// 派生类(公有继承)
class Dog : public Animal {
    // bark() 函数
};
```

## 5.1. 继承类型

- **公有继承（public）：**基类的**公有**成员也是派生类的**公有**成员，基类的**保护**成员也是派生类的**保护**成员，基类的**私有**成员不能直接被派生类访问，但是可以通过调用基类的**公有**和**保护**成员来访问。
- **保护继承（protected）：** 基类的**公有**和**保护**成员将成为派生类的**保护**成员。
- **私有继承（private）：**基类的**公有**和**保护**成员将成为派生类的**私有**成员。

通常使用 `public` 继承，几乎不使用 `protected` 或 `private` 继承，但如果不写明继承类型，默认为 `private` 继承。

## 5.2. 多继承

多继承即一个子类可以有多个父类，它继承了多个父类的特性。

代码结构：

```c++
class 派生类类名 : 继承类型1 基类名1, 继承类型2 基类名2,
{
	派生类类体
};
```

多了基类的继承类型可以单独指定，不一样的完全一致。

## 5.3. 基类 & 派生类

一个派生类继承了所有的基类方法，但除去以下内容：

- 基类的构造函数、析构函数和拷贝构造函数。
- 基类的重载运算符。
- 基类的友元函数。

```c++
#include <iostream>
using namespace std;

// 基类
class Shape {
public:
    void setWidth(int w) {
        width = w;
    }

    void setHeight(int h) {
        height = h;
    }

protected:
    int width;
    int height;
};

// 派生类
class Rectangle : public Shape {
public:
    int getArea() {
        // 使用基类的两个成员计算面积
        return (Shape::width * Shape::height);
    }
};

int main() {
    // 创建派生类对象
    Rectangle rect;
	// 使用派生类对象调用基类的函数
    rect.setWidth(5);
    rect.setHeight(7);

    // 输出对象的面积
    cout << "面积: " << rect.getArea() << endl;

    return 0;
}
```

---

<br/>

# 6. 多态

调用成员函数时，会根据调用函数的对象的类型来执行不同的函数。

## 6.1. 静态多态

**静态多态（静态链接）**是指函数调用在程序执行前就准备好了，即**只关心指针的类型，而不关心指针指向的内容**。

```c++
#include <iostream>
using namespace std;

// 基类
class Shape {
public:
    void setWidth(int w) {
        width = w;
    }

    void setHeight(int h) {
        height = h;
    }
	// 基类有一个计算面积函数
    int getArea() {
        cout << "调用的基类函数" << endl;
        return (width * height);
    }

protected:
    int width;
    int height;
};

// 派生类
class Rectangle : public Shape {
public:
    // 派生类也有一个同名的计算面积函数
    int getArea() {
        cout << "调用的派生类函数" << endl;
        return (Shape::width * Shape::height);
    }
};

int main() {
    // 多态：基类指针指向派生类对象
    Shape *shape = new Rectangle();
    // 基类指针调用函数
    shape->setWidth(5);
    shape->setHeight(7);

    // 输出对象的面积
    cout << shape->getArea() << endl;

    return 0;
}
```

【运行结果】

> 调用的基类函数
>
> 35

在编译期间，就已经确定了 `shape->getArea()` 调用的是基类的 `getArea()` 函数，即**静态多态**确定

## 6.2. 动态多态（虚函数）

**动态多态（动态链接）**是指函数调用时再根据指针指向来确定确定，即**只关心指针指向的内容，而不关心指针的类型**。

### 6.2.1. 虚函数

是在基类中使用关键字 `virtual` 声明的函数。在派生类中重新定义基类中的虚函数时，会告诉编译器不要静态链接到该函数。即使用动态链接。

```c++
// 基类有一个计算面积虚函数
virtual int getArea() {
    cout << "调用的基类函数" << endl;
    return (width * height);
}
```

【运行结果】

> 调用的派生类函数
>
> 35

【注意】

- 基类构造函数不要设置为虚函数：编译错误 `error: constructor cannot be declared 'virtual'`。
- 基类析构函数一般设置为虚函数：防止只析构基类而不析构派生类，导致派生类内存泄漏。

### 6.2.2. 纯虚函数

基类中对虚函数给出有意义的实现，以便在派生类中决定该函数的具体实现。纯虚函数是通过在声明中使用 `= 0` 来指定。

```c++
// 基类有一个计算面积纯虚函数
virtual int getArea() = 0;
```

### 6.2.3. 抽象类

抽象类就是至少有一个函数被声明为**纯虚函数**。

- 抽象类不能被用于实例化对象，它只能作为**接口**使用。如果试图实例化一个抽象类的对象，会导致编译错误。
- 派生类必须重写基类的纯虚函数，否则会导致编译错误。

