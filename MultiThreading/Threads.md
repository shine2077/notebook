# Threads

## std::thread

每个线程都需要一个起始函数（initial function）作为线程对象构造函数的参数。线程在构造相关线程对象后立即从起始函数函数开始执行。

```c++
#include <iostream>
#include <thread>

void hello()
{
    std::cout<<"Hello Concurrent World\n";
} 

int main()
{
    std::thread t(hello);  
    t.join();
}
```

该代码生成了一个新线程来打印 *hello语句*

`std::thread` 有三个操作函数

|操作|        |
|---------------|-----------------|
|join|等待线程完成其执行|
|detach|容许线程从线程句柄独立开来执行|
|swap|交换二个 thread 对象|

### std::thread::join

阻塞当前线程直至 *this 所标识的线程结束其执行

```c++
#include <iostream>
#include <thread>
#include <chrono>

void foo()
{
    // 模拟耗费大量资源的操作
    std::this_thread::sleep_for(std::chrono::seconds(2));
}

void bar()
{
    // 模拟耗费大量资源的操作
    std::this_thread::sleep_for(std::chrono::seconds(10));
}

int main()
{
    std::cout << "starting first helper...\n";
    std::thread helper1(foo);

    std::cout << "starting second helper...\n";
    std::thread helper2(bar);

    std::cout << "waiting for first helper to finish..." << std::endl;
    helper1.join();
    std::cout << "first helper finished" << std::endl;

    std::cout << "waiting for second helper to finish..." << std::endl;
    helper2.join();
    std::cout << "second helper finished\n";
}
```

输出

```c++
starting first helper...
starting second helper...
waiting for first helper to finish...
//2s后
first helper finished
waiting for second helper to finish...
//10后
second helper finished
```

如果线程`foo` 的sleep时间大于`bar`:

```c++
void foo()
{
    // 模拟耗费大量资源的操作
    std::this_thread::sleep_for(std::chrono::seconds(10));
}

void bar()
{
    // 模拟耗费大量资源的操作
    std::this_thread::sleep_for(std::chrono::seconds(5));
}
```

输出

```c++
starting first helper...
starting second helper...
waiting for first helper to finish...
//阻塞10s后
first helper finished
//second helper已经结束，会立即返回
waiting for second helper to finish...
second helper finished
```

### std::thread::detach

从 thread 对象分离执行线程，允许执行线程独立地运行。一旦该线程退出，则释放任何分配的资源

```c++
#include <iostream>
#include <chrono>
#include <thread>

void independentThread()
{
    std::cout << "Starting concurrent thread.\n";
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Exiting concurrent thread.\n";
}

void threadCaller()
{
    std::cout << "Starting thread caller.\n";
    std::thread t(independentThread);
    t.detach();
    std::this_thread::sleep_for(std::chrono::seconds(1));
    std::cout << "Exiting thread caller.\n";
}

int main()
{
    threadCaller();
    std::this_thread::sleep_for(std::chrono::seconds(2));
}
```

输出

```c++
Starting thread caller.
Starting concurrent thread.
Exiting thread caller.
Exiting concurrent thread.
```

如果我们注释`std::this_thread::sleep_for(std::chrono::seconds(1))`

```c++
int main()
{
    threadCaller();
    //std::this_thread::sleep_for(std::chrono::seconds(1));
}
```

进程会在`threadCaller()`后立即结束，使得无法继续运行分离出来的`independentThread()`线程。

```c++
Starting thread caller.
Starting concurrent thread.
Exiting thread caller.
```

### std::thread::swap

```c++
#include <iostream>
#include <thread>
#include <chrono>
 
void foo()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
void bar()
{
    std::this_thread::sleep_for(std::chrono::seconds(1));
}
 
int main()
{
    std::thread t1(foo);
    std::thread t2(bar);
 
    std::cout << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    std::swap(t1, t2);
 
    std::cout << "after std::swap(t1, t2):" << '\n'
              << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    t1.swap(t2);
 
    std::cout << "after t1.swap(t2):" << '\n'
              << "thread 1 id: " << t1.get_id() << '\n'
              << "thread 2 id: " << t2.get_id() << '\n';
 
    t1.join();
    t2.join();
}
```

输出

```c++
thread 1 id: 14544
thread 2 id: 8528
after std::swap(t1, t2):
thread 1 id: 8528
thread 2 id: 14544
after t1.swap(t2):
thread 1 id: 14544
thread 2 id: 8528
```

### std::thread::joinable

检查 std::thread 对象是否标识活动的执行线程。具体来说，如果 `get_id() != std::thread::id()` 则返回 true。所以默认构造的线程是`not joinable`。

已完成执行代码但尚未`join`的线程仍被视为活动执行线程，因此是`joinable`

```c++
#include <iostream>
#include <thread>
#include <chrono>
using namespace std::chrono_literals;
 
void foo()
{
    std::this_thread::sleep_for(500ms);
}
 
int main()
{
    std::cout << std::boolalpha;
 
    std::thread t;
    std::cout << "before starting, joinable: " << t.joinable() << '\n';
 
    t = std::thread{foo};
    std::cout << "after starting, joinable: " << t.joinable() << '\n';
 
    t.join();
    std::cout << "after joining, joinable: " << t.joinable() << '\n';
 
    t = std::thread{foo};
    std::cout << "before detaching, joinable: " << t.joinable() << '\n';
    t.detach();
    std::cout << "after detaching, joinable: " << t.joinable() << '\n';
    std::this_thread::sleep_for(1500ms);
}
```

输出

```c++
before starting, joinable: false
after starting, joinable: true
after joining, joinable: false
before detaching, joinable: true
after detaching, joinable: false
```

### std::thread::hardware_concurrency

返回程序在各次运行中可真正并发的线程数量。例如，在多核系统上，该值可能就是CPU的核芯数量。这仅仅是一个指标，若信息无法获取，该函数则可能返回0

## std::this_thread::yield

请求重新安排线程的执行，从而允许其他线程运行

```c++
#include <iostream>
#include <chrono>
#include <thread>
 
// "busy sleep" while suggesting that other threads run 
// for a small amount of time
void little_sleep(std::chrono::microseconds us)
{
    auto start = std::chrono::high_resolution_clock::now();
    auto end = start + us;
    do {
        std::this_thread::yield();
    } while (std::chrono::high_resolution_clock::now() < end);
}
 
int main()
{
    auto start = std::chrono::high_resolution_clock::now();
 
    little_sleep(std::chrono::microseconds(100));
 
    auto elapsed = std::chrono::high_resolution_clock::now() - start;
    std::cout << "waited for "
              << std::chrono::duration_cast<std::chrono::microseconds>(elapsed).count()
              << " microseconds\n";
}
```

## std::this_thread::get_id

Returns the id of the current thread

## std::this_thread::sleep_for

Blocks the execution of the current thread for at least the specified sleep_duration

```c++
std::this_thread::sleep_for(std::chrono::seconds(1));
``

## std::this_thread::sleep_until

Blocks the execution of the current thread until specified sleep_time has been reached.

```c++
#include <iostream>
#include <chrono>
#include <thread>
 
auto now() { return std::chrono::steady_clock::now(); }
 
auto awake_time() {
    using std::chrono::operator""ms;
    return now() + 2000ms;
}
 
int main()
{
    std::cout << "Hello, waiter...\n" << std::flush;
    const auto start {now()};
    std::this_thread::sleep_until(awake_time());
    std::chrono::duration<double, std::milli> elapsed {now() - start};
    std::cout << "Waited " << elapsed.count() << " ms\n";
}
```
