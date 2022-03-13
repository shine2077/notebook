# memory ownership

# Expressing memory ownership

## Expressing non-ownership

非所有权
该对象在其他地方被实现，稍后将再其他地方被删除，访问该对象的程序不负责其的管理。

使用原始指针或者引用来访问非所有对象。

## Expressing exclusive ownership
独占所有权
程序创建一个对象，稍后由该程序负责删除。

** 方式一 ** 
局部（栈）变量
```c++
void Work() {
    Widget w;
    Transmogrify(w);
    Draw(w);
}
```

** 方式二 **
主要用于无法在栈上创建对象，必须在堆上创建对象。

栈分配的对象将在离开作用域后被删除。如果我们需要让对象存活更长时间，它必须分配到堆上。

在堆上创建对象的另一个原因是编译时可能不知道对象的大小或类型。这通常发生在对象是多态的——创建了派生对象，但使用了基类指针。

```c++
class FancyWidget : public Widget { ... };
std::unique_ptr<Widget> w(new FancyWidget);
}
```

## Expressing transfer of exclusive ownership
它不能被复制到另一个 unique_ptr，不能通过值传递给函数，也不能用于任何需要复制的 C++ 标准库算法。 只能移动 unique_ptr。 这意味着，内存资源所有权将转移到另一 unique_ptr，并且原始 unique_ptr 不再拥有此资源。

## Expressing shared ownership
共享所有权

经常被过度使用
自动删除不需要共享所有权，只需一个明确表达（unique 指针、数据成员和容器都提供自动删除）

缺点
shared_ptr的循环依赖问题
```c++
#include <iostream>
#include <memory> // for std::shared_ptr
#include <string>

class Person
{
    std::string m_name;
    std::shared_ptr<Person> m_partner; // initially created empty

public:

    Person(const std::string &name): m_name(name)
    {
        std::cout << m_name << " created\n";
    }
    ~Person()
    {
        std::cout << m_name << " destroyed\n";
    }

    friend bool partnerUp(std::shared_ptr<Person> &p1, std::shared_ptr<Person> &p2)
    {
        if (!p1 || !p2)
            return false;

        p1->m_partner = p2;
        p2->m_partner = p1;

        std::cout << p1->m_name << " is now partnered with " << p2->m_name << "\n";

        return true;
    }
};

int main()
{
 auto lucy = std::make_shared<Person>("Lucy"); // create a Person named "Lucy"
 auto ricky = std::make_shared<Person>("Ricky"); // create a Person named "Ricky"

 partnerUp(lucy, ricky); // Make "Lucy" point to "Ricky" and vice-versa

 return 0;
}
```
两个对象互相引用，导致析构函数无法执行
```c++
output:
Lucy created
Ricky created
Lucy is now partnered with Ricky
```
使用weak_ptr解决循环依赖问题
```c++
#include <iostream>
#include <memory> // for std::shared_ptr and std::weak_ptr
#include <string>

class Person
{
    std::string m_name;
    std::weak_ptr<Person> m_partner; // note: This is now a std::weak_ptr

public:

    Person(const std::string &name): m_name(name)
    {
    std::cout << m_name << " created\n";
    }
    ~Person()
    {
        std::cout << m_name << " destroyed\n";
    }

    friend bool partnerUp(std::shared_ptr<Person> &p1, std::shared_ptr<Person> &p2)
    {
        if (!p1 || !p2)
            return false;

        p1->m_partner = p2;
        p2->m_partner = p1;

        std::cout << p1->m_name << " is now partnered with " << p2->m_name << "\n";

        return true;
    }
};

int main()
{
 auto lucy = std::make_shared<Person>("Lucy");
 auto ricky = std::make_shared<Person>("Ricky");

 partnerUp(lucy, ricky);

 return 0;
}
```
weak_ptr不增加shared_ptr的引用计数
```c++
Lucy created
Ricky created
Lucy is now partnered with Ricky
Ricky destroyed
Lucy destroyed
```
