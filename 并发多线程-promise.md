### 最简测试用例

```cpp
#include <iostream>
#include <future>

int main()
{
    std::promise<int> p;
    std::future<int> result = p.get_future();

    std::thread t([&p]() {
        p.set_value(42);
        });

    std::cout << "The answer is " << result.get() << std::endl; // 输出The answer is 42
    t.join();

    return 0;
}

```



### 详情参考C++新经典17.9.3

![image-20221017105625702](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105625702.png)

![image-20221017105658011](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105658011.png)

![image-20221017105801964](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105801964.png)