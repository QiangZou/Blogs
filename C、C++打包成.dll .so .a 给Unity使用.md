# C、C++打包成.dll .so .a 给Unity使用 #


## 打包.dll库 ##
工具：VS

1. 使用VS新建项目
2. 选择不大于.NET3.5的版本
3. 选择Visual C++ -> Win32 控制台应用程序
4. 输入项目名（dll名字）
5. 下一步
6. 勾选dll->勾选空项目

![](https://github.com/QiangZou/Blogs/blob/master/Images/1.png?raw=true)


![](https://github.com/QiangZou/Blogs/blob/master/Images/2.png?raw=true)


测试代码test.c
```C
#include <stdio.h>//引入C的库函数
#include "test.h" //引入头文件

int add(int a, int b)
{
	return a + b;
}
```
头文件代码test.h
```C
#ifndef  _DLL_TEST_
#define _DLL_TEST_

#if 1
#define EXPORT_DLL __declspec(dllexport) //导出dll声明
#else
#define EXPORT_DLL extern//导出so .a  不需要加声明
#endif

#pragma once

EXPORT_DLL  int add(int a, int b);

#endif
```

test.c放入源文件
test.h放入头文件
![](https://github.com/QiangZou/Blogs/blob/master/Images/3.png?raw=true)

选着X64框架
![](https://github.com/QiangZou/Blogs/blob/master/Images/4.png?raw=true)

- 点击生成解决方案

- 在输出日志查看生成dll路径

- 复制生成的dll到Unity的Plugins路径

## 打包.SO库 ##

工具NDK
- 配置NDK环境
- 测试代码用上面
- 修改test.c代码中的 编译宏

```C
#ifndef  _DLL_TEST_
#define _DLL_TEST_

#if 0//这里
#define EXPORT_DLL __declspec(dllexport) //导出dll声明
#else
#define EXPORT_DLL extern//导出so .a  不需要加声明
#endif

#pragma once

EXPORT_DLL  int add(int a, int b);

#endif
```

- test.c目录 新建文件Android.mk和Application.mk



Android.mk文件
```C
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)

LOCAL_MODULE     :=  zq
LOCAL_C_INCLUDES := $(LOCAL_PATH)
LOCAL_SRC_FILES  := test.c
LOCAL_LDLIBS     := -llog -landroid
LOCAL_CFLAGS    := -DANDROID_NDK

include $(BUILD_SHARED_LIBRARY)

```

Application.mk文件
```C
APP_STL := gnustl_static
APP_CPPFLAGS := -frtti -std=c++11
APP_PLATFORM := android-19
APP_CFLAGS += -Wno-error=format-security
APP_BUILD_SCRIPT := Android.mk
APP_ABI := armeabi-v7a x86
```
- 使用cmd命令CD到test.c目录
- 运行命令
```C
ndk-build NDK_PROJECT_PATH=. NDK_APPLICATION_MK=Application.mk
```
![](https://github.com/QiangZou/Blogs/blob/master/Images/5.png?raw=true)

- 复制生成的libs文件到Unity的Plugins/Andriod路径
- 在Unity中选中SO文件修改属性

![](https://github.com/QiangZou/Blogs/blob/master/Images/6.png?raw=true)

## 打包.a库 ##
工具xcode
- 新建静态库工程
  - Create a New Xcode project -> Framework&Library -> Cocoa Touch Static Library 
- 配置最低支持版本
	- 选中项目	选中info 选中Deployment Target 修改版本
- 配置适配所有模拟器架构
	- project  buildSeting  Build Active Architecture Only 设为NO
- 添加代码
	- 使用生成SO文件的代码
- 添加公开文件
	- 选中项目	选中Build Phasses 选中+图标 选中New Headers Phase 选中New Headers Phase+图表 添加头文件 拖入Pulic栏
- 生成文件
- 复制到unity的Plugins/IOS路径



## 测试代码 ##
```C#
using UnityEngine;
using System.Collections;
using System.Runtime.InteropServices;

public class NewBehaviourScript : MonoBehaviour
{

#if UNITY_IPHONE && !UNITY_EDITOR
    const string dllName = "__Internal";
#else
    const string dllName = "zq";
#endif


    [DllImport(dllName)]
    private static extern int add(int x, int y);

    void Start()
    {
        Debug.Log("zqzqzqzq " + add(111, 111));
    }
}

```

相关链接
https://blog.csdn.net/l773575310/article/details/72461579
https://blog.csdn.net/yangxuan0261/article/details/52420833
https://blog.csdn.net/u014361280/article/details/80693368
