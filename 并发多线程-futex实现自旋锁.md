- futex（fast userspace mutex）是一种实现互斥锁（mutex）的同步原语，旨在提高用户空间多线程应用程序的性能。
- 传统的互斥锁（mutex）实现通常需要在内核态和用户态之间进行频繁的上下文切换，这种上下文切换的开销可能会导致性能瓶颈。为了避免这种开销，futex 将互斥锁的实现分为两个部分：一个用户空间部分和一个内核空间部分。只有在互斥锁的状态发生变化时，才需要进行内核态和用户态之间的切换。
- futex 还支持等待队列，当一个线程需要等待一个互斥锁时，它可以将自己添加到等待队列中。当其他线程释放锁时，它们可以唤醒等待队列中的线程。
- futex 是一种非常高效的同步原语，通常用于高并发的多线程应用程序。在 Linux 内核中，futex 被广泛使用，并且成为了 POSIX 线程标准的一部分。

- **传统的互斥锁实现通常使用了内核态提供的系统调用，例如pthread_mutex_lock()，这些系统调用需要在用户态和内核态之间进行频繁的上下文切换。具体来说，当一个线程调用pthread_mutex_lock()获取锁时，如果该锁已被其他线程占用，那么该线程会进入等待状态并切换到内核态，等待内核将其唤醒。当该锁被释放时，内核会重新调度等待该锁的线程之一，并将其唤醒，使其重新进入用户态并继续执行。这样的频繁上下文切换会带来较大的系统开销。而futex则是一种用户态的锁，避免了频繁的上下文切换，从而提高了锁的性能。**

```cpp
#include <iostream>
#include <unistd.h>
#include <sys/syscall.h>
#include <linux/futex.h>
#include <atomic>

class SpinLock {
public:
    SpinLock() : flag(false) {}

    void lock() {
        while (true) {
            bool expected = false;
            if (flag.compare_exchange_strong(expected, true)) {
                return;
            }
            syscall(SYS_futex, &flag, FUTEX_WAIT, 1, nullptr, nullptr, 0);
        }
    }

    void unlock() {
        flag.store(false);
        syscall(SYS_futex, &flag, FUTEX_WAKE, 1, nullptr, nullptr, 0);
    }

private:
    std::atomic<bool> flag;
};

int main() {
    SpinLock lock;
    int count = 0;

    for (int i = 0; i < 10; ++i) {
        std::thread([&]{
            for (int j = 0; j < 100000; ++j) {
                lock.lock();
                ++count;
                lock.unlock();
            }
        }).detach();
    }

    sleep(1);
    std::cout << "count = " << count << std::endl;

    return 0;
}

```

