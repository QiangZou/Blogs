# Unity C# 字符串判断失败

## 问题又来

```
public class Test
{
    public string text;//被其他脚本赋值为 "招募"

    bool Check()
    {
        if (text == "招募")
        {
            return true;
        }
        return false;
    }
	//返回一直为false
}
```

## 解决方案


最后高人值点 
是脚本的编码问题
改成UTF8就好了