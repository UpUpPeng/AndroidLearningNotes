# 1. 数据类型

数据类型指的是用于声明不同类型的变量或函数的一个广泛的系统。

- **基本类型：** 算术类型，包括两种类型：整数类型和浮点类型。
- **bool 类型：** C 语言没有，C++ 中增加，只有两个取值，true 和 false。
- **枚举类型：** 算术类型，被用来定义在程序中只能赋予其一定的离散整数值的变量。
- **void 类型：** 类型说明符 *void* 表明没有可用的值。
- **派生类型：** 包括指针类型、数组类型、结构类型、共用体类型和函数类型。

## 1.1. 基本类型

**【整数类型】**

| 类型           | 存储大小    | 值范围                                             |
| :------------- | :---------- | :------------------------------------------------- |
| **char**       | 1 字节      | -128 ~ 127 或 0 ~ 255                              |
| unsigned char  | 1 字节      | 0 ~ 255                                            |
| signed char    | 1 字节      | -128 ~ 127                                         |
| **int**        | 2 或 4 字节 | -32,768 ~ 32,767 或 -2,147,483,648 ~ 2,147,483,647 |
| unsigned int   | 2 或 4 字节 | 0 ~ 65,535 或 0 ~ 4,294,967,295                    |
| **short**      | 2 字节      | -32,768 ~ 32,767                                   |
| unsigned short | 2 字节      | 0 ~ 65,535                                         |
| **long**       | 4 字节      | -2,147,483,648 ~ 2,147,483,647                     |
| unsigned long  | 4 字节      | 0 ~ 4,294,967,295                                  |

**【浮点类型】**

| 类型        | 存储大小 | 值范围                | 精度      |
| :---------- | :------- | :-------------------- | :-------- |
| float       | 4 字节   | 1.2E-38 ~ 3.4E+38     | 6 位小数  |
| double      | 8 字节   | 2.3E-308 ~ 1.7E+308   | 15 位小数 |
| long double | 16 字节  | 3.4E-4932 ~ 1.1E+4932 | 19 位小数 |

## 1.2. bool 类型

C 没有从语法上支持“真”和“假”，只是用 `0 false` 和 `非0 true` 来表示。

C++ 新增了 **bool 类型（布尔类型）**，占用 1 个字节，只有两个取值，true 和 false。

## 1.3. void 类型

void 类型指定没有可用的值。它通常用于以下三种情况下：

- **函数返回为空：** 函数可以不返回值，或者说返回空。如 `void exit (int status);`。
- **函数参数为空：** 函数可以不接受任何参数，或者说接受一个 `void` 类型的参数。例如 `int rand(void);`。
- **指针指向 void：** 类型为 `void *` 的指针代表对象的地址，而不是类型。例如，内存分配函数 `void *malloc(size_t size);` 返回指向 `void` 的指针，可以转换为任何数据类型。

---

<br/>

# 2. 数组

## 2.1. 声明数组

定长声明一个数组需要指定元素的类型和元素的数量。

```c++
// type可以是任意有效的数据类型，arraySize必须是一个大于零的整数常量
type arrayName [arraySize];
// 例如：
double balance[5];
```

这种方式声明的数组，占用**栈内存**空间，如果数组长度过大，可能会导致栈内存溢出。

## 2.2. 初始化数组

声明时初始化

```c++
double balance[] = {1000.0, 2.0, 3.4, 7.0, 50.0};
```

## 2.3. 动态数组

声明数组的大小之后，无法更改，但有时数组大小可能不够，就需要动态扩容，即动态内存分配。动态分配的内存占用**堆内存**空间，因此使用完毕后应主动释放。

### 2.3.1. malloc

内存分配（memory allocation）。 `malloc()` 函数保留指定字节数的内存块，返回一个 `void *` 可类型转换为任何形式的指针。

```c++
// type可以是任意有效的数据类型，size是一个需要保留的内存块大小
type *ptr = (type *) malloc(size);
// 例如：
int *ptr = (int *) malloc(100 * sizeof(int));
// 但是此时指针数组尚未初始化，需要手动初始化
// 将ptr中当前位置后面的n个字节用0替换并返回ptr。
memset(ptr, 0, 100 * sizeof(int));
```

### 2.3.2. calloc

连续分配（contiguous allocation）。`malloc()` 函数分配内存，但不初始化内存。而 `calloc()` 函数分配内存并将所有位初始化为0。

```c++
// type可以是任意有效的数据类型，分配count个，每个size字节的连续内存块
type *ptr = (type *) calloc(count, size);
// 例如：
int *ptr = (int *) calloc(100, sizeof(int));
```

### 2.3.3. realloc

重新分配（reset allocation）。重新分配内存大小，有两种情况：

- 如果当前指针有足够的连续空间，则扩大原指针指向的地址，并且将原指针返回。
- 如果当前指针没有足够的连续空间，则先分配 newsize 大小的新空间，再将原数据从头到尾拷贝到新区域，最后释放原指针所指内存区域（注意：旧内存会被自动释放，旧指针变成了野指针），同时返回新指针。

```c++
// type可以是任意有效的数据类型，分配count个，每个size字节的连续内存块
type *ptr = (type *) realloc(ptr, newsize);
// 例如：
ptr = (int *) realloc(ptr, 200 * sizeof(int));
```

### 2.3.4. free

使用 `calloc()` 或 `malloc()` 创建的动态分配内存，必须明确使用 `free()` 释放空间。

```c++
free(ptr);
```

---

<br/>

# 3. 指针

**指针**是一个变量，其值为另一个变量的地址，即，内存位置的直接地址。

```c++
int i = 10;
// p的值是i的内存地址
int *p = &i;

std::cout << "i：" << i << std::endl;	// 【10】
std::cout << "&i：" << &i << std::endl;	// 【0x7ffee46bba38】
std::cout << "p：" << p << std::endl;	// 【0x7ffee46bba38】
std::cout << "&p：" << &p << std::endl;	// 【0x7ffee46bba30】
// 解引用，解析p指向地址所存储的值
std::cout << "*p：" << *p << std::endl;	// 【10】
```

## 3.1. 指针与数组

### 3.1.1. 数组指针

即数组的指针，数组名本身就是数组的首地址。因此 `int *p1 = array1;` 和 `int (*p2)[3] = array2;` 都是指向数组的指针，即数组指针。

```c++
// 声明并初始化一维数组
int array1[] = {11, 22, 33, 44, 55, 66};
// 声明指针p1，并指向数组的首地址
int *p1 = array1;
// 打印指针p1指向的地址值
std::cout << "p1：" << p1 << std::endl;					// 【0x7ffeed1d5a20】
// 打印指针p1指向的地址所存储的值，即数组第0元素
std::cout << "*p1：" << *p1 << std::endl;				// 【11】
// 打印指针p1指向的地址的下1位地址所存储的值，即数组第1元素
std::cout << "*(p1 + 1)：" << *(p1 + 1) << std::endl;	// 【22】
// 打印指针p1指向的地址的下6位地址所存储的值，即数组第6元素，越界，值不可确定
std::cout << "*(p1 + 6)：" << *(p1 + 6) << std::endl;	// 【1485046000】

std::cout << "------------------------" << std::endl;

// 声明并初始化二维数组
int array2[2][3] = {{11, 22, 33}, {44, 55, 66}};
// 声明指针p2，并指向二维数组的首地址
int (*p2)[3] = array2;
// 打印指针p2指向的地址值
std::cout << "p2：" << p2 << std::endl;					// 【0x7ffeed1d5a00】
// 打印指针p2指向的地址所存储的值，即二维数组第0行的首地址值
std::cout << "*p2：" << *p2 << std::endl;				// 【0x7ffeed1d5a00】
// 打印指针p2指向的地址所存储的值，即二维数组第0行第0元素
std::cout << "**p2：" << **p2 << std::endl;				// 【11】
// 打印指针p2指向的地址的下1位地址所存储的值，即二维数组第1行的首地址值
std::cout << "*(p2 + 1)：" << *(p2 + 1) << std::endl;	// 【0x7ffeed1d5a0c】
// 打印指针p2指向的地址的下1位地址所存储的值，即二维数组第1行第0元素
std::cout << "**(p2 + 1)：" << **(p2 + 1) << std::endl;	// 【44】
// 打印指针p2指向的地址的下1位地址所存储的值的下1位地址所存储的值，即二维数组第1行第1元素
std::cout << "*(*(p2 + 1) + 1)：" << *(*(p2 + 1) + 1) << std::endl;	// 【55】
```

### 3.1.2. 指针数组

即指针类型的数组，数组中的每个元素都是指针类型。

```c++
int i = 10;
int j = 20;
int k = 30;
// 声明并初始化一个指针数组
int *p_array[] = {&i, &j, &k};
std::cout << "p_array：" << p_array << std::endl;				// 【0x7ffeea87ba20】
std::cout << "*p_array：" << *p_array << std::endl;				// 【0x7ffeea87ba18】
std::cout << "**p_array：" << **p_array << std::endl;			// 【10】
std::cout << "*(p_array + 1)：" << *(p_array + 1) << std::endl;	// 【0x7ffeea87ba14】
std::cout << "**(p_array + 1)：" << **(p_array + 1) << std::endl;// 【20】 
```

## 3.2. 指针与常量

### 3.2.1. 常量指针

数据类型前用 `const` 修饰，被定义的指针变量就是**指向常量的指针**，即常量指针。

- 声明格式为 `const type * p` 或 `type const * p` 。
- 常量指针所指向内存区域的值不能被改变。

```c++
// 定义一个常量指针
const char *p = "OK";
// ❎ 编译报错：修改指针所指向的内存区域的值 
p[0] = 'A';
// ✅ 编译通过：修改指针的指向
p = "YES";
// ✅ 编译通过：修改指针的指向
p = NULL;
```

### 3.2.2. 指针常量

指针变量前用 `const` 修饰，被定义的指针变量就是**指针类型的常量**，即指针常量。

- 声明格式为 `type * const p` 。
- 常量指针的指向不能被改变。

```c++
// 定义一个指针常量
const * char p = "OK";
// ✅ 编译通过：修改指针所指向的内存区域的值 
p[0] = 'A';
// ❎ 编译报错：修改指针的指向
p = "YES";
// ❎ 编译报错：修改指针的指向
p = NULL;
```

### 3.2.3. 常指针常量

数据类型前用 `const` 修饰，指针变量前也用 `const` 修饰，被定义的指针变量就是**常量指针类型的常量**，即常指针常量。

- 声明格式为 `const type * const p` 或 `type const * const p` 。
- 常指针常量的指向不能被改变，且所指向内存区域的值也不能被改变。

```c++
// 定义一个指针常量
const * char p = "OK";
// ❎ 编译报错：修改指针所指向的内存区域的值 
p[0] = 'A';
// ❎ 编译报错：修改指针的指向
p = "YES";
// ❎ 编译报错：修改指针的指向
p = NULL;
```

## 3.3. 多级指针

当指针变量所指的变量也是一种指针时，则该指针变量是一种**指向指针的指针**，即多级指针。

```c++
int i = 10;
int *p1 = &i;    // 一级指针
int **p2 = &p1;   // 二级指针
int ***p3 = &p2;  // 三级指针
std::cout << "i：" << i << std::endl;		// 【10】
std::cout << "*p1：" << *p1 << std::endl;	// 【10】
std::cout << "**p2：" << **p2 << std::endl;	// 【10】
std::cout << "***p3：" << ***p3 << std::endl;// 【10】
```

---

<br/>

# 4. 函数

函数是一组一起执行一个任务的语句。

## 4.1. 函数声明

函数**声明**会告诉编译器函数名称及如何调用函数。函数的实际主体可以单独定义。

- 声明格式为 `返回值类型 函数名(参数列表);`
- 函数声明中，参数名并不重要，因此可以省略参数名，只保留参数类型。
- 在一个源文件中定义函数且在另一个文件中调用函数时，函数声明是必需的。

## 4.2. 函数定义

函数函数定义的一般形式如下：

```c++
返回值类型 函数名(参数列表) 
{   
	函数体
}
```

函数由一个函数头和一个函数主体组成：

- **返回值类型：** 函数返回的值的数据类型。有些函数不返回值，则返回值类型为 `void`。
- **函数名称：** 函数的实际名称。函数名和参数列表一起构成了函数签名。
- **参数列表：** 当函数被调用时，向函数传递实际参数过来，参数列表中的形式参数接收。参数列表包括类型、顺序、数量，也可以为空。
- **函数主体：** 函数主体包含一组定义函数执行任务的语句。

## 4.3. 传递参数的方式

### 4.3.1. 值传递

把参数的**实际值**赋值给函数的形式参数，因此，修改形式参数对实际参数没有影响。

```c++
int fun(int x) { return x += 10; }

int main() {
    int num = 10;
    // 值传递
    fun(num);
    // 不会改变本身的实际参数的值
    std::cout << "num：" << num << std::endl;	// 【10】

    return 0;
}
```

- 值传递有一个形式参数向函数所属的栈拷贝数据的过程，如果值传递的对象是类对象或是大的结构体对象，将耗费一定的时间和空间。

### 4.3.2. 指针传递

把参数的**地址**赋值给函数的形式参数。在函数内，可以通过该地址访问实际参数。因此，修改形式参数对实际参数有影响。

```c++
int fun(int *x) { return *x += 10; }

int main() {
    int num = 10;
    // 指针传递，把实际参数的地址传给形式参数中的指针
    fun(&num);
    std::cout << "num：" << num << std::endl;	// 【20】

    return 0;
}
```

- 指针传递有一个形式参数向函数所属的栈拷贝数据的过程，但拷贝的数据是一个固定为 4 字节的地址。

### 4.3.3. 引用传递

把参数的**引用**赋值给函数的形式参数。在函数内，可以通过该引用访问实际参数。因此，修改形式参数对实际参数有影响。

```c++
int fun(int &x) { return x += 10; }

int main() {
    int num = 10;
    fun(num);
    std::cout << "num：" << num << std::endl;	// 【20】

    return 0;
}
```

- 指针传递有一个形式参数向函数所属的栈拷贝数据的过程，相当于为该数据所在的地址起了一个别名。

### 4.3.4. 方式总结

- 效率上，指针传递和引用传递效率更高。
- 引用传递是 C++ 的特性，C 语言不支持。

## 4.4. 可变参数

### 4.4.1. 可变参数宏

使用到四个宏：`va_list`、`va_start(va_list, arg)`、`va_arg(va_list, type)`、`va_end(va_list)` 。

```c++
/**
 * @param count 参数个数
 */
int sum(int count, ...) {
    // 声明一个va_list变量，arg代表...
    va_list arg;
    // 开始使用可变参数
    va_start(arg, count);
    int sum = 0;
    for (int i = 0; i < count; ++i) {
        // 获取可变参数
        sum += va_arg(arg, int);
    }
    // 清理可变参数
    va_end(arg);
    return sum;
}

int main() {
    int sum_num = sum(5, 1, 1, 1, 1, 1);
    std::cout << "sum：" << sum_num << std::endl;

    return 0;
}
```

**缺点：** 不安全！count 如果和实际参数个数不一致，就会发生严重错误。

### 4.4.2. initializer_list 标准库类型

- C++ 11 新增，需引入 `initializer_list` 头文件，且元素必须具有相同类型。
- 函数原型中使用实例化 `initializer_list` 模板代表可变参数列表。
- 使用迭代器，遍历 `initializer_list` 中的参数。
- 传入实参，必须放在一组大括号中。

```c++
#include <iostream>
#include <initializer_list>

int sum(std::initializer_list<int> args) {
    int sum = 0;
    // 有点像迭代器
    for (auto arg = args.begin(); arg != args.end(); arg++) {
        sum += *arg;
    }
    return sum;
}

int main() {
    // 实参必须全部放到{}中
    int sum_num = sum({1, 1, 1, 1, 1});
    std::cout << "sum：" << sum_num << std::endl;

    return 0;
}
```

## 4.5. 函数与指针

### 4.5.1. 指针函数

返回值是一个指针的函数。

声明格式：`返回值类型 *函数名(参数列表) `，`*` 与前者结合，使得返回值是一个指针类型。

```c++
int *sum(int num_1, int num_2) {
    // 基本数据类型，默认都在栈上分配内存
    int sum = num_1 + num_2;
    return &sum;
}

int main() {
    int *sum_p = sum(10, 20);
    std::cout << "*sum_p：" << *sum_p << std::endl;	// 【1】
    return 0;
}
```

⭐️ 注意：`int sum = num_1 + num_2;` 局部变量会在栈上分配，当 `int *sum() ` 函数运行结束，`sum` 就会被销毁，但它的内存地址却作为返回值保留了。因此，`main` 函数再获取原地址所对应的值时，得到的值就不再是正确结果了。

不同的编译平台，对销毁局部变量的处理不同：

- Xcode：不会立即销毁，极短时间内根据地址获得的值是正确的，但延时一段时间或者第二次获取值时，就不再正确了。
- Clion：会立即销毁，根据地址获得的值始终不正确。

为了能让外部函数能够通过地址获取到正确值，那么局部变量就不能栈上分配，可以放到堆中。

```c++
int *sum(int num_1, int num_2) {
    // 在堆上分配内存
    int *sum = static_cast<int *>(malloc(sizeof(int)));
    // 对该内存空间赋值
    sum[0] = num_1 + num_2;
    return sum;
}

int main() {
    int *sum_p = sum(10, 20);
    std::cout << "*sum_p：" << *sum_p << std::endl;	// 【30】
    free(sum_p);
    return 0;
}
```

### 4.5.2. 函数指针

指向函数的指针。

声明格式：`返回值类型 (*函数名)(参数列表) `，`*` 强制与函数名结合，使得函数名是一个指针。

```c++
// 普通函数
int add(int x, int y) { return x + y; }
// 普通函数
int sub(int x, int y) { return x - y; }

int main() {
    // 声明一个函数指针，返回值为int，接受两个int型参数
    int (*fun)(int, int);
	// 将函数指针指向add函数，或者fun = &add;
    fun = add;
   	// 两种调用函数的方法
    std::cout << "(*fun)(1,2)=" << (*fun)(1, 2) << std::endl;	// 【3】
    std::cout << "fun(1,2)=" << fun(1, 2) << std::endl;			// 【3】
    // 将函数指针指向sub函数，或者fun = &sub;
    fun = sub;
    // 两种调用函数的方法
    std::cout << "(*fun)(5,3)=" << (*fun)(5, 3) << std::endl;	// 【2】
    std::cout << "fun(5,3)=" << fun(5, 3) << std::endl;			// 【2】

    return 0;
}
```

### 4.5.3. 回调函数

把函数指针作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，这就是回调函数。

```c++
// 定义两种函数指针类型
typedef void(*Success)(char *);
typedef void(*Failure)(char *);

// 执行函数
void http(char *url, Success success, Failure failure) {
    // 通过函数指针指向真实函数
    if (true){
        success("请求成功");
    } else{
        failure("请求失败");
    }
}
// 回调函数
void success(char *msg) {
    printf(msg);
}
// 回调函数
void failure(char *msg) {
    printf(msg);
}

int main() {
    // 执行函数，并把回调函数传递进去
    http("https://www.qq.com", success, failure);
    return 0;
}
```

---

<br/>

# 5. 预处理器

预处理器不是编译器的组成部分，但是它是编译过程中一个单独的步骤。只不过是一个文本替换工具而已，它们会指示编译器在实际编译之前完成所需的预处理。

所有的预处理器命令都是以井号（#）开头。

| 指令     | 描述                                                        |
| :------- | :---------------------------------------------------------- |
| #define  | 定义宏                                                      |
| #include | 包含一个源代码文件                                          |
| #undef   | 取消已定义的宏                                              |
| #ifdef   | 如果宏已经定义，则返回真                                    |
| #ifndef  | 如果宏没有定义，则返回真                                    |
| #if      | 如果给定条件为真，则编译下面代码                            |
| #else    | #if 的替代方案                                              |
| #elif    | 如果前面的 #if 给定条件不为真，当前条件为真，则编译下面代码 |
| #endif   | 结束一个 #if……#else 条件编译块                              |
| #error   | 当遇到标准错误时，输出错误消息                              |
| #pragma  | 使用标准化方法，向编译器发布特殊的命令到编译器中            |

## 5.1. 预处理器实例

### 5.1.1. #define

用于创建符号常量，该符号常量通常称为**宏**。

```c++
#include <iostream>
using namespace std;

// 告诉编译器，把所有的PI替换为3.14159。
#define PI 3.14159
// 取消已经定义过的PI宏
#undef  PI
// 告诉编译器，把所有的PI替换为3.141592653。
#define PI 3.141592653

int main () {
    cout << PI << endl; 
    // 编译后的代码：
    // cout << 3.141592653 << endl; 
    return 0;
}
```

也可以定义一个带有参数的宏。

```c++
#include <iostream>
using namespace std;

// 告诉编译器，把所有的MIN(a,b)替换为(a<b ? a : b)。
#define MIN(a,b) (a<b ? a : b)

int main () {
    int i = 10;
    int j = 20;
    cout << MIN(i, j) << endl;
    // 编译后的代码：
    // cout << i<j ? i : j << endl;
    return 0;
}
```

注意，宏只做文本替换，不做任何校验。

```c++
#include <iostream>
using namespace std;

#define FUN(a, b) a*b

int main() {
    // 期望结果：(1+2)*(3+4) = 21
   	// 实际结果：1 + 2*3 + 4 = 11
    cout << FUN(1 + 2, 3 + 4) << endl;	// 【11】
    return 0;
}
```

- 优点：编译前就做替换，宏函数没有普通函数的调用开销。
- 缺点：没有编译检查，写错了也就错了。

### 5.1.2. 条件编译

有选择地对部分程序源代码进行编译，称为条件编译。

如果满足条件，就编译：

```c++
#if 0
	// 不参与编译，相当于注释
#endif	
```

如果宏没有定义，则新定义一个：

```c++
#ifndef MESSAGE
    #define MESSAGE "Hello World!"
#endif
```

如果宏已经定义，则取消这个宏：

```c++
#ifdef MESSAGE
	#undef MESSAGE
#endif
```

## 5.2. 预定义宏

| 宏       | 描述                                                |
| :------- | :-------------------------------------------------- |
| __DATE__ | 当前日期，一个以 "MMM DD YYYY" 格式表示的字符常量。 |
| __TIME__ | 当前时间，一个以 "HH:MM:SS" 格式表示的字符常量。    |
| __FILE__ | 这会包含当前文件名，一个字符串常量。                |
| __LINE__ | 这会包含当前行号，一个十进制常量。                  |
| __STDC__ | 当编译器以 ANSI 标准编译时，则定义为 1。            |

```c++
#include <iostream>
using namespace std;

int main() {
    cout << __LINE__ << endl;	// 【5】
    cout << __FILE__ << endl;	// 【/Users/admin/workspace/CppTest/main.cpp】
    cout << __DATE__ << endl;	// 【Mar  4 2021】
    cout << __TIME__ << endl;	// 【14:50:02】
    cout << __STDC__ << endl;	// 【1】

    return 0;
}
```

## 5.3. # 和 ## 运算符

① `#` 运算符会把令牌转换为用引号引起来的字符串。

```c++
#include <iostream>
using namespace std;

// ① 告诉编译器，把所有的MKSTR(x)替换为#x。
// ② 把传入的x转换为x的字符串
#define MKSTR(x) #x

int main() {
    cout << MKSTR(HELLO C++);	// 【HELLO C++】
    // cout << "HELLO C++";
    return 0;
}
```

② `##` 运算符用于连接两个令牌。

```c++
#include <iostream>
using namespace std;

// ① 告诉编译器，把所有的concat(a, b)替换为a ## b。
// ② 把传入的a和b转换为ab
#define concat(a, b) a##b

int main() {
    int xy = 100;
    cout << concat(x, y);	// 【100】
    // cout << xy;
    return 0;
}
```

---



# 6. 结构体与共用体

## 6.1. 结构体

结构体是用户自定义的可用的数据类型。

### 6.1.1. 结构体使用

声明格式：

```c++
struct 结构体名称 {
	成员类型1 成员名称1;
    成员类型2 成员名称2;
    成员类型3 成员名称3;
};
结构体名称 变量名称;
```

使用成员访问运算符 `.` 来访问结构体成员。

```c++
#include <iostream>

using namespace std;
// 定义一个结构体类型User 
struct User {
    int age;	// 成员1
    char *name;	// 成员2
};

int main() {

    User user1;
    User user2;
    // 给成员赋值
    user1.age = 20;
    user1.name = "张三";
    user2.age = 30;
    user2.name = "李四";
	// 获取成员的值
    cout << user1.name << " " << user1.age << endl;
    cout << user2.name << " " << user2.age << endl;
    return 0;
}
```

### 6.1.2. 字节对齐

在 64 位系统中，`int` 占用4个字节，`char *` 占用8个字节，因此 `User` 理论大小为12个字节。

```c++
struct User {
    int age;
    char *name;
};
cout << sizeof(User) << endl;	// 【16】
```

实际上 `User` 的大小为16个字节。

**【字节对齐的步骤】**

- 全部成员变量先按照自己的大小对齐，即**当前的偏移地址必须为类型大小的最小整数倍**：
  - 假设 `int age ` 的起始内存地址为 `0x0000` ，则结束内存地址为 `0x0003` ，共占用4个字节。
  - `char *name` 的理论起始地址为 `0x0004`，但是 `char *` 类型占8个字节，因此需要对 `0x0000` 做8个字节的最小整数倍偏移而得到起始地址，即 `0x0008` ，则结束内存地址为 `0x0011`。
- 结构体再按照最大对齐参数对齐，即**对齐到最大成员的整数倍**：
  - `int` 和 `char *` 对比，最大的是 `char *` 占用8个字节。
  - 从 `0x0000` 到 `0x0011` 总共占用16字节，恰好是8的整数倍，因此结构体不用再做对齐。

**【字节对齐的意义】**

- 根本原因是提高 CPU 访问数据的效率。减少一个数据横跨两个时钟周期才能读取的概率。
- 定义结构体时，成员变量定义的顺序应该按照从大到小的先后顺序，则有利于节约内存空间。

**【指定对齐大小】**

```c++
# pragma pack(2)	// 指定以2个字节来对齐，必须是2的整数倍
struct User {
    int age;
    char *name;
};
# pragma pack()		// 还原为系统默认的对齐大小
cout << sizeof(User) << endl;	// 【12】
```

## 6.2. 共用体

共用体是一种特殊的数据类型，允许在**相同的内存位置存储不同的数据类型**。

定义一个带有多成员的共用体，但是任何时候只能有一个成员带有值。共用体提供了一种使用相同的内存位置的有效方式。

声明格式：

```c++
union 共用体名称 {
	成员类型1 成员名称1;
    成员类型2 成员名称2;
    成员类型3 成员名称3;
};
共用体名称 变量名称;
```

使用成员访问运算符 `.` 来访问共用体成员。

```c++
#include <iostream>
using namespace std;

// 定义一个共用体类型U
union U {
    int i;	// 成员1
    int j;	// 成员2
};

int main() {

    U u;
    u.i = 10;	// 对成员1赋值
    u.j = 20;	// 对成员2赋值

    cout << u.i << " " << u.j << endl;	// 【20 20】任何时候，只有一个成员有正确值
    cout << sizeof(U) << endl;			// 【4】占用的内存 = 共用体中最大成员占用的内存
    return 0;
}
```

---

<br/>

# 7. 字符串

## 7.1. C 风格字符串

C 语言字符串实际上是使用 `null` 字符 `\0` 终止的一维字符数组。因此，一个以 null 结尾的字符串，包含了组成字符串的字符。

```c++
char str1[6] = {'H', 'e', 'l', 'l', 'o', '\0'};	// 以\0结尾的字符数组
char str2[] = "Hello";							// 数组初始化规则，自动补充\0
char *str3 = "Hello";							// 指向数组其实地址的指针，自动补充\0
```

字符数组操作函数：

| 函数            | 结果                                                         |
| :-------------- | :----------------------------------------------------------- |
| strcpy(s1, s2); | 复制字符串 s2 到字符串 s1。                                  |
| strcat(s1, s2); | 连接字符串 s2 到字符串 s1 的末尾。连接字符串也可以用 `+` 号。 |
| strlen(s1);     | 返回字符串 s1 的长度。                                       |
| strcmp(s1, s2); | 如果 s1 和 s2 是相同的，则返回 0；如果 s1<s2 则返回值小于 0；如果 s1>s2 则返回值大于 0。 |
| strchr(s1, c);  | 返回一个指针，指向字符串 s1 中字符 c 的第一次出现的位置。    |
| strstr(s1, s2); | 返回一个指针，指向字符串 s1 中字符串 s2 的第一次出现的位置。 |

## 7.2. C++ 风格字符串

C++ 标准库提供了 `string` 类类型，支持上述所有的操作，另外还增加了其他更多的功能。

```c++
string str1 = "Hello";      		// 使用字面量创建
string str2(str1);          		// 使用构造方法创建
string str3("Hello");   			// 使用构造方法创建
string *str4 = new string("Hello"); // 在堆中申请内存创建

string str5 = str1;				// 字符串赋值
string str6 = str1 + str2;		// 字符串拼接

cout<< str3 << endl;			// 输出一般类型的字符串
cout<< str4->c_str() << endl;	// 输出指针类型的字符串

delete str4;	// 用完了要释放堆内存
```

