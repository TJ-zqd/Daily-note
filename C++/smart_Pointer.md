# 智能指针



## 1、智能指针的种类

### 在C++中，有三种常用的智能指针：std::unique_ptr、std::shared_ptr和std::weak_ptr。

- std::unique_ptr：std::unique_ptr是一种独占所有权的智能指针。它通过使用独占所有权的方式来管理资源，只能有一个std::unique_ptr指向同一个对象或数组。当std::unique_ptr超出作用域或被显式释放时，它会自动删除所管理的对象或数组。它通常用于表示独占的资源所有权，如动态分配的单个对象或数组。
- std::shared_ptr：std::shared_ptr是一种共享所有权的智能指针。它可以有多个std::shared_ptr指向同一个对象，通过引用计数来管理资源的生命周期。只有当最后一个std::shared_ptr超出作用域或被显式释放时，资源才会被删除。std::shared_ptr允许多个指针共享对同一资源的访问，通常用于表示共享的资源所有权。
- std::weak_ptr：std::weak_ptr是一种弱引用的智能指针。它可以指向由std::shared_ptr管理的对象，但不会增加引用计数。std::weak_ptr主要用于解决std::shared_ptr的循环引用问题，通过std::weak_ptr.lock()方法可以获取一个有效的std::shared_ptr来访问被管理的对象。

## 2、在哪些场景下会应用智能指针？

​	我自己是在在动态内存管理中，使用智能指针可以避免手动管理内存的麻烦和出错风险。

##3、shared_ptr的作用是什么？

​	在传统的 C++ 编程中，使用 `new` 操作符分配的内存需要手动使用 `delete` 操作符释放，若忘记释放或者在异常情况下无法执行 `delete` 操作，就会造成内存泄漏。

​	std::shared_ptr` 可以自动处理内存的释放，当不再有 `std::shared_ptr` 指向该对象时，对象的内存会被自动释放。

​	`std::shared_ptr` 支持多个 `std::shared_ptr` 实例共享同一个对象的所有权。它通过引用计数机制来实现这一点，每个 `std::shared_ptr` 都会维护一个引用计数，记录有多少个 `std::shared_ptr` 共享同一个对象。当引用计数变为 0 时，对象的内存会被释放。

```c++
#include <iostream>
#include <memory>

class MyClass {
public:
    MyClass() { std::cout << "MyClass constructor" << std::endl; }
    ~MyClass() { std::cout << "MyClass destructor" << std::endl; }
};

int main() {
    std::shared_ptr<MyClass> ptr1 = std::make_shared<MyClass>();
    std::shared_ptr<MyClass> ptr2 = ptr1; // 共享所有权
    std::cout << "Use count: " << ptr1.use_count() << std::endl; // 输出 2
    ptr2.reset(); // 释放 ptr2 的所有权
    std::cout << "Use count: " << ptr1.use_count() << std::endl; // 输出 1
    // 当 ptr1 离开作用域时，MyClass 对象的内存会被释放
    return 0;
}

```

​		在这个例子中，`ptr1` 和 `ptr2` 共享同一个 `MyClass` 对象的所有权，引用计数变为 2。当调用 `ptr2.reset()` 时，`ptr2` 释放了对对象的所有权，引用计数减为 1。当 `ptr1` 离开作用域时，引用计数变为 0，对象的内存被释放。

##4、weak_ptr的作用是什么？如何和shared_ptr结合使用？

​	`	std::weak_ptr` 主要用于辅助 `std::shared_ptr` 进行内存管理，解决 `std::shared_ptr` 可能存在的循环引用问题，同时还可以用于观察 `std::shared_ptr` 所管理对象的生命周期。

​	当两个或多个 `std::shared_ptr` 相互引用形成循环时，会导致引用计数永远不会降为 0，从而造成内存泄漏。`std::weak_ptr` 不会增加所指向对象的引用计数，因此可以打破这种循环引用。

```c++
#include <iostream>
#include <memory>

class B;

class A {
public:
    std::shared_ptr<B> b_ptr;
    ~A() { std::cout << "A destructor" << std::endl; }
};

class B {
public:
    std::weak_ptr<A> a_ptr;  // 使用 std::weak_ptr 打破循环引用
    ~B() { std::cout << "B destructor" << std::endl; }
};

int main() {
    std::shared_ptr<A> a = std::make_shared<A>();
    std::shared_ptr<B> b = std::make_shared<B>();
    a->b_ptr = b;
    b->a_ptr = a;
    return 0;
}
```

​	在上述代码中，如果 `B` 类中的 `a_ptr` 也使用 `std::shared_ptr`，就会形成循环引用，导致 `A` 和 `B` 对象的内存无法释放。使用 `std::weak_ptr` 后，`b->a_ptr` 不会增加 `A` 对象的引用计数，当 `main` 函数结束时，`a` 和 `b` 的引用计数降为 0，`A` 和 `B` 对象的内存会被正确释放。

​	另外，`std::weak_ptr` 可以用于观察 `std::shared_ptr` 所管理对象的生命周期。通过 `std::weak_ptr` 的 `expired()` 方法可以检查所指向的对象是否已经被释放。

```c++
#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> shared = std::make_shared<int>(42);
    std::weak_ptr<int> weak = shared;

    if (!weak.expired()) {
        std::cout << "Object is still alive." << std::endl;
    }

    shared.reset();
    if (weak.expired()) {
        std::cout << "Object has been destroyed." << std::endl;
    }

    return 0;
}
```

​	在这个例子中，`weak` 观察 `shared` 所管理的对象。当 `shared` 释放对象后，`weak.expired()` 返回 `true`，表示对象已经被销毁。

## 5、智能指针会造成内存泄漏吗？

​	会的，比如循环引用场景，在多个shared_ptr形成循环引用，资源将无法释放。

​	std::shared_ptr：多个shared_ptr对象可以共享同一个资源的所有权，它们会维护一个引用计数。只有当引用计数为零时，才会释放资源。这种智能指针的内存管理是自动的，因此可以帮助我们避免显式地释放内存或出现内存泄漏的情况。

```c++
std::shared_ptr<int> sharedPtr1 = std::make_shared<int>(10);
std::shared_ptr<int> sharedPtr2 = sharedPtr1;
sharedPtr1.reset();  // 不会导致内存泄漏，资源仍由sharedPtr2管理
sharedPtr2.reset();  // 资源被释放
```

​	在使用std::shared_ptr时，需要注意循环引用的情况。如果两个或多个shared_ptr对象相互引用，形成循环依赖，它们的引用计数永远不会达到零，资源将无法释放，从而导致内存泄漏。

​	针对这个问题，可以使用 weak_ptr 弱引用来解决这个问题。weak_ptr是用来监视shared_ptr的生命周期，它不管理shared_ptr内部的指针，它的拷贝的析构都不会影响引用计数，纯粹是作为一个旁观者监视shared_ptr中管理的资源是否存在，可以用来返回this指针和解决循环引用问题。

## 6、一个 unique_ptr 怎么赋值给另一个 unique_ptr 对象？

​	借助 std::move() 可以实现将一个 unique_ptr 对象赋值给另一个 unique_ptr 对象，其目的是实现所有权的转移。 也可以让当前unique_ptr调用release释放对ptr控制权，然后在另一个unique_ptr获取控制权。

```c++
// A 作为一个类 
std::unique_ptr<A> ptr1(new A());
std::unique_ptr<A> ptr2 = std::move(ptr1);
std::unique_ptr<A> ptr3(ptr2.release());
```

##7、使用智能指针会出现什么问题？怎么解决？

​	智能指针可能出现的问题：循环引用

​	在如下例子中定义了两个类 Parent、Child，在两个类中分别定义另一个类的对象的共享指针，由于在程序结束后，两个指针相互指向对方的内存空间，导致内存无法释放。



```c++
#include <iostream>
#include <memory>

using namespace std;

class Child;
class Parent;

class Parent {
private:
    shared_ptr<Child> ChildPtr;
public:
    void setChild(shared_ptr<Child> child) {
        this->ChildPtr = child;
    }

    void doSomething() {
        if (this->ChildPtr.use_count()) {

        }
    }

    ~Parent() {
    }
};

class Child {
private:
    shared_ptr<Parent> ParentPtr;
public:
    void setPartent(shared_ptr<Parent> parent) {
        this->ParentPtr = parent;
    }
    void doSomething() {
        if (this->ParentPtr.use_count()) {

        }
    }
    ~Child() {
    }
};

int main() {
    weak_ptr<Parent> wpp;
    weak_ptr<Child> wpc;
    {
        shared_ptr<Parent> p(new Parent);
        shared_ptr<Child> c(new Child);
        p->setChild(c);
        c->setPartent(p);
        wpp = p;
        wpc = c;
        cout << p.use_count() << endl; // 2
        cout << c.use_count() << endl; // 2
    }
    cout << wpp.use_count() << endl;  // 1
    cout << wpc.use_count() << endl;  // 1
    return 0;
}
```

​	循环引用的解决方法： weak_ptr

​	循环引用：该被调用的析构函数没有被调用，从而出现了内存泄漏。

- weak_ptr 对被 shared_ptr 管理的对象存在非拥有性（弱）引用，在访问所引用的对象前必须先转化为 shared_ptr；
- weak_ptr 用来打断 shared_ptr 所管理对象的循环引用问题，若这种环被孤立（没有指向环中的外部共享指针），shared_ptr 引用计数无法抵达 0，内存被泄露；令环中的指针之一为弱指针可以避免该情况；
- weak_ptr 用来表达临时所有权的概念，当某个对象只有存在时才需要被访问，而且随时可能被他人删除，可以用 weak_ptr 跟踪该对象；需要获得所有权时将其转化为 shared_ptr，此时如果原来的 shared_ptr 被销毁，则该对象的生命期被延长至这个临时的 shared_ptr 同样被销毁。

```c++
#include <iostream>
#include <memory>

using namespace std;

class Child;
class Parent;

class Parent {
private:
    //shared_ptr<Child> ChildPtr;
    weak_ptr<Child> ChildPtr;
public:
    void setChild(shared_ptr<Child> child) {
        this->ChildPtr = child;
    }

    void doSomething() {
        //new shared_ptr
        if (this->ChildPtr.lock()) {

        }
    }

    ~Parent() {
    }
};

class Child {
private:
    shared_ptr<Parent> ParentPtr;
public:
    void setPartent(shared_ptr<Parent> parent) {
        this->ParentPtr = parent;
    }
    void doSomething() {
        if (this->ParentPtr.use_count()) {

        }
    }
    ~Child() {
    }
};

int main() {
    weak_ptr<Parent> wpp;
    weak_ptr<Child> wpc;
    {
        shared_ptr<Parent> p(new Parent);
        shared_ptr<Child> c(new Child);
        p->setChild(c);
        c->setPartent(p);
        wpp = p;
        wpc = c;
        cout << p.use_count() << endl; // 2
        cout << c.use_count() << endl; // 1
    }
    cout << wpp.use_count() << endl;  // 0
    cout << wpc.use_count() << endl;  // 0
    return 0;
}
```

