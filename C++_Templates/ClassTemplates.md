# 类模板

## 类模板栈
```c++
#include <vector>
#include <cassert>

template<typename T>
class Stack {
private:
     std::vector<T> elems;      // elements

public:
     void push(T const& elem);  // push element
     void pop();                // pop element
     T const& top() const;      // return top element
     bool empty() const 
     {       // return whether the stack is empty
        return elems.empty();
     }
};

template<typename T>
void Stack<T>::push (T const& elem)
{
   elems.push_back(elem);      // append copy of passed elem
}

template<typename T>
void Stack<T>::pop ()
{
   assert(!elems.empty());
   elems.pop_back();           // remove last element
}

template<typename T>
T const& Stack<T>::top () const
{
   assert(!elems.empty());
   return elems.back();        // return copy of last element
}
```

### 声明类模板
```c++
template<typename T>
class Stack {
...
};
```
在我们手动声明拷贝和赋值构造函数时，以下两种方式是等价的
```c++
//第一种
template<typename T> 
class Stack {
   ...
   Stack (Stack const&);                    // copy constructor
   Stack& operator= (Stack const&);         // assignment operator
   ...
};
//第二种
template<typename T> 
class Stack {
   ...
   Stack (Stack<T> const&);                  // copy constructor
   Stack<T>& operator= (Stack<T> const&);    // assignment operator
   ...
};
```
但通常`<T>`表示对特殊模板参数的特殊处理，所以通常使用第一种方式会更好。


## 类模板栈的使用

```c++
#include "stack1.hpp"
#include <iostream>
#include <string>

int main()
{
  Stack< int>       intStack;      // stack of ints
  Stack<std::string> stringStack;  // stack of strings

  // manipulate int stack
  intStack.push(7);
  std::cout << intStack.top() << ’\n’;

  // manipulate string stack
  stringStack.push("hello");
  std::cout << stringStack.top() << ’\n’;
  stringStack.pop();
}
```

## 友元

## 类模板的特化
```c++
#include "stack1.hpp"
#include <deque>
#include <string>
#include <cassert>

template<>
class Stack<std::string> {
   private:
      std::deque<std::string> elems;    // elements

   public: 
      void push(std::string const&);    // push element 
      void pop();                       // pop element
      std::string const& top() const;   // return top element 
      bool empty() const {              // return whether the stack is empty 
          return elems.empty();
   }
};

void Stack<std::string>::push (std::string const& elem)
{
   elems.push_back(elem);     // append copy of passed elem
}

void Stack<std::string>::pop ()
{
   assert(!elems.empty());
   elems.pop_back();          // remove last element
}

std::string const& Stack<std::string>::top () const
{
   assert(!elems.empty()); 
   return elems.back();      // return copy of last element
}
```

## 类模板的偏特化
```c++
#include "stack1.hpp"

// partial specialization of class Stack<> for pointers:
template<typename T> 
class Stack<T*> { 
   private:
      std::vector<T*> elems;     // elements

public: 
   void push(T*);                // push element
   T* pop();                     // pop element
   T* top() const;               // return top element 
   bool empty() const {          // return whether the stack is empty 
      return elems.empty();
   }
};

template<typename T>
void Stack<T*>::push (T* elem)
{
   elems.push_back(elem);        // append copy of passed elem
}
template<typename T>
T* Stack<T*>::pop ()
{
    assert(!elems.empty());
    T* p = elems.back();
    elems.pop_back();           // remove last element 
    return p;                   // and return it (unlike in the general case)
}

template<typename T>
T* Stack<T*>::top () const
{
   assert(!elems.empty()); 
   return elems.back();        // return copy of last element
}
```

```c++
Stack< int*> ptrStack;        // stack of pointers (special implementation)

ptrStack.push(new int{42});
std::cout << *ptrStack.top() << ’\n’; 
delete ptrStack.pop();
```

多模板参数的偏特化
```c++
template<typename T1, typename T2> 
class MyClass {
   ...
};

// partial specialization: both template parameters have same type 
template<typename T> 
class MyClass<T,T> {
   ...
};

// partial specialization: second type is int 
template<typename T> 
class MyClass<T,int> {
   ...
};

// partial specialization: both template parameters are pointer types 
template<typename T1, typename T2> 
class MyClass<T1*,T2*> {
   ...
};
```

```c++
MyClass< int, float> mif;      // uses MyClass<T1,T2>
MyClass< float, float> mff;    // uses MyClass<T,T>
MyClass< float, int> mfi;      // uses MyClass<T,int>
MyClass< int*, float*> mp;     // uses MyClass<T1*,T2*>
```
如果一个实例化匹配了多个偏特化则会报错

```c++
MyClass< int, int> m;             // ERROR1: matches MyClass<T,T>and MyClass<T,int>
MyClass< int*, int*> m;         // ERROR2: matches MyClass<T,T>and MyClass<T1*,T2*>
```
为了解决多匹配报错，需要定义一个能唯一匹配的特化版本
```c++
//全特化解决ERROR1
template<> 
class MyClass<int,int> {
   ...
};
//指针类型的偏特化解决ERROR2
template<typename T> 
class MyClass<T*,T*> {
   ...
};
```
## 默认类模板参数
```c++
#include <vector>
#include <cassert>

template<typename T, typename Cont = std::vector<T>>
class Stack { 
   private:
      Cont elems;                // elements

   public: 
      void push(T const& elem);  // push element 
      void pop();                // pop element
      T const& top() const;      // return top element 
      bool empty() const {       // return whether the stack is empty
            return elems.empty();
   }
};

template<typename T, typename Cont>
void Stack<T,Cont>::push (T const& elem)
{
   elems.push_back(elem);        // append copy of passed elem
}

template<typename T, typename Cont>
void Stack<T,Cont>::pop ()
{
   assert(!elems.empty());
   elems.pop_back();             // remove last element
}

template<typename T, typename Cont>
T const& Stack<T,Cont>::top () const
{
   assert(!elems.empty()); 
   return elems.back();        // return copy of last element
}
```

```c++
#include "stack3.hpp"
    
#include <iostream>
    
#include <deque>
int main()
{
  // stack of ints using default std::vector<>
  Stack< int> intStack;
  // stack of doubles using a std::deque<> to manage the elements
  Stack< double,std::deque< double>> dblStack;

  // manipulate int stack
  intStack.push(7);
  std::cout << intStack.top() << ’\n’;
  intStack.pop();

  // manipulate double stack
  dblStack.push(42.42);
  std::cout << dblStack.top() << ’\n’;
  dblStack.pop();
}
```

## 类型别名
在C++11之前
```c++
typedef Stack
typedef Stack<int> IntStack;     // typedef
void foo (IntStack const& s);    // s is stack ofints
IntStack istack[10];             // istack is array of 10 stacks of ints
```
C++11
```c++
using IntStack = Stack <int>;        // alias declaration 
void foo (IntStack const& s);        // s is stack of ints
IntStack istack[10];                 // istack is array of 10 stacks of ints
```
别名模板
```c++
template<typename T> 
using DequeStack = Stack<T, std::deque<T>>;
```
`DequeStack<int>` 等价于 `Stack<int, std::deque<int>>` 

成员类型的别名模板
```c++
struct MyType {
  using iterator = ...;
  ...
};

template<typename T> 
using MyTypeIterator = typename MyType<T>::iterator;

MyTypeIterator<int> pos;
```

## 类模板参数推导
在C++17标准中，一个类模板的类型模板参数也变得可以推导。
```c++
Stack<int> intStack1;                 // stack of int
Stack<int> intStack2 = intStack1;     // OK in all cpp standred
Stack intStack3 = intStack1;          // OK since C++17
```
通过拷贝构造函数不用写明模板参数。

通过提供传递一些初始参数的构造函数，可以推导栈的元素类型。例如提供一个可以由单参数初始化的栈。
```c++
template<typename T> 
class Stack { 
   private:
     std::vector<T> elems;        // elements 
   public:
     Stack () = default;
     Stack (T const& elem)        // initialize stack with one element
       : elems({elem}) {
     }
     ...
};
```
```c++
Stack intStack = 0;              // Stack<int> deduced since C++17
```
### 类模板参数推导与字符字面量的关系
```c++
template<typename T> 
class Stack { 
    private:
       std::vector<T> elems;        // elements 
    public:
       Stack (T elem)               // initialize stack with one element by value
        : elems({std::move(elem)}) {
    }
   ...
};
```
```c++
Stack stringStack{"bottom"}； // Stack<char const[7]> deduced since C++17
Stack stringStack = "bottom"; // Stack<char const[7]> deduced since C++17
```

### 推导指南
C++ 17标准提出的概念。推导指南主要用来在推导类模板参数时提供推导指引
```c++
Stack(char const*) -> Stack<std::string>;
```
```c++
Stack stringStack{"bottom"};        // OK: Stack<std::string> deduced since C++17
```
指引编译器将`Stack<char const[7]>`推导为`Stack<std::string>`。
```c++
Stack stringStack = "bottom";       // Stack<std::string> deduced, but still not valid
```
定义了推导指南，推导完成但无法完成实例化，编译器报错不存在从const char[7]转换到`Stack<std::string>`的适当构造函数