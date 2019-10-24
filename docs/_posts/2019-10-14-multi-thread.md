---
layout: post
title:  "C++ Multithread"
date:   2019-10-14 12:00:00 +0800
toc: true
categories: [cpp]
tags: [note]
description: no
---

- 目录
{:toc #markdown-toc}

## thread_local

`thread_local`数据专属于线程，类似于静态数据，在第一次使用时创建，其生命周期将绑定到线程的生命周期。因为是专属于线程，可以在`不加锁时修改`。

~~~cpp
#include <iostream>
#include <string>
#include <thread>
#include <mutex>
 
thread_local unsigned int rage = 1; 
std::mutex cout_mutex;
 
void increase_rage(const std::string& thread_name)
{
    ++rage; // modifying outside a lock is okay; this is a thread-local variable
    std::lock_guard<std::mutex> lock(cout_mutex);
    std::cout << "Rage counter for " << thread_name << ": " << rage << '\n';
}
 
int main()
{
    std::thread a(increase_rage, "a"), b(increase_rage, "b");
    {
        std::lock_guard<std::mutex> lock(cout_mutex);
        std::cout << "Rage counter for main: " << rage << '\n';
    }
    a.join();
    b.join();
}
~~~

## condition_variable
[详情](https://www.cnblogs.com/wangshaowei/p/9593201.html)

- `std::condition_variable::notify_all()`：唤醒所有线程。如果当前没有等待线程，则该函数什么也不做。
- `std::condition_variable::wait()`：当前线程获得锁lck之后，调用`wait()`会自动释放锁lck，使得其他被锁阻塞的线程继续执行。一旦获得通知notify，`wait()`会自动获得锁。

~~~cpp
#include <iostream>
#include <thread>
#include <mutex>
#include <condition_variable>

std::mutex mtx;             // 全局互斥锁.
std::condition_variable cv; // 全局条件变量.
bool ready = false;         // 全局标志位.

void do_print_id(int id)
{
    std::unique_lock <std::mutex> lck(mtx);
    while (!ready)  // 如果标志位不为 true, 则等待...
        cv.wait(lck);   // 当前线程被阻塞, 当全局标志位变为 true 之后,
                        // 线程被唤醒, 继续往下执行打印线程编号id.
    std::cout << "thread " << id << '\n';
}

void go()
{
    std::unique_lock <std::mutex> lck(mtx);
    ready = true;       // 设置全局标志位为 true.
    cv.notify_all();    // 唤醒所有线程.
}

int main()
{
    std::thread threads[10];
    for (int i = 0; i < 10; ++i)
        threads[i] = std::thread(do_print_id, i);
    std::cout << "10 threads ready to race...\n";
    go();

    for (auto & th:threads)
        th.join();
    return 0;
}
~~~
