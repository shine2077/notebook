# 非类型模板参数

## 非类型类模板参数
```c++
#include <array>
#include <cassert>
 
template<typename T, std::size_t Maxsize>
class Stack {
  private:
    std::array<T,Maxsize> elems; // elements
    std::size_t numElems;        // current number of elements
  public:
    Stack();                     // constructor
    void push(T const& elem);    // push element
    void pop();                  // pop element
    T const& top() const;        // return top element
    bool empty() const {         //return whether the stack is empty
        return numElems == 0;
    }
    std::size_t size() const {    //return current number of elements
        return numElems;
    }
};
 
template<typename T, std::size_t Maxsize>
Stack<T,Maxsize>::Stack ()
 : numElems(0)                 //start with no elements
{
   // nothing else to do
}
 
template<typename T, std::size_t Maxsize>
void Stack<T,Maxsize>::push (T const& elem)
{
    assert(numElems < Maxsize);
    elems[numElems] = elem;    // append element
    ++numElems;                // increment number of elements
}
 
template<typename T, std::size_t Maxsize>
void Stack<T,Maxsize>::pop ()
{
    assert(!elems.empty());
    --numElems;                // decrement number of elements
}
 
template<typename T, std::size_t Maxsize>
T const& Stack<T,Maxsize>::top () const
{
    assert(!elems.empty());
    return elems[numElems-1];  // return last element
}
```
第二个模板参数`Maxsize`是一个整型量。

## 非类型函数模板参数
```c++
template<int Val, typename T>
T addValue (T x)
{
  return x + Val;
}
```
### 非类型模板参数的限制
一般只能是常整型量，指针类型，左值引用或者`std::nullptr_t`。
- 浮点型和类类型都不被允许。
```c++
template<double VAT>         // ERROR: floating-point values are not
double process (double v)    //        allowed as template parameters
{
    return v * VAT;
}
 
template<std::string name>   // ERROR: class-type objects are not
class MyClass {              //        allowed as template parameters
  …
};
```
- 字符串常量也无法作为模板参数
```c++
template<char const* name>
class MyClass {
  …
};
 
MyClass<"hello"> x;  //ERROR: string literal "hello" not allowed
```
- [链接性](https://stackoverflow.com/questions/1358400/what-is-external-linkage-and-internal-linkage)限制
```c++
extern char const s03[] = "hi";    // external linkage
char const s11[] = "hi";           // internal linkage
char const* s21 = "hi";
 
int main()
{
  Message<s03> m03;                // OK (all versions)
  Message<s11> m11;                // OK since C++11

  static char const s17[] = "hi";  // no linkage
  Message<s17> m17;                // OK since C++17

  Message<s21> m21; //ERROR
}
```
最后一条语句报错的原因是`s21`不是 [编译期常量](https://www.learncpp.com/cpp-tutorial/compile-time-constants-constant-expressions-and-constexpr/) 。