### 最简测试用例

```cpp
#include <iostream>
#include <future>

int sum(int a, int b)
{
    return a + b;
}

int main()
{
    std::packaged_task<int(int, int)> task(sum);
    std::future<int> result = task.get_future();

    // 将任务交给另一个线程执行
    std::thread t(std::move(task), 1, 2);
    t.join();

    std::cout << "1 + 2 = " << result.get() << std::endl; // 输出1 + 2 = 3
    return 0;
}
```

### 详情参考C++新经典17.9.2

![image-20221017105417310](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105417310.png)

![image-20221017105442607](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105442607.png)

![image-20221017105505596](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105505596.png)

![image-20221017105536509](https://hanbabang-1311741789.cos.ap-chengdu.myqcloud.com/Pics/image-20221017105536509.png)

