# Unity Editor模式 Invoke()函数 失效

## 如题今天踩的坑

## 解决方法
使用EditorApplication.update += 自己的Updata()
使用EditorApplication.timeSinceStartup获取update间隔时间