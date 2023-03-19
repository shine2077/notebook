# Perfect Forwarding

## 值类型（C++11）

![值类型](./../img/bfig01.jpg "值类型")

glvalue(泛左值),简单来说是一个具名对象，都够确定对象、位域或者函数的标识的表达式

- xvalue(将亡值)，表示资源可以被重用的对象或位域。通常其接近生命周期结束，或者通过右值引用转化过来
- lvalue(左值),除了非将亡值的泛左值
  
右值，包括xvalue(将亡值)和纯右值

- 纯右值，能够用于初始化对象和位域，或者能够计算运算符操作数的值的表达式

```c++
// x here is a variable, not an lvalue. 3 is a prvalue initializing
// the variable x.
int x = 3;  
// x here is an lvalue. The evaluation of that lvalue expression does not
// produce the value 3, but a designation of an object containing the value 3.
// That lvalue is then then converted to a prvalue, which is what initializes y.
int y = x;  
```

## 完美转化

不使用模板来传递不同值类型的参数

```c++
#include <utility>
#include <iostream>

class X {
public:
    X() {};
};

void g(X&) {
    std::cout << "g() for variable\n";
}
void g(X const&) {
    std::cout << "g() for constant\n";
}
void g(X&&) {
    std::cout << "g() for movable object\n";
}

// let f() forward argument val to g():
void f(X& val) {
    g(val);             // val is non-const lvalue => calls g(X&)
}
void f(X const& val) {
    g(val);             // val is const lvalue => calls g(X const&)
}
void f(X&& val) {
    g(std::move(val));  // val is non-const lvalue => needs std::move() to call g(X&&)
    g(val);  // val is non-const lvalue => calls g(X&)
}

int main()
{
    X v;              // create variable
    X const c;        // create constant

    f(v);             // f() for nonconstant object calls f(X&) => calls g(X&)
    f(c);             // f() for constant object calls f(X const&) => calls g(X const&)
    f(X());           // f() for temporary calls f(X&&) => calls g(X&&)
    f(std::move(v));  // f() for movable variable calls f(X&&) => calls g(X&&)
}
```

尽管第三个f()中的val被声明为右值引用，但当它被用作表达式时，它的值类别是一个非常数左值。（why?）

将三种函数`f`用一种通用的代码来表示

```c++

//ERROR
//只能表示前两种，无法传递移动对象
template<typename T>
void f (T val) {
  g(T);
}

// perfect forwarding
template<typename T>
void f (T&& val) {
  g(std::forward<T>(val));  // perfect forward val to g()
}
```

```c++
#include <utility>
#include <iostream>

class X {
public:
    X() {};
};

void g(X&) {
    std::cout << "g() for variable\n";
}
void g(X const&) {
    std::cout << "g() for constant\n";
}
void g(X&&) {
    std::cout << "g() for movable object\n";
}

template<typename T>
void f(T&& val) {
    g(std::forward<T>(val));  // perfect forward val to g()
}

int main()
{
    X v;              // create variable
    X const c;        // create constant

    f(v);             // f() for nonconstant object calls f(X&) => calls g(X&)
    f(c);             // f() for constant object calls f(X const&) => calls g(X const&)
    f(X());           // f() for temporary calls f(X&&) => calls g(X&&)
    f(std::move(v));  // f() for movable variable calls f(X&&) => calls g(X&&)
}
```
