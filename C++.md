# C++

## 介绍

- 静态类型
- 编译式的
- 通用的
- 大小写敏感的
- 不规则的编程语言
- 支持过程化编程
- 面向对象编程（封装、抽象、继承、多态）
- 泛型编程
- 中级语言（C的超集）

## 基本语法

- **对象 -** 对象具有状态和行为。例如：一只狗的状态 - 颜色、名称、品种，行为 - 摇动、叫唤、吃。对象是类的实例。
- **类 -** 类可以定义为描述对象行为/状态的模板/蓝图。
- **方法 -** 从基本上说，一个方法表示一种行为。一个类可以包含多个方法。可以在方法中写入逻辑、操作数据以及执行所有的动作。
- **即时变量 -** 每个对象都有其独特的即时变量。对象的状态是由这些即时变量的值创建的。

## 数据类型

### 基本数据类型

- bool
- char
- int
- float
  - 4字节 32位 1位符号 8位指数 23位小数
- double
  - 8字节 64位 1位符号 11位指数 52位小数
- void
- wchar_t
- emum

### 修饰符（可多个）

- signed 有符号（可负数）
- unsigned 无符号（整数）
- short 短的 2字节
- long 长的 8字节
- 类型限定符
  - const 不能被修改
  - volatile 不需要优化volatile声明的变量，让程序可以直接从内存中读取变量
  - restrict 不知道有啥用

### typedef 声明

- 给组合数据类型取一个名称

- ```c++
  typedef long int int64;
  int64 number;
  ```

- ```
  typedef struct Books
  {
     char  title[50];
     char  author[50];
     char  subject[100];
     int   book_id;
  }Books;
  Books myBook1;//自定义别名
  ```

  

## 常量

### 整数常量

- 前缀：0x 或 0X 表示十六进制
- 前缀：0表示八进制
- 不带前缀为十进制
- 后缀：U或u表示无符号整数
- 后缀：L或l表示长整数

### 浮点常量

- 由整数部分、小数点、小数部分和指数部分组成
- 小数形式表示时，必须包含整数部分、小数部分，或同时包含两者
- 指数形式表示时， 必须包含小数点、指数，或同时包含两者。带符号的指数是用 e 或 E 引入的

### 字符常量

-  L'x'（仅当大写时）开头，则表示它是一个宽字符常量，存储在 **wchar_t** 类型中
- 前面有反斜杠时，它们就具有特殊的含义，被用来表示如换行符（\n）或制表符（\t）等

### 定义常量

- 使用 **#define** 预处理器。
- 使用 **const** 关键字。

## 储存类

- auto
- register
  - 定义存储在寄存器中而不是 RAM 中的局部变量（有硬件限制）
  - 不能对它应用一元的 '&' 运算符（因为它没有内存位置）
- static
  - 修饰局部变量可以在函数调用之间保持局部变量的值
  - 修饰全局变量会使变量的作用域限制在声明它的文件内
- extern
  - 提供一个全局变量的引用
  - 
- mutable
- thread_local 
  - 声明的变量仅可在它在其上创建的线程上访问
  - 变量在创建线程时创建，并在销毁线程时销毁

## 运算符

- 算术运算符
  - +-*/（++）（--）
- 关系运算符
  - ==  != > < >= <=
- 逻辑运算符
  - && 且||或 !非
- 位运算符（二进制）
  - & AB都为1则为1
  - | AB有一个为1则为1
  - ^  AB不相等则为1
  - ~ 取反
  - << 向左移动右操作数指定的位数
  - \>> 向右移动右操作数指定的位数
- 赋值运算符
- 杂项运算符
  - sizeof 运算符:  返回变量的大小
  - 条件运算符 Condition ? X : Y   如果 Condition 为真 ? 则值为 X : 否则值为 Y
  - 逗号运算符,  是以逗号分隔的列表中的最后一个表达式的值
  - 成员运算符.（点）和 ->（箭头）  引用类、结构和共用体的成员
  - 强制转换运算符: 把一种数据类型转换为另一种数据类型。例如，int(2.2000) 将返回 2
  - 指针运算符:&*  返回变量的地址
    - 取地址运算符 &
    - 间接寻址运算符 *
  - 指针运算符 :* 指向一个变量。例如，*var; 将指向变量 var。
  - 流插入运算符:<<  输出
  - 流提取运算符:<<  输入

## 函数

### 函数参数

- 传值调用
  - 修改形式参数对实际参数没影响
- 指针调用
  - 修改形式参数会影响实际参数
- 引用调用
  - 修改形式参数会影响实际参数

### 内置函数

- **double cos(double);**

  该函数返回弧度角（double 型）的余弦。

- **double sin(double);**
  该函数返回弧度角（double 型）的正弦。

- **double tan(double);**
  该函数返回弧度角（double 型）的正切。

- **double log(double);**
  该函数返回参数的自然对数。

- **double pow(double, double);**
  假设第一个参数为 x，第二个参数为 y，则该函数返回 x 的 y 次方。

- **double hypot(double, double);**
  该函数返回两个参数的平方总和的平方根，也就是说，参数为一个直角三角形的两个直角边，函数会返回斜边的长度。

- **double sqrt(double);**
  该函数返回参数的平方根。

- **int abs(int);**
  该函数返回整数的绝对值。

- **double fabs(double);**
  该函数返回任意一个浮点数的绝对值。

- **double floor(double);**
  该函数返回一个小于或等于传入参数的最大整数。

## 指针

- &变量   访问内存中的一个地址
- **指针**是一个变量，其值为另一个变量的地址

## 引用

### 引用于指针的区别

- 不存在空引用。引用必须连接到一块合法的内存
- 一旦引用被初始化为一个对象，就不能被指向到另一个对象。指针可以在任何时候指向到另一个对象
- 引用必须在创建时被初始化。指针可以在任何时间被初始化

## 基本的输入输出

### 标准输出流 cout

```c++
char str[] = "Hello C++";
cout << "Value of str is : " << str << endl;
```

### 标准输入流 cin

```c++
 char name[50];
 cout << "请输入您的名称： ";
 cin >> name;
 cout << "您的名称是： " << name << endl;
```

### 标准错误流 cerr

```
char str[] = "Unable to read....";
cerr << "Error message : " << str << endl;
```

### 标准日志流 clog

```
char str[] = "Unable to read....";
clog << "Error message : " << str << endl;
```

## 类

### 类成员函数

- 定义在类内部

```C++
class Box
{
   public:
      double length;      // 长度
      double breadth;     // 宽度
      double height;      // 高度
      double getVolume(void)
      {
         return length * breadth * height;
      }
};
```

- 定义在类外部
```c++
double Box::getVolume(void)
{
    return length * breadth * height;
}
```

### 构造函数 析构函数

### 拷贝构造函数

- 类没定义拷贝构造函数，编译器会自行定义一个

- 带指针变量且有动态内存分配，必须实现拷贝构造函数

- 常见形式

  - ```c++
    classname (const classname &obj) {
       // 构造函数的主体
    }
    ```

- ```c++
  class Line
  {
     public:
        int getLength( void );
        Line( int len );             // 简单的构造函数
        Line( const Line &obj);      // 拷贝构造函数
        ~Line();                     // 析构函数
   
     private:
        int *ptr;
  };
   
  // 成员函数定义，包括构造函数
  Line::Line(int len)
  {
      cout << "调用构造函数" << endl;
      // 为指针分配内存
      ptr = new int;
      *ptr = len;
  }
   
  Line::Line(const Line &obj)
  {
      cout << "调用拷贝构造函数并为指针 ptr 分配内存" << endl;
      ptr = new int;
      *ptr = *obj.ptr; // 拷贝值
  }
   
  Line::~Line(void)
  {
      cout << "释放内存" << endl;
      delete ptr;
  }
  int Line::getLength( void )
  {
      return *ptr;
  }
   
  void display(Line obj)
  {
     cout << "line 大小 : " << obj.getLength() <<endl;
  }
  ```

### 友元函数

- 友元可以是一个函数也可以是一个类
- 友元类的所有成员都是友元
- 友元函数可以访问私有成员
- （相当于C#可访问私有成员的类静态函数？）
- 声明需要使用关键字friend

### 内联函数

- 需要在函数名前面放置关键字 inline
- 编译时会把函数代码副本放置在每个调用该函数的地方

  ### 类的继承

- 派生类不继承基类以下方法
  - 构造函数、析构函数、拷贝构造函数
  - 重载运算符
  - 友元函数（啥玩意）
- 多继承
  
  - 子类可以继承多个父类

### 多态

- 基类成员函数需要添加关键字virtual
  
  - 否则子类调用同名函数会调用到子类中去
  
- 纯虚函数

  - ```c++
    virtual int area() = 0;
    ```


## 文件和流

- 标准库
  - ofstream 创建写入
  - ifstream 读取
  - fstream 创建写入、读取

### 打开文件

```c++
void open(const char *filename, ios::openmode mode);
```

| 模式标志   | 描述                                                         |
| :--------- | :----------------------------------------------------------- |
| ios::app   | 追加模式。所有写入都追加到文件末尾。                         |
| ios::ate   | 文件打开后定位到文件末尾。                                   |
| ios::in    | 打开文件用于读取。                                           |
| ios::out   | 打开文件用于写入。                                           |
| ios::trunc | 如果该文件已经存在，其内容将在打开文件之前被截断，即把文件长度设为 0。 |

- 模式可以使用多个

  - ```c++
    ofstream outfile;
    outfile.open("file.dat", ios::out | ios::trunc );//覆盖文件
    ```

### 关闭文件

```c++
void close();
```

### 文件位置指针

- **istream** **ostream** 接口：**seekg**
  - 第一个参数是长整型
  - 第二个参数是查找方向
    - **ios::beg**（默认的，从流的开头开始定位）
    -  **ios::cur**（从流的当前位置开始定位）
    -  **ios::end**（从流的末尾开始定位）

## 异常处理

- **throw** 抛出一个异常

- **try** try 块中放置可能抛出异常的代码

- **catch **捕获不同类型的异常

- 捕获任何类型异常（括号中使用省略号）

  - ```
    try
    {
       // 保护代码
    }catch(...)
    {
      // 能处理任何异常的代码
    }
    ```

### 标准的异常

- C++ 提供了一系列标准的异常，定义在 **exception**中

### 定义新的异常

- 通过继承和重载 **exception** 类来定义新的异常

## 动态内存

- **栈：**在函数内部声明的所有变量都将占用栈内存。
- **堆：**这是程序中未使用的内存，在程序运行时可用于动态分配内存。
- **new** 数据类型; 分配内存 创建对象（储存器被用完 无法分配内存）
- **delete** 变量; 释放内存

## 命名空间

### 嵌套的命名空间

- 定义

  - ```c++
    namespace namespace_name1 {
     // 代码声明
     namespace namespace_name2 {
        // 代码声明
     }
    }
    ```

- 使用

  - ```c++
    // 访问 namespace_name2 中的成员
    using namespace namespace_name1::namespace_name2;
    ```

## 模板

- 模板是泛型编程的基础

- 泛型编程独立于特定类型的方式编程

- 函数模板

  - ```c++
    template <typename T>
    inline T const& Max (T const& a, T const& b) 
    { 
        return a < b ? b:a; 
    }
    ```

- 类模板

  - ```c++
    template <class T>
    class Stack { 
      private: 
        vector<T> elems;     // 元素 
     
      public: 
        void push(T const&);  // 入栈
        void pop();               // 出栈
        T top() const;            // 返回栈顶元素
        bool empty() const{       // 如果为空则返回真。
            return elems.empty(); 
        } 
    }; 
    ```

    

## 预处理器

- #define 预处理

  - ```c++
    #define PI 3.14159//常量
    cout << "Value of PI :" << PI << endl; 
    ```

  - 编译后会替换值

    - ```c++
      #define PI 3.14159
      cout << "Value of PI :" << 3.14159 << endl; 
      ```

- 参数宏

  - ```c++
    #define MIN(a,b) (a<b ? a : b)
    ```

- 条件编译

  - ```c++
    #ifdef DEBUG
       cerr <<"Variable x = " << x << endl;
    #endif
    ```

- #和##运算符（字符串需求？）

  - \# 运算符会把令牌转换为用引号引起来的字符串。
  - \## 运算符用于连接两个令牌

- 预定义宏（用于日志？）

  - | 宏       | 描述                                                         |
    | :------- | :----------------------------------------------------------- |
    | \_\_LINE\_\_ | 这会在程序编译时包含当前行号。                               |
    | \_\_FILE\_\_ | 这会在程序编译时包含当前文件名。                             |
    | \_\_DATE\_\_ | 这会包含一个形式为 month/day/year 的字符串，它表示把源文件转换为目标代码的日期。 |
    | \_\_TIME\_\_ | 这会包含一个形式为 hour:minute:second 的字符串，它表示程序被编译的时间。 |

## 信号处理

| 信号    | 描述                                         |
| :------ | :------------------------------------------- |
| SIGABRT | 程序的异常终止，如调用 **abort**。           |
| SIGFPE  | 错误的算术运算，比如除以零或导致溢出的操作。 |
| SIGILL  | 检测非法指令。                               |
| SIGINT  | 程序终止(interrupt)信号。                    |
| SIGSEGV | 非法访问内存。                               |
| SIGTERM | 发送到程序的终止请求。                       |

- signal() 函数     捕获突发事件

  - ```c++
    void signalHandler( int signum )
    {
        cout << "Interrupt signal (" << signum << ") received.\n";
     
        // 清理并关闭
        // 终止程序  
     
       exit(signum);  
     
    }
    signal(SIGINT, signalHandler);  
    ```

-  raise() 函数 生成信号

  - ```c++
    raise( SIGINT);
    ```

## 多线程

- 基于进程的多任务处理是程序的并发执行

- 基于线程的多任务处理是同一程序的片段的并发执行。

- 创建线程

  - ```c++
    #include <pthread.h>
    pthread_create (thread, attr, start_routine, arg) 
    ```

- 终止线程

  - ```c++
    #include <pthread.h>
    pthread_exit (status) 
    ```

- 实例

  - ```c++
    #include <iostream>
    // 必须的头文件
    #include <pthread.h>
     
    using namespace std;
     
    #define NUM_THREADS 5
     
    // 线程的运行函数
    void* say_hello(void* args)
    {
        cout << "Hello Runoob！" << endl;
        return 0;
    }
     
    int main()
    {
        // 定义线程的 id 变量，多个变量使用数组
        pthread_t tids[NUM_THREADS];
        for(int i = 0; i < NUM_THREADS; ++i)
        {
            //参数依次是：创建的线程id，线程参数，调用的函数，传入的函数参数
            int ret = pthread_create(&tids[i], NULL, say_hello, NULL);
            if (ret != 0)
            {
               cout << "pthread_create error: error_code=" << ret << endl;
            }
        }
        //等各个线程退出后，进程才结束，否则进程强制结束了，线程可能还没反应过来；
        pthread_exit(NULL);
    }
    ```

- 向线程传递参数

- 连接和分离线程

## STL标准模板库

| 组件                | 描述                                                         |
| :------------------ | :----------------------------------------------------------- |
| 容器（Containers）  | 容器是用来管理某一类对象的集合。C++ 提供了各种不同类型的容器，比如 deque、list、vector、map 等。 |
| 算法（Algorithms）  | 算法作用于容器。它们提供了执行各种操作的方式，包括对容器内容执行初始化、排序、搜索和转换等操作。 |
| 迭代器（iterators） | 迭代器用于遍历对象集合的元素。这些集合可能是容器，也可能是容器的子集。 |