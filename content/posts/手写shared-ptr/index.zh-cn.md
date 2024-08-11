---
title: '手写 shared_ptr'
date: 2024-03-11T23:07:08+08:00
---

手写 `shared_ptr` 是面试的常考点，提前准备一下吧。

## shared_ptr 的关键特性

首先明确 `shared_ptr` 的关键特性：

- 允许多个 `shared_ptr` 指向同一个对象
- 每个 `shared_ptr` 都有一个关联的引用计数，用于记录总共有多少个 `shared_ptr` 指向相同的对象
- 一旦一个 `shared_ptr` 所关联的引用计数变为 0，它就会自动释放自己所管理的对象

## 代码思路

显然，最关键的就是要实现引用计数这个机制，大致思路如下：

- 在堆中创建一个整型对象作为引用计数
- 在拷贝时递增引用计数
- 在析构函数中递减引用计数，并考察它是否变为 0，如果变为 0，则释放所指向的对象以及引用计数对象

## 代码实现

```cpp
#pragma once

template <typename T>
class SharedPtr {
   public:
    // 构造函数
    explicit SharedPtr(T* ptr = nullptr) : ptr_(ptr) {
        if (ptr_)
            count_ = new int(1);
    }

    // 析构函数
    ~SharedPtr() {
        if (count_ && --(*count_) == 0) {
            delete ptr_;
            delete count_;
        }
    }

    // 拷贝构造函数
    SharedPtr(const SharedPtr& that) {
        ptr_ = that.ptr_; 
        count_ = that.count_;
        if (count_)
            ++(*count_);
    }

    // 移动构造函数
    SharedPtr(SharedPtr&& that) {
        ptr_ = that.ptr_;
        count_ = that.count_;
        that.ptr_ = nullptr;
        that.count_ = nullptr;
    }

    // 拷贝赋值运算符
    SharedPtr& operator=(const SharedPtr& that) {
        if (&that == this)
            return *this;
        if (count_ && --(*count_) == 0) {
            delete ptr_;
            delete count_;
        }
        ptr_ = that.ptr_;
        count_ = that.count_;
        if (count_)
            ++(*count_);
        return *this;
    }

    // 移动赋值运算符
    SharedPtr& operator=(SharedPtr&& that) {
        if (&that == this)
            return *this;
        if (count_ && --(*count_) == 0) {
            delete ptr_;
            delete count_;
        }
        ptr_ = that.ptr_;
        count_ = that.count_;
        that.ptr_ = nullptr;
        that.count_ = nullptr;
        return *this;
    }

    T* get() { return ptr_; }
    T* operator->() { return ptr_; }
    T& operator*() { return *ptr_; }
    int use_count() { return count_ ? *count_ : 0; }

   private:
    T* ptr_{};
    int* count_{};
};

struct X {
    X() { puts(__PRETTY_FUNCTION__); }
    ~X() { puts(__PRETTY_FUNCTION__); }
};

int main() {
    SharedPtr<X> sp1;
    std::cout << std::format("sp1.use_count = {}", sp1.use_count()) << '\n';
    {
        SharedPtr<X> sp2(new X());
        std::cout << std::format("sp2.use_count = {}", sp2.use_count()) << '\n';
        SharedPtr<X> sp3(std::move(sp2));
        std::cout << std::format("sp2.use_count = {}", sp2.use_count()) << '\n';
        std::cout << std::format("sp3.use_count = {}", sp3.use_count()) << '\n';
        sp1 = sp3;
        std::cout << std::format("sp1.use_count = {}", sp1.use_count()) << '\n';
    }
    std::cout << std::format("sp1.use_count = {}", sp1.use_count()) << '\n';
    return 0;
}

// sp1.use_count = 0
// X::X()
// sp2.use_count = 1
// sp2.use_count = 0
// sp3.use_count = 1
// sp1.use_count = 2
// sp1.use_count = 1
// X::~X()
```