#Unity 处理预设中的中文

##需求由来
- 项目接入越南版本

##需要解决的文本问题
- 获取UI预设Label里面的中文(没被代码控制)提供给越南
- Label里面的中文替换成越南文

##解决流程

* 迭代获取Assets目录下所有文件
* 获取所有的.prefab预设文件
* 加载预设文件
* 获取预设下所有的UILabel组建
* 判断UILabel中的值是否为中文
* 把所有的中文实例化成文本

* 替换成越南文
* 保存实例化对象为预设文件
* 销毁实例化对象



##实现代码
- 获取UI预设Label里面的中文
```C#
[MenuItem("检查预设中文并且生成文本")]
    static void CheckChinesePrefabsAndSerialization()
    {
        List<string> paths = GetAllFilePaths();

        List<string> prefabPaths = GetAllPrefabFilePaths(paths);

        if (prefabPaths == null)
        {
            return;
        }

        List<string> text = new List<string>();

        for (int i = 0; i < prefabPaths.Count; i++)
        {
            string prefabPath = prefabPaths[i];

            //修改路径格式
            prefabPath = ChangeFilePath(prefabPath);

            AssetImporter tmpAssetImport = AssetImporter.GetAtPath(prefabPath);

            GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(tmpAssetImport.assetPath);

            if (prefab == null)
            {
                continue;
            }

            UILabel[] uiLabels = prefab.GetComponentsInChildren<UILabel>(true);

            for (int j = 0; j < uiLabels.Length; j++)
            {
                UILabel uiLabel = uiLabels[j];

                if (IsIncludeChinese(uiLabel.text))
                {
                    Debug.LogError(string.Format("路径:{0} 预设名:{1} 对象名:{2} 中文:{3}", prefabPath, prefab.name, uiLabel.name, uiLabel.text));
                    text.Add(uiLabel.text);
                }
            }

            //进度条
            float progressBar = (float)i / prefabPaths.Count;
            EditorUtility.DisplayProgressBar("检查预设中文", "进度 ：" + ((int)(progressBar * 100)).ToString() + "%", progressBar);
        }

        SerializationText(Application.dataPath + "/中文.txt", text);

        EditorUtility.ClearProgressBar();

        AssetDatabase.Refresh();

        Debug.Log("完成检查预设中文并且生成文本");
    }
```

- Label里面的中文替换成越南文

```C#
    [MenuItem("ZouQiang/Prefab(预设)/检查预设中文并且替换为越南文")]
    static void CheckChinesePrefabsAndReplaceChinese()
    {
        List<string> paths = GetAllFilePaths();

        List<string> prefabPaths = GetAllPrefabFilePaths(paths);

        if (prefabPaths == null)
        {
            return;
        }

        for (int i = 0; i < prefabPaths.Count; i++)
        {
            string prefabPath = prefabPaths[i];

            //修改路径格式
            prefabPath = ChangeFilePath(prefabPath);

            AssetImporter tmpAssetImport = AssetImporter.GetAtPath(prefabPath);

            GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>(tmpAssetImport.assetPath);

            if (prefab == null)
            {
                continue;
            }

            GameObject obj = Instantiate(prefab) as GameObject;

            UILabel[] uiLabels = obj.GetComponentsInChildren<UILabel>(true);

            bool isChange = false;

            for (int j = 0; j < uiLabels.Length; j++)
            {
                UILabel uiLabel = uiLabels[j];

                if (IsIncludeChinese(uiLabel.text))
                {
                    Debug.LogError(string.Format("路径:{0} 预设名:{1} 对象名:{2} 中文:{3}", prefabPath, prefab.name, uiLabel.name, uiLabel.text));
                    uiLabel.text = "越南文";
                    isChange = true;
                }
            }

            if (isChange)
            {
                PrefabUtility.ReplacePrefab(obj, prefab, ReplacePrefabOptions.ReplaceNameBased);
            }

            DestroyImmediate(obj);

            //进度条
            float progressBar = (float)i / prefabPaths.Count;
            EditorUtility.DisplayProgressBar("检查预设中文并且替换为越南文", "进度 ：" + ((int)(progressBar * 100)).ToString() + "%", progressBar);
        }

        EditorUtility.ClearProgressBar();

        AssetDatabase.Refresh();

        Debug.Log("检查预设中文并且替换为越南文");
    }
```

##相关代码接口
- 迭代获取目录下所有文件路径
```C#
public static void IterationGetFilesPath(string directory, List<string> outPaths)
    {
        string[] files = Directory.GetFiles(directory);

        outPaths.AddRange(files);

        string[] childDirectories = Directory.GetDirectories(directory);

        if (childDirectories != null && childDirectories.Length > 0)
        {
            for (int i = 0; i < childDirectories.Length; i++)
            {
                string dir = childDirectories[i];
                if (string.IsNullOrEmpty(dir)) continue;
                IterationGetFilesPath(dir, outPaths);
            }
        }
    }
```

- 获取项目Assets下所有文件路径
```C#
    public static List<string> GetAllFilePaths()
    {
        List<string> paths = new List<string>();

        IterationGetFilesPath(Application.dataPath, paths);

        return paths;
    }
```


- 获取所有预设文件路径
```C#
    public static List<string> GetAllPrefabFilePaths(List<string> paths)
    {
        if (paths == null)
        {
            return null;
        }

        List<string> prefabPaths = new List<string>();

        for (int i = 0; i < paths.Count; i++)
        {
            string path = paths[i];

            if (path.EndsWith(".prefab") == true)
            {
                prefabPaths.Add(path);
            }

            //进度条
            float progressBar = (float)i / paths.Count;
            EditorUtility.DisplayProgressBar("获取所有预设文件路径", "进度 ： " + ((int)(progressBar * 100)).ToString() + "%", progressBar);
        }

        EditorUtility.ClearProgressBar();

        return prefabPaths;
    }
```

- 是否包含是否有中文
```C#
    public static bool IsIncludeChinese(string content)
    {
        string regexstr = @"[\u4e00-\u9fa5]";

        if (Regex.IsMatch(content, regexstr))
        {
            return true;
        }
        else
        {
            return false;
        }
    }
```

- 改变路径 例如  "C:/Users/XX/Desktop/aaa/New Unity Project/Assets\a.prefab" 改变成 "Assets/a.prefab"
```C#
    public static string ChangeFilePath(string path)
    {
        path = path.Replace("\\", "/");
        path = path.Replace(Application.dataPath + "/", "");
        path = "Assets/" + path;

        return path;
    }
```

- 序列化
```C#
    public static void SerializationText(string filePath, List<string> content)
    {
        if (content == null)
        {
            return;
        }

        FileStream fileStream = new FileStream(filePath, FileMode.Create, FileAccess.ReadWrite);
        StreamWriter streamWriter = new StreamWriter(fileStream);

        StringBuilder stringBuilder = new StringBuilder();

        for (int i = 0; i < content.Count; i++)
        {
            stringBuilder.AppendLine(content[i]);
        }

        streamWriter.Write(stringBuilder);

        streamWriter.Close();
    }
```