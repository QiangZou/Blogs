# Unity 代码编译成dll 更新dll实现热更代码

## 实现流程


* 代码编译成DLL
* DLL打包成AssetBundle
* 加载AssetBundle
* 加载代码程序集
* 获取指定类
* 使用反射赋值




## C#代码编译成DLL

* 使用VS创建类库项目
  * 模版->Visual C#-> .NET Framework 3.5-> 类库
  * 名称即为DLL名字（反射的时候要用）
  ![模版->Visual C#-> .NET Framework 3.5-> 类库](https://github.com/QiangZou/Blogs/blob/master/Images/TIM%E6%88%AA%E5%9B%BE20181201180434.png?raw=true)


* 引用两个Unity相关DLL（防止编译报错）   
  * 右键项目->添加->引用
  ![右键项目->添加->引用](https://github.com/QiangZou/Blogs/blob/master/Images/TIM%E6%88%AA%E5%9B%BE20181201181404.png?raw=true)
  * 在引用管理器窗口->浏览->dll路径
  * UnityEngine.dll默认路径：C:\Program Files\Unity\Editor\Data\Managed
  * UnityEngine.UI.dll默认路径：C:\Program Files\Unity\Editor\Data\UnityExtensions\Unity\GUISystem
  ![111](https://github.com/QiangZou/Blogs/blob/master/Images/TIM%E6%88%AA%E5%9B%BE20181201182437.png?raw=true)


* 编写一个继承MonoBehaviour的简单代码

```C##
using UnityEngine;
using UnityEngine.UI;

namespace A
{
    public class Class1 : MonoBehaviour
    {
        public Text text;

        int number = 0;

        void Update()
        {
            if (Input.GetMouseButtonDown(0))
            {
                number++;
                text.text = "鼠标左键按下：" + number;
            }
        }
    }
}
```

* 生成DLL
  * 右键项目 生成
  * 在项目的bin\Debug目录获得DLL


## DLL打包成AssetBundle
* 把生成的DLL后缀修改为bytes（unity不支持dll后缀打包为AssetBundle）（下图1）
* 放入项目中 设置AssetBundleName（下图2）
* 打包代码（放入Editor文件夹）（下图3）


```C#
using UnityEngine;
using System.Collections;
using UnityEditor;
using System.IO;

public class BuildAssetBunble
{
    [MenuItem("BuildAsset/Bunble")]
    public static void Build()
    {
        BuildPipeline.BuildAssetBundles(Application.streamingAssetsPath, BuildAssetBundleOptions.DeterministicAssetBundle, EditorUserBuildSettings.activeBuildTarget);

        AssetDatabase.Refresh();
    }
}
```

* 创建StreamingAssets放入AssetBundle文件（下图4）
* 点击BuildAsset/Bunble按钮（下图5）

![](https://github.com/QiangZou/Blogs/blob/master/Images/TIM%E6%88%AA%E5%9B%BE20181203154840.png?raw=true)

## 测试代码
* 创建一个Text游戏对象
* 新建一个Test代码挂在到Text游戏对象上

```C#

using UnityEngine;
using UnityEngine.UI;
using System;
using System.Reflection;

public class Test : MonoBehaviour
{
    void Start()
    {
        Text text = gameObject.GetComponent<Text>();//获取组建

        string path = string.Empty;

        if (Application.platform == RuntimePlatform.WindowsEditor)
        {
            path = Application.streamingAssetsPath + "/a_dll";
        }
        else if (Application.platform == RuntimePlatform.Android)
        {
            path = Application.streamingAssetsPath + "!assets/a_dll";
        }

        AssetBundle assetBundle = AssetBundle.LoadFromFile(path);//加载AssetBundle

        TextAsset textAsset = assetBundle.LoadAsset<TextAsset>("A");//加载AssetBundle中的A


        Assembly assembly = Assembly.Load(textAsset.bytes);//加载托管程序集

        Type item = assembly.GetType("A.Class1");//获取程序集指定类


        Component comparer = gameObject.AddComponent(item);//添加到游戏对象上

        FieldInfo fieldInfo = comparer.GetType().GetField("text");//使用反射获取实例的字段

        fieldInfo.SetValue(comparer, text);//给字段赋值
    }
}

```
效果如下
![](https://i.imgur.com/uvdiqZJ.gif)


##总结
* 代码图片都有就不上传工程
* 安卓测试完全没问题
* IOS不允许使用动态代码所以GG
* 我这里只是简单实现了 实际上有很多限制