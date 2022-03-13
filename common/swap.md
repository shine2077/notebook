## When and why to use swap ##

** most import **
exception-safe
抛出异常不应该让程序处于未定义状态
Effective C++ 条款29

## How to implement and use swap ##
 ### Implementing swap ###
 声明成员函数swap()
 ```c++
 class C {
    public:
    void swap(C& rhs) noexcept;
};
 ```
 定义
 1.simply
```c++
#include <utility> // <algorithm> before C++11
...
class C {
    public:
    void swap(C& rhs) noexcept {
        using std::swap; // Brings in std::swap into this scope
        v_.swap(rhs.v_);
        swap(i_, rhs.i_); // Calls std::swap
    }
    ...
    private:
    std::vector<int> v_;
    int i_;
};
```
2.pimpl idiom
```c++
// In the header C.h:
class C_impl; // Forward declaration
class C {
    public:
    void swap(C& rhs) noexcept {
        swap(pimpl_, rhs.pimpl_);
    }
    void f(...); // Declaration only
    ...
    private:
    C_impl* pimpl_;
};

// In the C file:
class C_impl {
    ... real implementation ...
};
void C::f(...) { pimpl_->f(...); } // Actual implementation of C::f()
```



