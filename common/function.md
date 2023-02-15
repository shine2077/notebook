# 一些避免使用指针的策略
** 首选在栈上构造对象而不是在堆上 **

```c++
#include "Customer.h"
//...
Customer customer;
```
在栈上创建`Customer`类实例，如果离开其作用域，实例则会自动销毁。
如果必须将在函数和方法中创建的对象返回调用者
在旧式c++中，需要在堆上创建对象，然后从函数中返回该实例的指针
在c++11中，可以将对象直接作为返回值返回
```c++
Customer createDefaultCustomer(){
    Customer customer;
    //Do something...
    return customer;
}
```
