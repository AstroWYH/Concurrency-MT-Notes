### 概念

`memory_order` 是用于指定 C++ 中原子操作的内存序(memory ordering)的枚举类型，它定义了一组常量，每个常量代表一种不同的内存序。

原子操作是一种在多线程程序中实现线程同步和数据一致性的机制。C++11 标准引入了原子操作库，其中的操作都是原子的，可以保证在多线程环境下的正确性。在多线程程序中，原子操作的内存序指定了多个线程之间访问同一个共享变量时读写顺序的规则。通过指定正确的内存序，可以避免数据竞争和不正确的执行顺序，保证多线程程序的正确性。

`memory_order` 的使用场景包括：

1. 多线程程序中使用原子操作时，需要指定操作的内存序。比如，当多个线程同时对同一个共享变量进行操作时，需要指定内存序来保证操作的正确性。
2. 在实现高性能的多线程数据结构时，需要使用原子操作，同时指定合适的内存序来保证数据的一致性和正确性。

常用的 `memory_order` 常量包括：

- `memory_order_relaxed`：允许对内存操作的顺序进行重排，不保证任何内存序的要求。
- `memory_order_acquire`：保证本次原子操作之前的读操作不会被重排到本次原子操作之后。
- `memory_order_release`：保证本次原子操作之后的写操作不会被重排到本次原子操作之前。
- `memory_order_seq_cst`：对所有原子操作进行全局排序，保证所有线程对同一原子变量执行的原子操作按照同一个全局顺序执行。

在使用原子操作时，需要根据具体情况选择合适的内存序。在多数情况下，使用 `memory_order_relaxed` 可以提高程序的性能，但需要注意在某些情况下会导致不正确的结果。同时，使用 `memory_order_seq_cst` 可以保证程序的正确性，但会带来性能上的开销。

### 例1

假设有一个共享的计数器变量 `count`，多个线程需要对它进行递增操作。在多线程环境下，由于多个线程同时访问该变量，需要使用原子操作来保证正确性。同时，为了避免数据竞争和不正确的执行顺序，需要指定适当的内存序。

以下是一个使用 `std::atomic` 和 `memory_order` 的例子：

```cpp
#include <atomic>
#include <thread>

std::atomic<int> count{0};

void increment_count() {
    for (int i = 0; i < 100000; ++i) {
        count.fetch_add(1, std::memory_order_relaxed);
    }
}

int main() {
    std::thread t1(increment_count);
    std::thread t2(increment_count);

    t1.join();
    t2.join();

    std::cout << "Count: " << count << std::endl;
    return 0;
}

```

在上面的代码中，我们创建了两个线程，它们分别执行 `increment_count` 函数。在该函数中，我们使用 `std::atomic` 提供的 `fetch_add` 原子操作来递增计数器。同时，我们使用了 `std::memory_order_relaxed` 来指定内存序，表示读写操作可以按照任意顺序进行，不需要满足任何内存序的要求。

在该例子中，由于只有一种操作，且不需要考虑数据的一致性，所以使用 `std::memory_order_relaxed` 是安全的。但在实际情况下，需要根据具体的使用场景来选择合适的内存序。

### 例2

假设有一个全局变量 `ready`，它表示某个条件是否已经准备就绪。多个线程需要等待该条件准备就绪后再执行相应的操作。在多线程环境下，需要使用原子操作来保证正确性。同时，为了避免不正确的执行顺序，需要使用 `memory_order_acquire` 内存序来保证读操作不会被重排到原子操作之后。

以下是一个使用 `std::atomic` 和 `memory_order_acquire` 的例子：

```cpp
#include <atomic>
#include <thread>

std::atomic<bool> ready{false};

void wait_for_ready() {
    while (!ready.load(std::memory_order_acquire)) {
        // busy-waiting
    }
    std::cout << "Ready!" << std::endl;
}

void set_ready() {
    // do some preparation work
    ready.store(true, std::memory_order_release);
}

int main() {
    std::thread t1(wait_for_ready);
    std::thread t2(set_ready);

    t1.join();
    t2.join();

    return 0;
}

```

在上面的代码中，我们创建了两个线程，其中一个线程执行 `wait_for_ready` 函数，另一个线程执行 `set_ready` 函数。在 `wait_for_ready` 函数中，我们使用 `std::atomic` 提供的 `load` 原子操作来读取 `ready` 变量，同时使用 `std::memory_order_acquire` 来指定内存序，表示读操作不会被重排到原子操作之后。在 `set_ready` 函数中，我们使用 `std::atomic` 提供的 `store` 原子操作来设置 `ready` 变量为 `true`，同时使用 `std::memory_order_release` 来指定内存序，表示写操作不会被重排到原子操作之前。

在该例子中，使用 `std::memory_order_acquire` 内存序可以保证 `wait_for_ready` 函数读取 `ready` 变量的操作不会被重排到 `set_ready` 函数设置 `ready` 变量的操作之后。这样可以确保线程在等待条件准备就绪时不会进入无限循环，从而保证了程序的正确性。