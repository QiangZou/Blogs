# C++ Enum转Lua Table工具 #


## 观察C++ Enum结构 ##

### 总结结构 ###

```C++
enum GameMessage
{ 
	//*******

    ///******************
	GM_GAMESERVER_INIT_OK = 101,//逻辑服务器初始化完成消息(共享内存部分)
	GM_DATABASESERVER_INIT_OK,//数据库服务器初始化完成
	//GM_REGISTER_GAME_SERVER,
	/*************** 仙盟开始 *************************/
	SM_PILL_RETURN,
}
```

## 分析结构 ##
- enum GameMessage开头
- {}中包含所有枚举注释
- 每一行可能为枚举或注释
  - 枚举 （带,号 带枚举值 带注释）
  - 注释（//开头 ///开头 /*/包含）


## 定义每行的结构类 ##
```C##
public class EnumLineStream
{
    public string name;//枚举名
    public int valuse;//枚举值
    public string annotation;注释
    public bool isStart;是否是开头{
    public bool isEnd;是否是结尾}
}
```

## 读取文件 关键API ##

```C##
StreamReader sr = new StreamReader(路径, Encoding.Default);
string line;
while ((line = sr.ReadLine()) != null)
｛
	解析(line)
｝
```


## 解析思想 ##
- 判断该行是否为{或}
- 判断该行是否为空
- 判断该行是否为注释（//开头 ///开头 /*/包含）
- 判断该行是否为枚举（带,号 带枚举值 带注释）

## 根据解析获得的数据生成文件 关键API ##
```C##
string filePath = Directory.GetCurrentDirectory() + "/GameMessage.lua";//保存文件路径

FileStream fileStream = new FileStream(filePath, FileMode.Create, FileAccess.ReadWrite);//创建文件

StreamWriter streamWriter = new StreamWriter(fileStream);
StringBuilder stringBuilder = new StringBuilder();
for (int i = 0; i < data.Count; i++)
{
	//。。。。读取数据

	stringBuilder.AppendLine(数据);//添加并换行
}
	

streamWriter.Write(stringBuilder);//写入文件
streamWriter.Close();//关闭文件


```





