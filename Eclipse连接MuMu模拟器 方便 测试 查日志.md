# Eclipse连接MuMu模拟器 方便 测试 查日志

----------

## 问题由来
- 真机测试麻烦(首先你得拿一部手机,然后在用数据线连接电脑和手机...) 



## 解决流程
- 确保打开MuMu模拟器和Eclipse的DDMS功能
- 进入C:\Program Files (x86)\MuMu\emulator\nemu\vmonitor\bin


- 在当前目录下**Shift+鼠标右键**在此处打开命令行，输入**adb_server connect 127.0.0.1:7555 **


```
adb server is out of date.  killing...
* daemon started successfully *
connected to 127.0.0.1:7555
```

- 窗口显示如上  则连接成功


## 优化

- 每次都需要进入指定目录输入命令很麻烦
- 创建一个bat每次连接只需要双击一下


## 优化流程
- 在C:\Program Files (x86)\MuMu\emulator\nemu\vmonitor\bin目录新建一个bat文件(.bat的文本)
```
@echo off
cmd /k "echo %cd%  & adb_server connect 127.0.0.1:7555"
pause
cls
```
- 填入如上代码保存 发送一个快捷方式到桌面
- 双击运行