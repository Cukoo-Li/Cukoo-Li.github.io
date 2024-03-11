+++
title = '手写shared_ptr'
date = 2024-03-11T23:07:08+08:00
+++

手写`shared_ptr`是面试的常考点，提前准备一下吧。

## shared_ptr的关键特性

首先明确`shared_ptr`的关键特性：

- 允许多个`shared_ptr`指向同一个对象
- 每个`shared_ptr`都有一个关联的引用计数，用于记录总共有多少个`shared_ptr`指向相同的对象
- 一旦一个`shared_ptr`所关联的引用计数变为0，它就会自动释放自己所管理的对象

## 代码思路

显然，最关键的就是要实现引用计数这个机制，大致思路如下：

- 在堆中创建一个整型对象作为引用计数
- 在拷贝或赋值时修改引用计数
- 在析构函数中递减引用计数，并考察它是否变为0，如果变为0，则释放所指向的对象以及引用计数对象

## 代码实现

```cpp
#include <iostream>

template <typename T>
class TinySharedPtr {
   public:
    // 构造函数
    explicit TinySharedPtr(T* ptr = nullptr) : ptr_(ptr) {
        if (ptr_)
            count_ = new int(1);
    }
    // 拷贝构造函数
    TinySharedPtr(const TinySharedPtr& other)
        : ptr_(other.ptr_), count_(other.count_) {
        if (count_)
            ++*count_;
    }
    // 拷贝赋值运算符
    TinySharedPtr& operator=(TinySharedPtr other) {
        std::swap(ptr_, other.ptr_);
        std::swap(count_, other.count_);
        return *this;
    }
    // 析构函数
    ~TinySharedPtr() {
        if (count_ && --(*count_) == 0) {
            delete ptr_;
            delete count_;
        }
    }

    T* get() { return ptr_; }
    T* operator->() { return ptr_; }
    T& operator*() { return *ptr_; }
    int use_count() { return count_ ? *count_ : 0; }

   private:
    T* ptr_ = nullptr;
    int* count_ = nullptr;
};

struct Cukoo {
    Cukoo() { puts("Cukoo()"); }
    ~Cukoo() { puts("~Cukoo()"); }
    int age = 24;
};

int main() {
    TinySharedPtr<Cukoo> sp0;
    std::cout << "sp0: " << sp0.use_count() << '\n';
    {
        TinySharedPtr<Cukoo> sp1(new Cukoo());
        std::cout << "age: " << sp1->age << '\n';
        std::cout << "sp1: " << sp1.use_count() << '\n';
        TinySharedPtr<Cukoo> sp2(sp1);
        std::cout << "sp2: " << sp2.use_count() << '\n';
        sp0 = sp2;
        std::cout << "sp0: " << sp2.use_count() << '\n';
    }
    std::cout << "sp0: " << sp0.use_count() << '\n';
    return 0;
}

// sp0: 0
// Cukoo()
// age: 24
// sp1: 1
// sp2: 2
// sp0: 3
// sp0: 1
// ~Cukoo()
```