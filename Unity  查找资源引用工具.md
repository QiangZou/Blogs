# Unity 查找资源引用工具

## 问题由来

- 想查看组件被那些预设场景使用
- 想查看图片被那些预设场景使用
- 想查看资源被那些预设场景使用

## 解决方法

- 遍历项目中所有资源判断是否引用


## 解决流程

- 获取选中对象GUID
- 使用Thread遍历项目中所有的资源
- 判断资源是否包含GUID

## 使用方式

- 选中脚本
- 右键
- ZQ -> 工具 -> 查找资源引用

## 源码

```
using System.Collections.Generic;
using System.IO;
using System.Threading;
using UnityEditor;
using UnityEngine;

public static class FindreAssetFerencesTool
{
    static string[] assetGUIDs;
    static string[] assetPaths;
    static string[] allAssetPaths;
    static Thread thread;

    [MenuItem("ZQ/工具/查找资源引用", false)]
    [MenuItem("Assets/ZQ/工具/查找资源引用", false, 1)]
    static void FindreAssetFerencesMenu()
    {
        Debug.LogError("查找资源引用");

        if (Selection.assetGUIDs.Length == 0)
        {
            Debug.LogError("请先选择任意一个组件，再右键点击此菜单");
            return;
        }

        assetGUIDs = Selection.assetGUIDs;

        assetPaths = new string[assetGUIDs.Length];

        for (int i = 0; i < assetGUIDs.Length; i++)
        {
            assetPaths[i] = AssetDatabase.GUIDToAssetPath(assetGUIDs[0]);
        }

        allAssetPaths = AssetDatabase.GetAllAssetPaths();

        thread = new Thread(new ThreadStart(FindreAssetFerences));
        thread.Start();
    }


    static void FindreAssetFerences()
    {
        List<string> logInfo = new List<string>();
        string path;
        string log;
        for (int i = 0; i < allAssetPaths.Length; i++)
        {
            path = allAssetPaths[i];

            Debug.Log("正在查找文件：" + path);

            if (path.EndsWith(".prefab") || path.EndsWith(".unity"))
            {
                string content = File.ReadAllText(path);
                if (content == null)
                {
                    continue;
                }

                for (int j = 0; j < assetGUIDs.Length; j++)
                {
                    if (content.IndexOf(assetGUIDs[j]) > 0)
                    {
                        log = string.Format("{0} 引用了 {1}", path, assetPaths[j]);
                        logInfo.Add(log);
                    }
                }
            }
        }

        for (int i = 0; i < logInfo.Count; i++)
        {
            Debug.LogError(logInfo[i]);
        }

        Debug.LogError("选择对象引用数量：" + logInfo.Count);

        Debug.LogError("查找完成");
    }
}
```