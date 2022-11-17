# memory ownership

# Expressing memory ownership

## Expressing non-ownership

������Ȩ
�ö����������ط���ʵ�֣��Ժ��������ط���ɾ�������ʸö���ĳ��򲻸�����Ĺ���

ʹ��ԭʼָ��������������ʷ�����Ȩ����

## Expressing exclusive ownership
��ռ����Ȩ
���򴴽�һ�������Ժ��ɸó�����ɾ����

** ��ʽһ ** 
�ֲ���ջ������
```c++
void Work() {
    Widget w;
    Transmogrify(w);
    Draw(w);
}
```

** ��ʽ�� **
��Ҫ�����޷���ջ�ϴ������󣬱����ڶ��ϴ�������

ջ����Ķ������뿪�������ɾ�������������Ҫ�ö��������ʱ�䣬��������䵽���ϡ�

�ڶ��ϴ����������һ��ԭ���Ǳ���ʱ���ܲ�֪������Ĵ�С�����͡���ͨ�������ڶ����Ƕ�̬�ġ����������������󣬵�ʹ���˻���ָ�롣

```c++
class FancyWidget : public Widget { ... };
std::unique_ptr<Widget> w(new FancyWidget);
```

### Expressing transfer of exclusive ownership
�����ܱ����Ƶ���һ�� unique_ptr������ͨ��ֵ���ݸ�������Ҳ���������κ���Ҫ���Ƶ� C++ ��׼���㷨�� ֻ���ƶ� unique_ptr�� ����ζ�ţ��ڴ���Դ����Ȩ��ת�Ƶ���һ unique_ptr������ԭʼ unique_ptr ����ӵ�д���Դ��

## Expressing shared ownership
��������Ȩ

����������ʹ��
�Զ�ɾ������Ҫ��������Ȩ��ֻ��һ����ȷ��unique ָ�롢���ݳ�Ա���������ṩ�Զ�ɾ����

ȱ��
shared_ptr��ѭ����������
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
�������������ã��������������޷�ִ��
```c++
output:
Lucy created
Ricky created
Lucy is now partnered with Ricky
```
ʹ��weak_ptr���ѭ����������
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
weak_ptr������shared_ptr�����ü���
```c++
Lucy created
Ricky created
Lucy is now partnered with Ricky
Ricky destroyed
Lucy destroyed
```
