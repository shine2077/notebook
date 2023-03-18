# Variadic Templates

## 可变参数模板

```c++
#include <iostream>
 
void print ()
{
}
 
template<typename T, typename... Types>
void print (T firstArg, Types... args)
{
  std::cout << firstArg << '\n';  //print first argument
  print(args...);                 // call print() for remaining arguments
}
```

```c++
std::string s("world");
print (7.5, "hello", s);
```

`Types` 称为模板参数包

`args` 称为函数参数包

输出

```c++
7.5
hello
world
```

通过递归调用`print`函数，最后调用`print`的非模板重载

```c++
print<double, char const*, std::string> (7.5, "hello", s)
=================================1=============================
    std::cout << 7.5 << ’\n’;
    print<char const*, std::string> ("hello", s)
=================================2============================
        std::cout << "hello" << ’\n’;
        print<std::string> (s)
=================================3============================
            std::cout << s << ’\n’;
            print()
```

## 重载可变和非可变参数模板

上面的例子可以改写如下

```c++
#include <iostream>
 
template<typename T>
void print (T arg)
{
  std::cout << arg << ’\n’;  //print passed argument
}
 
template<typename T, typename... Types>
void print (T firstArg, Types... args)
{
  print(firstArg);                // call print() for the first argument
  print(args...);                 // call print() for remaining arguments
}
```

### `sizeof...` 操作符

```c++
template<typename T, typename… Types>
void print (T firstArg, Types… args)
{
  std::cout << sizeof...(Types) << '\n';  //print number of remaining types
  std::cout << sizeof...(args) << '\n';   //print number of remaining args
  …
}
```

```c++
template<typename T, typename... Types>
void print (T firstArg, Types... args)
{
  std::cout << firstArg << ’\n’;
  if (sizeof...(args) > 0) {      // error if sizeof…(args)==0
    print(args...);               // and no print() for no arguments declared
  }
}
```

## 折叠表达式(C++17)

```c++
template<typename... T>
auto foldSum (T... s) {
  return (... + s);   // ((s1 + s2) + s3) …
}
```

折叠规则

- `( ... op pack )` -> `( ( ( pack1 op pack2 ) op pack3 ) ... op packN  )`
- `( pack op ... )` -> `( pack1 op ( ... ( packN-1 op packN ) ) )`
- `( init op ... op pack )` -> `( ( ( init op pack1 ) op pack2 ) ... op packN )`
- `( pack op ... op init )` -> `( pack1 op ( ... ( packN op init ) )  )`

使用折叠表达式反转链表

```c++
// define binary tree structure and traverse helpers:
struct Node {
  int value;
  Node* left;
  Node* right;
  Node(int i=0) : value(i), left(nullptr), right(nullptr) {
  }
  ...
};
auto left = &Node::left;
auto right = &Node::right;
 
// traverse tree, using fold expression:
template<typename T, typename... TP>
Node* traverse (T np, TP... paths) {
  return (np ->* ... ->* paths); // np ->* paths1 ->* paths2 …
}
 
int main()
{
  // init binary tree structure:
  Node* root = new Node{0};
  root->left = new Node{1};
  root->left->right = new Node{2};
  ...
  // traverse binary tree:
  Node* node = traverse(root, left, right);
  ...
}
```

只有一下三种运算符允许参数包为空

- `&&` 结果为 `true`
- `||` 结果为 `false`
- `,`  结果为 `void()`

使用折叠表达式改写`print`

```c++
template<typename T>
class AddSpace
{
private:
    T const& ref;                  // refer to argument passed in constructor
public:
    AddSpace(T const& r) : ref(r) {
    }
    friend std::ostream& operator<< (std::ostream& os, AddSpace<T> s) {
        return os << s.ref << ' ';   // output passed argument and a space
    }
};

template<typename... Types>
void print(Types const&... args)
{
  (std::cout << ... << AddSpace(args) ) << std::endl;
}

int main()
{
  std::string s("world");
  print(7.5, "hello", s);
}
```

输出

```c++
7.5 hello world
```

## 可变参表达式

定义函数`printDoubled (T const&... args)`

```c++
template<typename... T>
void printDoubled (T const&... args)
{
  print (args + args...);
}
```

一下两段代码作用一样

```c++
printDoubled(7.5, std::string("hello"), std::complex<float>(4,2));

print(7.5 + 7.5,
      std::string("hello") + std::string("hello"),
      std::complex<float>(4,2) + std::complex<float>(4,2) );
```

可变表达式可以对参数包中所有参数进行运算。

需要注意，省略号中的点不能直接跟在数字字面后面。

```c++
template<typename... T>
void addOne (T const&... args)
{
  print (args + 1...);    // ERROR: 1... is a literal with too many decimal points
  print (args + 1 ...);   // OK
  print ((args + 1)...);  // OK
}
```

编译时表达式可以用同样的方式处理模板参数包

```c++
template<typename T1, typename… TN>
constexpr bool isHomogeneous (T1, TN...)
{
  return (std::is_same<T1,TN>::value && ...);  // since C++17
}

isHomogeneous(43, -1, "hello")
```

输出

```c++
false
```

可变索引

```c++
template<typename C, typename... Idx>
void printElems (C const& coll, Idx... idx)
{ 
  print (coll[idx]...);
}
//非类型模板
template<std::size_t... Idx, typename C>
void printIdx (C const& coll)
{
  print(coll[Idx]...);
}
```

```c++
std::vector<std::string> coll = {"good", "times", "say", "bye"}; 
//等效
printElems(coll,2,0,3);
print (coll[2], coll[0], coll[3])
printIdx<2,0,3>(coll);
```

## 可变参类模板

```c++
#include <string>
 
class Customer
{
  private:
    std::string name;
  public:
    Customer(std::string const& n) : name(n) { }
    std::string getName() const { return name; }
};
 
struct CustomerEq {
    bool operator() (Customer const& c1, Customer const& c2) const {
      return c1.getName() == c2.getName();
    }
};
 
struct CustomerHash {
    std::size_t operator() (Customer const& c) const {
      return std::hash<std::string>()(c.getName());
    }
};
 
// define class that combines operator() for variadic base classes:
template<typename... Bases>
struct Overloader : Bases...
{
      using Bases::operator()...;  // OK since C++17
}; 
 
int main()
{
    // combine hasher and equality for customers in one type:
    using CustomerOP = Overloader<CustomerHash, CustomerEq>;

    std::string c1_name{ "sun" };
    std::string c2_name{ "hui" };
    Customer c1(c1_name);
    Customer c2(c2_name);
    CustomerOP op;
    std::cout << op(c1, c2) <<std::endl;
    std::cout << op(c1) << std::endl;
    std::cout << op(c2) << std::endl;
}
```

## 可变参推导指南

在标准库中，`std::array` 定义了如下推导指南：
```c++
namespace std {
  template<typename T, typename... U> array(T, U...)
    -> array<enable_if_t<(is_same_v<T, U> && ...), T>,
             (1 + sizeof...(U))>;
}
```

[std::enable_if](https://en.cppreference.com/w/cpp/types/enable_if) 定义在头文件<type_traits>

```c++
template< bool B, class T = void >
struct enable_if;
```

如果 B 为真，则 std::enable_if约束类型为T；否则没有类型。

折叠表达式`（is_same_v<T, U> && ...)`展开如下：

```c++
is_same_v<T, U1> && is_same_v<T, U2> && is_same_v<T, U3> …
```

```c++
std::array a{42,45,77};
//推导如下
std::array<int, 3> a{42,45,77};
```
