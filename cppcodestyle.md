# ABACUS 开源项目 C++ 代码规范

## 0 前言

1. 使代码易于管理的方法之一是加强代码一致性。
2. 让任何开发者都可以快速读懂你的代码这点非常重要。

- 禁止用 C++11 之后版本的语法
- 每个代码文件不超过 500 行
- 每个函数不超过 50 行

## 1 关于头文件

### 1. 尽可能少的引入依赖

不需要的 include 记得删除

### 2. #define 保护

```cpp
#ifndef FOO_BAR_BAZ_H_
#define FOO_BAR_BAZ_H_
...
#endif // FOO_BAR_BAZ_H_
```

### 3. 避免使用前置声明

什么是前置声明？

```C++
// MyClassA.h

// Bad: Overuse of forward declarations
class MyClassB;

class MyClassA {
public:
    void DoSomething(MyClassB* obj_b);
};

void MyClassA::DoSomething(MyClassB* obj_b) {
    // ...
}
```

在这段代码中，`MyClassB` 被声明为一个类，但没有给出其定义。这被称为前置声明（Forward Declaration），它告诉编译器 `MyClassB` 是一个存在的类，但不提供该类的详细信息。

多使用 `#include` 包含需要的头文件。避免使用前置声明。

### 4. 内联函数

- 只有当函数只有 10 行甚至更少时才将其定义为内联函数.

### 5. #include 的路径及顺序（clang-format 可以自动）

- 使用标准的头文件包含顺序可增强可读性, 避免隐藏依赖
- 项目内头文件应按照项目源代码目录树结构排列, 避免使用 UNIX 特殊的快捷目录 `.` (当前目录) 或 `..` (上级目录).

`foo.cpp` 中包含头文件的次序如下:

> 1. `dir2/foo2.h` (优先位置, 详情如下)
> 2. C 系统文件
> 3. C++ 系统文件
> 4. 其他库的 `.h` 文件
> 5. 本项目内 `.h` 文件

这种优先的顺序排序保证当 `dir2/foo2.h` 遗漏某些必要的库时， `dir/foo.cc` 或 `dir/foo_test.cc` 的构建会立刻中止。

## 2 作用域

### 1. 命名空间

不应该使用 <em>using 指示</em> 引入整个命名空间的标识符号。

```cpp
// 禁止 —— 污染命名空间
using namespace foo;
```
应用举例：
例如在某些编译器环境下，在不使用`using space std`;的情况下，`std::abs()` 和 `abs()`的行为有可能不同。
`std::abs()`包括`std::abs(int)`, `std::abs(float)`, `std::abs(long long)`等，而`abs()`有可能只有`abs(int)`一种形式，导致`abs(float)`返回的值永远为`0`。
因此应尽量避免使用`using namespace std`+`abs()`的形式，建议直接使用使用`std::abs()`.

### 2. 非成员函数、静态成员函数和全局函数

- 不要用裸的全局函数，可以使用 静态成员函数 或者 命名空间内的非成员函数。
- 不要使用类的静态成员函数模拟命名空间的效果

```cpp
// 使用命名空间
// <strong>非成员函数</strong>
namespace my_math {
    int add(int a, int b) {
        return a + b;
    }
    
    int sub(int a, int b) {
        return a - b;
    }
}

int main() {
    int x = 3, y = 5;
    int z = my_math::add(x, y); // 调用 my_math 命名空间中的函数
    return 0;
}
```

```cpp
// 不要使用类的静态成员函数模拟命名空间
class MyMath {
public:
    static int add(int a, int b) {
        return a + b;
    }
    
    static int sub(int a, int b) {
        return a - b;
    }
};

int main() {
    int x = 3, y = 5;
    int z = MyMath::add(x, y); // 调用 MyMath 静态方法
    return 0;
}
```

### 3. 局部变量

1. 将函数变量尽可能置于最小作用域内
2. 在变量声明时进行初始化.

```cpp
int i;
i = f(); // 坏——初始化和声明分离

int j = f(); // 好——初始化时声明
```

```cpp
vector<int> v;  // 坏——初始化和声明分离
v.push_back(1);
v.push_back(2);

vector<int> v = {1, 2}; // 好——初始化时声明
```

> 在 `if`, `while` 和 `for` 语句中：<br/>1. 变量不是对象，则变量应当在内部声明与初始化，这样子这些变量的作用域就被限制在这些语句中了，举例而言：<br/>1. 如果变量是一个对象, 每次进入作用域都要调用其构造函数, 每次退出作用域都要调用其析构函数. 这会导致效率降低.

```cpp
while (const char* p = strchr(str, '/'))
{
    str = p + 1;
}
```

```cpp
Foo f;  // 构造函数和析构函数只调用 1 次
for (int i = 0; i < 1000000; ++i) 
{
    f.DoSomething(i);
}

// 低效的实现
for (int i = 0; i < 1000000; ++i) 
{
    Foo f;  // 构造函数和析构函数分别调用 1000000 次!
    f.DoSomething(i);
}
```

### 4. 静态和全局变量 （static 的使用）</strong>

静态生存周期的对象包括：全局变量，静态变量，静态类成员变量和函数静态变量。

都必须是原生数据类型 (POD : Plain Old Data): 即 int, char 和 float, 以及 POD 类型的指针、数组和结构体。

## 3 类

### 1. <strong>声明顺序</strong>

将相似的声明放在一起, 将 `public` 部分放在最前.

### 2. <strong>运算符重载</strong>

建议不使用使用运算符重载，若要使用提交 issue。

### 3. <strong>结构体 VS. 类</strong>

仅当只有数据成员时使用 `struct`, 其它一概使用 `class`.

### 4. <strong>隐式类型转换</strong>

使用 C++ 的类型转换, 如 `static_cast<>()`. 不要使用 `int y = (int)x` 或 `int y = int(x)` 等转换方式;

不要使用 C 风格类型转换. 而应该使用 C++ 风格.

> - 用 `static_cast` 替代 C 风格的值转换, 或某个类指针需要明确的向上转换为父类指针时.<br/>- 用 `const_cast` 去掉 `const` 限定符.<br/>- 用 `reinterpret_cast` 指针类型和整型或其它指针之间进行不安全的相互转换. 仅在你对所做一切了然于心时使用.

### 5. <strong>继承</strong>

1. 所有继承必须是 `public` 的. 如果你想使用私有继承, 你应该替换成把基类的实例作为成员对象的方式.
2. 不要过度使用实现继承. 组合常常更合适一些.

```cpp
class Engine {
public:
    void start() { /* 启动引擎 */ }
};

class Car {
public:
    Car(const std::string& n, Engine& e) : name_(n), engine_(e) {}
    void drive() { engine_.start(); std::cout << name_ << " is driving\n"; }

private:
    std::string name_;
    Engine& engine_;
};

int main() {
    Engine engine;
    Car car("mycar", engine);
    car.drive();
    return 0;
}
```

1. 如果你的类有虚函数, 则析构函数也应该为虚函数.

## 4 函数

### 1. 函数返回值

函数的返回 值，多使用 值返回 和 引用返回，尽量避免使用指针返回。

### 2. <strong>编写简短函数</strong>

我们倾向于编写简短, 凝练的函数.

### 3. <strong>缺省参数（默认参数）</strong>

少使用缺省函数参数，尽可能改用函数重载。

## 5 其他 C++ 特性

### 1. <strong>引用参数</strong>

如果这个引用参数不会变化，那么必须加上 `const`.

### 2. <strong>前置自增和自减</strong>

对于迭代器和其他模板对象使用前缀形式 (`++i`) 的自增, 自减运算符.

不考虑返回值的话, 前置自增 (`++i`) 通常要比后置自增 (`i++`) 效率更高.

### 3. Const 用法

强烈建议在任何可能的情况下都要使用 `const`.

### 4. <strong>sizeof</strong>

尽可能用 `sizeof(a)` 代替 `sizeof(int)`.

### 5. <strong>列表初始化</strong>

C++11 中，该特性得到进一步的推广，任何对象类型都可以被列表初始化。示范如下：

```cpp
// Vector 接收了一个初始化列表。
// 不考虑细节上的微妙差别，大致上相同。
// 可以任选其一。
vector<string> v{"foo", "bar"};
vector<string> v = {"foo", "bar"};


// 可以配合 new 一起用。
auto p = new vector<string>{"foo", "bar"};


// map 接收了一些 pair, 列表初始化大显神威。
map<int, string> m = {{1, "one"}, {2, "2"}};


// 初始化列表也可以用在返回类型上的隐式转换。
vector<int> test_function() { return {1, 2, 3}; }


// 初始化列表可迭代。
for (int i : {-1, -2, -3}) {}


// 在函数调用里用列表初始化。
void TestFunction2(vector<int> v) {}
TestFunction2({1, 2, 3});
```

### 6. <strong>0, </strong><strong>nullptr</strong><strong> 和 </strong><strong>NULL</strong>

整数用 `0`, 实数用 `0.0`, 指针用 `nullptr` 或 `NULL`, 字符 (串) 用 `'\0'`.

### 7. 关于 auto

- 陈老师建议：所有人不用 auto！

auto 在 C++11 中引入了自动类型推导，C++14 扩展了其用法以支持函数返回类型的自动推导，C++17 进一步扩展了其用法以支持结构化绑定和 lambda 表达式的参数类型的自动推导。

```cpp
// C++11
auto x = 42; // x的类型为int
auto y = 3.14; // y的类型为double
```

```cpp
// C++14
auto add(int x, int y) 
{ 
    return x + y;  // add函数返回类型为int
}
```

```cpp
// C++17
std::pair<int, double> p{42, 3.14};
auto [x, y] = p; // x的值为42，y的值为3.14
```

但是大家注意，在 abacus 中，我们只支持 C++11 的标准，C++14/17 都是不接受的。

#### Auto 和 for 的混合使用

在 C++11 中，auto 和 for 循环的结合使用已成为一种常见的编程范式，它可以让代码更加简洁、易读，并且减少了手动指定类型的错误。

```cpp
int arr[] = {1, 2, 3};

for (auto i : arr)  // 相当于复制
{
    std::cout << i << " "; 
}

for (const auto& i : arr) // 常量引用
{
    std::cout << i << " ";
}

for (auto& i : arr) // 能够实现对元素的直接修改。
{
    i *= 2;
}
```

### 8. <strong> </strong><strong>constexpr</strong><strong> 用法</strong>

在 C++11 里，用 constexpr 来定义真正的常量，或实现常量初始化。（真正的常量在编译时和运行时都不变）。

建议： `constexpr` 替代宏定义和 const 常量。

```cpp
#define PI 3.14159
const double kE = 2.71828;

constexpr double kGravity = 9.8;
```

- `PI` 是一个宏定义常量，它不会进行类型检查，容易出错；
- `kE` 是一个 const 常量，它不能用于编译期计算。

因此，`constexpr` 可以替代宏定义和 const 常量，提供更安全、更可读、更灵活的编译期常量和编译期计算功能。

## 6 命名约定

### 1. 变量命名

#### <strong>普通变量命名</strong>

普通变量的命名统一是小写，并使用下划线命名法。

不要使用 `驼峰命名法` 和 `匈牙利命名法`。（[四种命名方法都有哪些](https://zhuanlan.zhihu.com/p/89909623)）

```cpp
string table_name;  // 好 - 用下划线.
string tablename;   // 好 - 全小写.

string tableName;  // 差 - 混合大小写
```

#### <strong>类数据成员与结构体变量</strong>

- 结构体数据成员和普通变量的命名方式一样
- 类数据成员和普通变量的命名方式一样，但要在最后以下划线结尾，以区分自己是类数据成员。

```cpp
class TableInfo 
{
...
private:
    string table_name_;  // 好 - 后加下划线.
    string tablename_;   // 好.
    static Pool<TableInfo>* pool_;  // 好.
};
```

### 2. 常量命名

声明为 `constexpr` 或 `const` 的变量, 或在程序运行期间其值始终保持不变的, 命名时以 “k” 开头, 大小写混合. 例如:

```cpp
const int kDaysInAWeek = 7;
```

### 3. 函数命名

- 下划线命名法。
- 不可以超过 18 个字符
- 函数名用小写

### 4. ABACUS 中常用的关键词缩写

有些名字很长，我们希望尽量言简意赅的表达出一些关键词的意思。原则是一般 3-5 个字母的范围下尽量说清楚一个变量的含义。这些统一的命名会出现在函数名或者变量名里。

1. 2 个字符
2. pw，代表 plane wave 平面波
3. op，代表具有 multi-device 和 multi-precision 支持的算子（operator），和 Operator 模块含义不同
4. 3 个字符
5. fft：快速傅里叶变换
6. kpt：布里渊区 kpoint 的缩写
7. nao，代表 numerical atomic orbitals  （nao 经常用来表示 number of atomic orbitals，不知道会不会混）
8. orb：orbital，轨道
9. hmt，代表 hamilt 或者 hamiltonian
10. ovp, 代表 overlap （pyscf 中是 ovlp，我们是否需要保持一致？）
11. pot，代表 potential
12. chg，代表 charge
13. den，代表 density
14. scf，代表自洽迭代 self consistent field
15. thr，代表 threshold
16. tab，代表 table
17. fac，代表 factor
18. kin，代表 kinetic，动能的
19. mem，代表 memory
20. dst 代表 distance，dtb 代表 distribution，这样不容易混起来
21. cal，代表 calculate
22. opt，代表 optimize
23. gen，代表 generate
24. req，代表 request
25. 4 个字符
26. iter，代表 iteration
27. init，代表初始化 initializaiton
28. meth，代表 method
29. read，读入
30. stru，代表 structure
31. veff，代表有效势
32. vloc，代表局域势
33. 5 个字符
34. setup，设置
35. basis，基矢量
36. trsfm，代表 transform
37. ???, expansion
38. update
39. converge
40. before
41. after

## Reference

[C++ Style Guide](https://xmywuqhxb0.feishu.cn/docx/TcnidIWmLoEE1YxB2VmcZ1nBnhn)

## 待统一的问题（建议）

1. [auto 的使用](https://xmywuqhxb0.feishu.cn/docx/Xoh2dR0bwoRMoJxdpivcNSHOnBd#Hqqcdi6qaoOkgSxqILvcEm9Unef)，是否直接禁用？
2. 登辉师兄 和 我 都觉得 C++11 的 auto 在很多时候是很好用的，并且有利于避免隐式的类型转换。
3. <del>缺省参数是否有必要禁用，而改成函数重载？</del>
4. <del>当前 abacus 中有很多缺省参数的使用</del>
5. <del>有的先不改，</del>
6. 添加了 <strong>constexpr</strong><strong> 用法</strong>
7. 大家再看一下
8. 想要使用运算符重载，必须提交 issue 是否太过苛刻？
9. Bxk 沟通
10. <strong>关于代码格式化的问题：</strong>

我们当前很多修改都只是针对整个文件的一小部分，如果对整个文件格式化，就会引入不必要的修改，这个问题该怎么办？

1. 陈老师建议：改哪个文件就格式化哪个文件
2. 当修改完代码后先进行一次 commit，然后对于格式化该文件的时候，单独进行一次 commit 记录（例如：Format diago_cg.cpp file）。
3. Bxk 沟通
