# Unity C# 多线程

## 创建调用一个简单的线程 thread.Start()

```c#
using System.Threading;
using UnityEngine;

public class ThreadExample : MonoBehaviour
{
    void Start()
    {
        Thread thread = new Thread(Fun);
        thread.Start();
    }

    void Fun()
    {
        Debug.Log("Fun");
    }
}
```

## 暂停一个线程 Thread.Sleep()

```c#
using System;
using System.Threading;
using UnityEngine;

public class ThreadExample1 : MonoBehaviour
{
    void Start()
    {
        Thread thread = new Thread(Fun);
        thread.Name = "a";
        thread.Start();
        Debug.Log("主线程调用完成");
    }

    void Fun()
    {
        Thread.Sleep(TimeSpan.FromSeconds(2));//暂停2秒
        Debug.Log(Thread.CurrentThread.Name);
        Debug.Log("Fun");
    }
}
```

## 柱塞其他线程 thread.Join()

```c#
using System;
using System.Threading;
using UnityEngine;

public class ThreadExample2 : MonoBehaviour
{
    void Start()
    {
        Thread thread = new Thread(Fun);
        thread.Start();
        thread.Join();//阻塞其他线程
        Debug.Log("子线程调用完成");
    }

    void Fun()
    {
        Thread.Sleep(TimeSpan.FromSeconds(2));
        Debug.Log("Fun");
    }
}
```

## 线程状态

```c#
using System.Threading;
using UnityEngine;

public class ThreadExample3 : MonoBehaviour
{
    void Start()
    {
        Thread thread = new Thread(Fun);

        Debug.Log(thread.ThreadState.ToString());

        thread.Start();

        Debug.Log(thread.ThreadState.ToString());

        thread.Join();

        Debug.Log(thread.ThreadState.ToString());
    }

    void Fun()
    {
        Debug.Log("Fun");
    }
}
```

## 线程传递参数

- 使用参数为ParameterizedThreadStart(object obj)的函数作为线程函数 调用void Start(object parameter)传递参数
- 使用lambada表达式封包机制传递参数 原理下面的一样 C#编辑器会帮我们实现这个类
- 将类的成员函数作为线程函数，成员函数获取成员变量

```c#
using System;
using System.Threading;
using UnityEngine;

public class ThreadExample4 : MonoBehaviour
{
    private int value = 0;

    void Start()
    {
        Thread thread = new Thread(Fun);
        thread.Name = "a";
        thread.Start(10);
        thread.Join();

        Thread thread1 = new Thread(() =>
        {
            Fun(20);
        });
        thread1.Name = "b";
        thread1.Start();
        thread1.Join();

        value = 30;
        Thread thread2 = new Thread(Fun1);
        thread2.Name = "c";
        thread2.Start();
    }

    void Fun(object value)
    {
        Debug.Log(Thread.CurrentThread.Name);
        Debug.Log(value.ToString());
    }
    void Fun1()
    {
        Debug.Log(Thread.CurrentThread.Name);
        Debug.Log(this.value.ToString());//使用类的成员变量
    }
}
```

