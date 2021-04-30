# 模板编程

模板是泛型编程的基础，泛型编程即以一种独立于任何特定类型的方式编写代码。

## 函数模板

函数模板定义的一般形式：

```c++
template <typename 泛型占位符> 
返回值类型 函数名(参数列表) {
	// 函数体 
}
```

示例代码：

```c++
#include <iostream>
#include <string>
using namespace std;

template<typename T>
T &Max(T &a, T &b) {
    return a < b ? b : a;
}

int main() {

    int i = 10;
    int j = 20;
    cout << Max(i, j) << endl;		// 10 

    char c1 = 'a';
    char c2 = 'z';
    cout << Max(c1, c2) << endl;	// z

    string s1 = "ABC";
    string s2 = "XYZ";
    cout << Max(s1, s2) << endl;	// XYZ

    return 0;
}
```

## 类模板

类模板定义的一般形式：

```c++
template <class 泛型占位符> 
class 类名 {
	// 类体 
}
```

示例代码：

```c++
#include <iostream>
#include <string>
using namespace std;

template<class T>
class Adder {
public:
   	// 两个元素相加
    T add(T elem1, T elem2) {
        return elem1 + elem2;
    }
};

int main() {
    // 在栈中创建一个Adder对象
    Adder<int> adder1;
    cout << adder1.add(10, 20) << endl;	// 30

    // 在堆中创建一个Adder对象
    auto *adder2 = new Adder<float>;
    cout << adder2->add(11.1, 22.2) << endl;	// 33.3
    // 堆中的对象，使用完毕要手动释放
    delete adder2;

    Adder<string> adder3;
    cout << adder3.add("Hello", "World") << endl;	// HelloWorld

    return 0;
}
```

---

<br/>

# 容器

容器是用来管理某一类对象的集合。C++ 提供了各种不同类型的容器，比如 `vector`、`list`、`deque`、`stack`、`map` 等。

## vector

vector（向量）是一个封装了动态大小数组的顺序容器。即能够存放任意类型的动态数组。

### 特性

- 顺序序列。可以通过下标访问对应的元素。
- 动态数组。支持随机访问，支持指针访问。
- 能够感知内存分配器的容器。使用一个内存分配器对象来动态地处理它的存储需求。

### 基本操作

**① 创建容器：**

```c++
vector<int> vec1;			// 创建默认的vector对象
vector<int> vec2(10);		// 创建初始容量为10的vector对象
vector<int> vec3(10, 100);	// 创建初始容量为10，且初始值为100的vector对象
vector<int> vec4(vec2);		// 创建以vec2为拷贝的vector对象
vector<int> vec5(vec2.begin(), vec2.end());	// 同上
```

**② 添加元素：**

```c++
// 原始容器元素 []
// 在尾部增加一个元素1
vec.push_back(1);					// [1]
// 在1位置插入1个元素2，后面的元素后移
vec.insert(vec.begin() + 1, 2);		// [1, 2]
// 在2位置插入2个元素5，后面的元素后移
vec.insert(vec.begin() + 2, 2, 5);	// [1, 2, 5, 5]
```

**③ 访问元素：**

```c++
// 原始容器元素 [1, 2, 5, 5]
// 返回0位置元素的引用
int &int_0 = vec.at(0);		// 1
// 下标访问1位置元素
int int_1 = vec[1];			// 2
// 返回第一个元素的引用
int &front = vec.front();	// 1
// 返回最后一个元素的引用
int &back = vec.back();		// 5
```

**④ 修改元素：**

```c++
// 原始容器元素 [1, 2, 5, 5]
// 访问元素后修改元素值
vec.at(0) = 100;	// [100, 2, 5, 5]
vec[1] = 200;		// [100, 200, 5, 5]
// 交换两个同类型向量的数据
vec.swap(vec1);		// vec:[]，vec1:[100, 200, 5, 5]
vec.assign(5,6);	// [6, 6, 6, 6, 6]
```

**⑤ 删除元素：**

```c++
// 原始容器元素 [1, 2, 5, 5]
// 删除1位置的元素2，后面的元素前移
vec.erase(vec.cbegin() + 1);	// [1, 5, 5]
// 删除最后一个元素
vec.pop_back();					// [1, 5]
// 清空所有元素
vec.clear();					// []
```

### 注意

添加元素会使容器的容量和长度增大，删除元素只会使长度减小，并不会减小容量。

```c++
// 全局变量
vector<int> vec;

int main() {
    vec.push_back(1);
    vec.pop_back();
	// vec.size()为0，vec.capacity()为1。
    return 0;
}
```

如果要使容量也缩减为 0，需要用另外一个容器和他交换。

```c++
// 全局变量
vector<int> vec;

// 对全局变量缩容
void recapacity() {
    // 创建一个空的容器
    vector<int> tmp;
    // 交换两个容器的内容
    vec.swap(tmp);
    // 函数结束时，tmp带着原vec的内容一起被回收
}

int main() {
    vec.push_back(1);
    vec.pop_back();
    recapacity(vec);
	// vec.size()为0，vec.capacity()为0。
    return 0;
}
```

## priority_queue

优先队列具有队列的所有特性，然后添加了内部的一个排序，逻辑上是一个堆实现的。

### 实现方式

优先队列可以用 `vector`（向量）或 `deque`（双向队列）来实现。

### 基本操作

**① 创建容器：**

```c++
// 创建默认优先队列，元素为int类型，默认递减，默认vector实现
priority_queue<int> pq;
// 创建递减优先队列，元素为int类型，使用vector实现
priority_queue<int, vector<int>, less<>> pq1;
// 使用递增优先队列，元素为int类型，使用deque实现
priority_queue<int, deque<int>, greater<>> pq2;
```

**② 添加元素：**

```c++
// 添加一个元素，并按顺序排列
pq.push(1);	// [1]
pq.push(3);	// [3, 1]
pq.push(2);	// [3, 2, 1]
```

**③ 访问元素：**

```c++
// 访问堆顶元素，不会删除
pq.top();	// 3，队列中剩余元素：[3, 2, 1]
```

**④ 删除元素：**

```c++
// 弹出堆顶元素，剩余元素位置刷新
pq.pop();	// 3，队列中剩余元素：[2, 1]
pq.pop();	// 2，队列中剩余元素：[1]
pq.pop();	// 1，队列中剩余元素：[]
```

### 自定义类型排序

priority_queue 可以存储自定义类型，并自行实现排序规则。

```c++
#include <iostream>
#include <queue>
using namespace std;

// 自定义Test类型
class Test {
public:
    int value;

    Test(int value) : value(value) {}
};

// 自定义Test类型的递减排序规则
struct TestLess {
    // 重载操作符，实现递减比较
    constexpr bool operator()(const Test &left, const Test &right) const {
        return left.value < right.value;
    }
};

int main() {
    // 自定义类型的优先队列
    priority_queue<Test, vector<Test>, TestLess> pq;
	// 添加元素
    pq.push(Test(1));
    pq.push(Test(3));
    pq.push(Test(2));

    cout << pq.top().value << endl;	// 3

    return 0;
}
```

## set

 set 是一个关联式容器。

### 特性

- 元素唯一：每个元素的值都唯一，而且系统能根据元素的值自动进行排序。
- 不可变：元素的值不能直接被改变。
- 高效：内部采用红黑树实现。

### 基本操作

**① 创建容器：**

```c++
// 创建默认set容器，元素为int类型，默认递增
set<int> s;
// 创建默认set容器，元素为int类型，初始值为[1,2,3]
set<int> s1 = {1, 2, 3};
// 创建递减set容器，元素为int类型
set<int, less<>> s2;
// 创建递增set容器，元素为int类型
set<int, greater<>> s3;
```

**② 添加元素：**

```c++
// 添加一个元素，并按顺序排列
s.insert(10);	// [10]
s.insert(30);	// [10, 30]
s.insert(20);	// [10, 20, 30]
```

**③ 访问元素：**

```c++
// 使用迭代器访问
set<int>::iterator iterator = s.begin();
// C++11可以使用自动类型推断
// auto iterator = s.begin();
while (iterator != s.end()) {
    cout << *iterator << endl;	// 10,20,30
    iterator++;
}
// 查找元素是否存在
set<int>::iterator f1 = s.find(20);
if (f1 != s.end()) {
    // 如果存在，则迭代器指向在集合中搜索的元素
    cout << "存在" << endl;
} else {
    // 如果不存在，则迭代器指向集合中最后一个元素之后的位置。
    cout << "不存在" << endl;
}
```

**④ 删除元素：**

```c++
// 删除某个元素
s.erase(20);	// [10, 30]
// 删除全部元素
s.clear();		// []
```

