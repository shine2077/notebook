# 函数模板

## 1.函数模板定义

由于历史原因，关键字`class`而不是`tyename`都可以来定义一个类型参数。关键字`tyename`在C++98标准的演变中出现得比较晚。在此之前，关键字`class`是引入类型参数的唯一方法，而且现在仍然是一种有效的方法。

```c++
template<typename T>
T max (T a, T b)
{
    // if b < a then yield a else yield b 
    return b < a ? a : b;
}

// 上下俩种都是可以的

template<class T>
T max (T a, T b)
{
    return b < a ? a : b;
}
```

然而，由于`class`可能会产生误导（不仅是类类型可以被替换为`T`），最好使用`typename`。

### 1.1 函数模板的使用

```c++
#include "max1.hpp"
#include <iostream>
#include <string>
int main()
{
    int i = 42;
    std::cout << "max(7,i):      " << ::max(7,i) << ’\n’;

    double f1 = 3.4; double f2 = -6.7;
    std::cout << "max(f1,f2):    " << ::max(f1,f2) << ’\n’;

    std::string s1 = "mathematics"; std::string s2 = "math";
    std::cout << "max(s1,s2):    " << ::max(s1,s2) << ’\n’;
}
```
以上函数模板`max`都使用`::`来限定,确保函数模板`max`在全局命名空间（`global namespace`）,防止调用了标准模板库中的函数`std::max()`。函数模板`max`编译了3个不同的函数实体，分别是`int max(int, int)`， `double max(double,double)`，`std::string max(std::string, std::string) `。

### 1.2 模板的编译分为2阶段

*definition time* 

忽略模板参数进行检查，检查内容包括：

- 检查语法错误
- 检查使用不依赖模板参数的未知名称（类型名、函数名...）
- 检查不依赖模板参数的静态断言


*instantiate time*

整体检查


```c++
int main()
{
    std::complex<float> c1, c2; // doesn’t provide operator <
    … 
    ::max(c1,c2);               // ERROR at compile time
}
```
如果试图为一个不支持操作的类型实例化(instantiate)模板，将导致编译时错误。

# 2.模板参数推导

```c++
template<typename T>
T max (T const& a, T const& b)
{
    return b < a ? a : b;
}
```
当传递`int`时，`T`被推导成`int`

## 2.1 在类型推导时的类型转化

```c++
max(4, 7.2);        // ERROR: T can be deduced as int or double

//Cast the arguments so that they both match
max(static_cast<double>(4), 7.2);        // OK

// Specify (or qualify) explicitly the type of T to prevent the compiler from attempting type deduction
max<double>(4, 7.2);                   // OK

std::string s;
foo("hello", s);    //ERROR: T can be deduced as char const[6] or std::string

```

## 2.2 默认参数的类型推导

类型推导对默认参数不起作用
```c++
template<typename T>
void f(T = "");
…

f(1);        // OK: deduced T to be int, so that it calls f<int>(1)
f();         // ERROR: cannot deduce T
```
为模板参数声明一个默认参数

```c++
template<typename T = std::string>
void f(T = "");
…

f();        // OK
```

# 3. 多参数模板

```c++
template<typename T1, typename T2>
T1 max (T1 a, T2 b)
{
    return b < a ? a : b; 
}
…
auto m = ::max(4, 7.2);        // OK, but type of first argument defines return type
```
如何确定返回值的类型：

- 使用第三个参数模板作为返回类型
- 让编译器决定
- 声明返回类型是两个参数类型的 "共同类型"

## 3.1 返回类型的参数模板
使用第三个参数模板作为返回类型
```c++
template<typename T1, typename T2, typename RT>
RT max (T1 a, T2 b);
...
::max<int,double,double>(4, 7.2)

```
`T1`和`T2`可以根据传入的参数推导出类型，而返回类型无法在编译时被推导，因此必须明确指定模板参数列表中的所有模板参数。

```c++
template<typename RT, typename T1, typename T2>
RT max (T1 a, T2 b);
...
::max<double>(4, 7.2)        //OK: return type is double, T1 and T2 are deduced
```
一旦从某个模板参数开始推断，后续的所有模板参数都需要让编译器推导。因此可以明确指定第一个模板参数类型作为模板参数，后面的两个模板参数可以通过推导确定。

## 3.2 推导返回类型
从C++14开始，返回类型可以让编译器决定（必须声明返回类型为[auto](../Common/auto.md)）。
```c++
template<typename T1, typename T2>
auto max (T1 a, T2 b)
{
  return b < a ? a : b;
}
```
在C++11中需要搭配[decltype](../Common/auto.md)来实现返回类型推导
```c++
template<typename T1, typename T2>
auto max (T1 a, T2 b) -> decltype(b<a?a:b)
{
  return b < a ? a : b;
}
```
## 3.3 共同类型
```c++
#include <type_traits>
template<typename T1, typename T2>
std::common_type_t<T1,T2> max (T1 a, T2 b)
{
  return b < a ? a : b;
}
```

# 4. 默认模板参数


# 5. 重载函数模板