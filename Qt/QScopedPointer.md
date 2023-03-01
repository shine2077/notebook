# Description
QScopedPointer guarantees that the object pointed to will get deleted when the current scope disappears.
## example
```c++
void myFunction(bool useSubClass)
{
    MyClass *p = useSubClass ? new MyClass() : new MySubClass();
    QIODevice *device = handsOverOwnership();

    if (m_value > 3) {
        delete p;
        delete device;
        return;
    }

    try {
        process(device);
    }
    catch (...) {
        delete p;
        delete device;
        throw;
    }

    delete p;
    delete device;
}
```
With QScopedPointer, the code can be simplified to:
```c++
void myFunction(bool useSubClass)
{
    // assuming that MyClass has a virtual destructor
    QScopedPointer<MyClass> p(useSubClass ? new MyClass() : new MySubClass());
    QScopedPointer<QIODevice> device(handsOverOwnership());

    if (m_value > 3)
        return;

    process(device);
}
```

QScopedPointer's second template parameter can be used for custom cleanup handlers.
## example
```c++
// this QScopedPointer deletes its data using the delete[] operator:
QScopedPointer<int, QScopedPointerArrayDeleter<int> > arrayPointer(new int[42]);

// this QScopedPointer frees its data using free():
QScopedPointer<int, QScopedPointerPodDeleter> podPointer(reinterpret_cast<int *>(malloc(42)));

// this struct calls "myCustomDeallocator" to delete the pointer
struct ScopedPointerCustomDeleter
{
    static inline void cleanup(MyCustomClass *pointer)
    {
        myCustomDeallocator(pointer);
    }
};

// QScopedPointer using a custom deleter:
QScopedPointer<MyCustomClass, ScopedPointerCustomDeleter> customPointer(new MyCustomClass);
```
Classes that are forward declared can be used within QScopedPointer, as long as the destructor of the forward declared class is available whenever a QScopedPointer needs to clean up.
Concretely, this means that all classes containing a QScopedPointer that points to a forward declared class must have non-inline constructors, destructors and assignment operators:
## example
```c++
class MyPrivateClass; // forward declare MyPrivateClass

class MyClass
{
private:
    QScopedPointer<MyPrivateClass> privatePtr; // QScopedPointer to forward declared class

public:
    MyClass(); // OK
    ~MyClass() {};

private:
    Q_DISABLE_COPY(MyClass) // OK - copy constructor and assignment operators
                             // are now disabled, so the compiler won't implicitely
                             // generate them.
};
```
Otherwise, the compiler outputs a warning about not being able to destruct MyPrivateClass.