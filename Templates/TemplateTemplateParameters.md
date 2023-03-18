# Template Template Parameters

将一个模板参数表示一个类模板

```c++
template<typename T,
         template<typename Elem> class Cont = std::deque>
class Stack {
  private:
    Cont<T> elems;             // elements

  public:
    void push(T const&);       // push element
    void pop();                // pop element
    T const& top() const;      // return top element
    bool empty() const {       // return whether the stack is empty
        return elems.empty();
    }
    ...
};
```

其中`template<typename Elem> class Cont` 称为模板模板参数，该模板模板参数指代的默认类模板为`std::deque`。 

在C++11之前只能用`class`表示模板模板参数，C++17开始允许使用`typename`：

```c++
template<typename T,
         template<typename  Elem> typename  Cont = std::deque>
class Stack {                                 //OK
  ...
};
```

`typename Elem` 称为模板模板参数的模板参数，在没有被使用的时候可以省略其名字`Elem`:

```c++
template<typename T,
         template<typename> class Cont = std::deque>
class Stack {                                 //OK
  ...
};
```

在C++17之前，上述代码会报默认的`std::deque`没有匹配的模板模板参数`Cont`，因为模板`std::deque`有不止一个模板参数，在C++17 前编译器无法将其与只有一个模板参数的`Cont`匹配。因此需做出如下修改：

```c++
template<typename T,
         template< typename Elem,
                   typename Alloc = std::allocator<Elem> >
         class Cont = std::deque >
class Stack {
  private:
    Cont<T> elems;         // elements
    ...
};
```

Stack的完整代码如下：

```c++
#include <deque>
#include <cassert>
#include <memory>

#include <iostream>
#include <vector>

template<typename T,
         template<typename Elem,typename = std::allocator<Elem>>class Cont = std::deque>
class Stack 
{
private:
    Cont<T> elems;// elements

public:
    void push(T const&);      // push element
    void pop(); // pop element
    T const& top() const;  // return top element
    bool empty() const { // return whether the stack is empty
        return elems.empty();
    }

    // assign stack of elements of type T2
    template<typename T2, 
             template<typename Elem2,typename = std::allocator<Elem2>> class Cont2>
    Stack<T, Cont>& operator= (Stack<T2, Cont2> const&);
    // to get access to private members of any Stack with elements of type T2:
    template<typename, template<typename, typename>class>
    friend class Stack;
};

template<typename T, template<typename, typename> class Cont>
void Stack<T, Cont>::push(T const& elem)
{
    elems.push_back(elem); // append copy of passed elem
}

template<typename T, template<typename, typename> class Cont>
void Stack<T, Cont>::pop()
{
    assert(!elems.empty());
    elems.pop_back();          // remove last element
}

template<typename T, template<typename, typename> class Cont>
T const& Stack<T, Cont>::top() const
{
    assert(!elems.empty());
    return elems.back();       // return copy of last element
}

template<typename T, template<typename, typename> class Cont>
template<typename T2, template<typename, typename> class Cont2>
Stack<T, Cont>&
Stack<T, Cont>::operator= (Stack<T2, Cont2> const& op2)
{
    elems.clear();                       // remove existing elements
    elems.insert(elems.begin(),         // insert at the beginning
                 op2.elems.begin(),     // all elements from op2
                 op2.elems.end());
    return *this;
}

int main()
{
    Stack<int>   iStack;    // stack of ints
    Stack<float> fStack;    // stack of floats

    // manipulate int stack
    iStack.push(1);
    iStack.push(2);
    std::cout << "iStack.top(): " << iStack.top() << '\n';

    // manipulate float stack:
    fStack.push(3.3);
    std::cout << "fStack.top(): " << fStack.top() << '\n';

    // assign stack of different type and manipulate again
    fStack = iStack;
    fStack.push(4.4);
    std::cout << "fStack.top(): " << fStack.top() << '\n';

    // stack for doubless using a vector as an internal container
    Stack<double, std::vector> vStack;
    vStack.push(5.5);
    vStack.push(6.6);
    std::cout << "vStack.top(): " << vStack.top() << '\n';

    vStack = fStack;
    std::cout << "vStack: ";
    while (!vStack.empty()) {
        std::cout << vStack.top() << ' ';
        vStack.pop();
    }
    std::cout << '\n';
}
```
