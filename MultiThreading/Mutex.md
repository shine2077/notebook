# Mutual exclusion

## std::mutex

```c++
#include <iostream>
#include <map>
#include <string>
#include <chrono>
#include <thread>
#include <mutex>
 
std::map<std::string, std::string> g_pages;
std::mutex g_pages_mutex;
 
void save_page(const std::string &url)
{
    // simulate a long page fetch
    std::this_thread::sleep_for(std::chrono::seconds(2));
    std::string result = "fake content";
 
    std::lock_guard<std::mutex> guard(g_pages_mutex);
    g_pages[url] = result;
}
 
int main() 
{
    std::thread t1(save_page, "http://foo");
    std::thread t2(save_page, "http://bar");
    t1.join();
    t2.join();
 
    // safe to access g_pages without lock now, as the threads are joined
    for (const auto &pair : g_pages) {
        std::cout << pair.first << " => " << pair.second << '\n';
    }
}
```

`std::lock_guard` 是一个互斥锁包装器，它提供了一种方便的 RAII 机制，用于在作用域内获得互斥锁，离开作用域自动释放互斥锁。

当然也可以手动调用`lock` `try_lock` 来加锁，`unlock`来解锁

```c++
#include <chrono>
#include <mutex>
#include <thread>
#include <iostream> // std::cout
 
std::chrono::milliseconds interval(100);
 
std::mutex mutex;
int job_shared = 0; // both threads can modify 'job_shared',
    // mutex will protect this variable
 
int job_exclusive = 0; // only one thread can modify 'job_exclusive'
    // no protection needed
 
// this thread can modify both 'job_shared' and 'job_exclusive'
void job_1() 
{
    std::this_thread::sleep_for(interval); // let 'job_2' take a lock
 
    while (true) {
        // try to lock mutex to modify 'job_shared'
        if (mutex.try_lock()) {
            std::cout << "job shared (" << job_shared << ")\n";
            mutex.unlock();
            return;
        } else {
            // can't get lock to modify 'job_shared'
            // but there is some other work to do
            ++job_exclusive;
            std::cout << "job exclusive (" << job_exclusive << ")\n";
            std::this_thread::sleep_for(interval);
        }
    }
}
 
// this thread can modify only 'job_shared'
void job_2() 
{
    mutex.lock();
    std::this_thread::sleep_for(5 * interval);
    ++job_shared;
    mutex.unlock();
}
 
int main() 
{
    std::thread thread_1(job_1);
    std::thread thread_2(job_2);
 
    thread_1.join();
    thread_2.join();
}
```

## std::recursive_mutex

不同与是std::mutex, std::recursive_mutex允许在同一线程多次给同一个锁进行加锁操作。

```c++
#include <iostream>
#include <thread>
#include <mutex>
 
class X {
    std::recursive_mutex m;
    std::string shared;
  public:
    void fun1() {
      std::lock_guard<std::recursive_mutex> lk(m);
      shared = "fun1";
      std::cout << "in fun1, shared variable is now " << shared << '\n';
    }
    void fun2() {
      std::lock_guard<std::recursive_mutex> lk(m);
      shared = "fun2";
      std::cout << "in fun2, shared variable is now " << shared << '\n';
      fun1(); // recursive lock becomes useful here
      std::cout << "back in fun2, shared variable is " << shared << '\n';
    };
};
 
int main() 
{
    X x;
    std::thread t1(&X::fun1, &x);
    std::thread t2(&X::fun2, &x);
    t1.join();
    t2.join();
}
```

## std::shared_mutex

```c++
#include <iostream>
#include <mutex>
#include <shared_mutex>
#include <thread>
 
class ThreadSafeCounter {
public:
    ThreadSafeCounter() = default;

    // Multiple threads/readers can read the counter's value at the same time.
    unsigned int get() const {
        std::shared_lock<std::shared_mutex> lock(mutex_);
        return value_;
    }

    // Only one thread/writer can increment/write the counter's value.
    void increment() {
        std::unique_lock<std::shared_mutex> lock(mutex_);
        ++value_;
    }

    // Only one thread/writer can reset/write the counter's value.
    void reset() {
        std::unique_lock<std::shared_mutex> lock(mutex_);
        value_ = 0;
    }

private:
    mutable std::shared_mutex mutex_;
    unsigned int value_ = 0;
};

int main() {
    ThreadSafeCounter counter;

    auto increment_and_print = [&counter]() {
        for (int i = 0; i < 3; i++) {
            counter.increment();
            std::cout << std::this_thread::get_id() << ' ' << counter.get() << '\n';

            // Note: Writing to std::cout actually needs to be synchronized as well
            // by another std::mutex. This has been omitted to keep the example small.
        }
    };

    std::thread thread1(increment_and_print);
    std::thread thread2(increment_and_print);

    thread1.join();
    thread2.join();
}
```

`std::shared_mutex` 允许多个使用 `std::shared_lock` 进行加锁的线程同时获得共享锁；当用`std::unique_lock` 进行加锁时，只有一个线程可以获得该共享锁，其他尝试加锁的线程将被阻塞。

## 指定锁定策略

有三种锁定策略：

| 类型 | 作用 |
|---------|-----------|
| defer_lock_t | 不获取互斥锁的所有权 |
| try_to_lock_t | 尝试在不阻塞的情况下获取互斥锁的所有权 |
| adopt_lock_t | 假设调用线程已经拥有互斥锁的所有权 |

三种锁定策略搭配`std::lock`锁定给定的 Lockable 对象 lock1、lock2、...、lockn 以避免死锁。

```c++
#include <mutex>
#include <thread>
#include <iostream>

struct bank_account {
    explicit bank_account(int balance) : balance{ balance } {}
    int balance;
    std::mutex m;
};
```

### defer_lock

```c++
void transfer(bank_account& from, bank_account& to, int amount)
{
    if (&from == &to) return; // avoid deadlock in case of self transfer

    std::unique_lock<std::mutex> lock1{from.m, std::defer_lock};
    std::unique_lock<std::mutex> lock2{to.m, std::defer_lock};
    std::lock(lock1, lock2);

    from.balance -= amount;
    to.balance += amount;
}
```

### adopt_lock

```c++
 
void transfer(bank_account &from, bank_account &to, int amount)
{
    if(&from == &to) return; // avoid deadlock in case of self transfer
 
    // lock both mutexes without deadlock
    std::lock(from.m, to.m);
    // make sure both already-locked mutexes are unlocked at the end of scope
    std::lock_guard<std::mutex> lock1{from.m, std::adopt_lock};
    std::lock_guard<std::mutex> lock2{to.m, std::adopt_lock};
 
    from.balance -= amount;
    to.balance += amount;
}
```

```c++
int main()
{
    bank_account my_account{ 100 };
    bank_account your_account{ 50 };

    std::thread t1{ transfer, std::ref(my_account), std::ref(your_account), 10 };
    std::thread t2{ transfer, std::ref(your_account), std::ref(my_account), 5 };

    t1.join();
    t2.join();

    std::cout << "my_account.balance = " << my_account.balance << "\n"
        "your_account.balance = " << your_account.balance << '\n';
}
```

## std::scoped_lock(C++17)

std::scoped_lock是一个互斥锁包装器，它提供了一种方便的 RAII 样式机制，用于在作用域块的持续时间内拥有零个或多个互斥锁

```c++
#include <chrono>
#include <functional>
#include <iostream>
#include <mutex>
#include <string>
#include <thread>
#include <vector>
using namespace std::chrono_literals;
 
struct Employee
{
    std::vector<std::string> lunch_partners;
    std::string id;
    std::mutex m;
    Employee(std::string id) : id(id) {}
    std::string partners() const
    {
        std::string ret = "Employee " + id + " has lunch partners: ";
        for (const auto& partner : lunch_partners)
            ret += partner + " ";
        return ret;
    }
};
 
void send_mail(Employee &, Employee &)
{
    // simulate a time-consuming messaging operation
    std::this_thread::sleep_for(1s);
}
 
void assign_lunch_partner(Employee &e1, Employee &e2)
{
    static std::mutex io_mutex;
    {
        std::lock_guard<std::mutex> lk(io_mutex);
        std::cout << e1.id << " and " << e2.id << " are waiting for locks" << std::endl;
    }
 
    {
        // use std::scoped_lock to acquire two locks without worrying about
        // other calls to assign_lunch_partner deadlocking us
        // and it also provides a convenient RAII-style mechanism
 
        std::scoped_lock lock(e1.m, e2.m);
 
        // Equivalent code 1 (using std::lock and std::lock_guard)
        // std::lock(e1.m, e2.m);
        // std::lock_guard<std::mutex> lk1(e1.m, std::adopt_lock);
        // std::lock_guard<std::mutex> lk2(e2.m, std::adopt_lock);
 
        // Equivalent code 2 (if unique_locks are needed, e.g. for condition variables)
        // std::unique_lock<std::mutex> lk1(e1.m, std::defer_lock);
        // std::unique_lock<std::mutex> lk2(e2.m, std::defer_lock);
        // std::lock(lk1, lk2);
        {
            std::lock_guard<std::mutex> lk(io_mutex);
            std::cout << e1.id << " and " << e2.id << " got locks" << std::endl;
        }
        e1.lunch_partners.push_back(e2.id);
        e2.lunch_partners.push_back(e1.id);
    }
 
    send_mail(e1, e2);
    send_mail(e2, e1);
}
 
int main()
{
    Employee alice("Alice"), bob("Bob"), christina("Christina"), dave("Dave");
 
    // assign in parallel threads because mailing users about lunch assignments
    // takes a long time
    std::vector<std::thread> threads;
    threads.emplace_back(assign_lunch_partner, std::ref(alice), std::ref(bob));
    threads.emplace_back(assign_lunch_partner, std::ref(christina), std::ref(bob));
    threads.emplace_back(assign_lunch_partner, std::ref(christina), std::ref(alice));
    threads.emplace_back(assign_lunch_partner, std::ref(dave), std::ref(bob));
 
    for (auto &thread : threads)
        thread.join();
    std::cout << alice.partners() << '\n'  << bob.partners() << '\n'
              << christina.partners() << '\n' << dave.partners() << '\n';
}
```
