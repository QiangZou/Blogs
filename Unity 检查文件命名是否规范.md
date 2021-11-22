# Unity 检查文件命名是否规范

## 问题由来

- 非程序人员经常上传一些命名不规范的文件到Unity中

## 解决方式

- 命名不规范文件不能上传到库中（以后做）
- Unity加载到命名不规范文件的时候弹框提示（以后做）
- 定期扫描项目中所有文件

## 如果扫描项目中所有文件

- 技术点
  - 如何获取Unity Asset 中所有文件命名
    - Application.dataPath API可获取本地Asset目录
    - DirectoryInfo 目录信息类 可获取该目录下所有目录DirectoryInfo、文件FileSystemInfo
    - 使用迭代方式获取Asset目录所有文件
  - 如何判断命名是否规范
    - 规范定义
      - 只包含小写字母和大写字母
      - 是否包含空格看项目情况
    - 遍历文件命名字符串每个char是否符合规范

## 使用方式

- 把源码放入工程
- 在Unity顶部的菜单栏点击 Tool->文件命名是否规范
- 查看Console面版日志

## 源码

```c#
using System.Collections.Generic;
using System.IO;
using UnityEditor;
using UnityEngine;

/// <summary>
/// 文件夹
/// </summary>
public class Folders
{
    [MenuItem("Tools/文件命名是否规范")]
    static void CheckFolderName()
    {
        Folders folder = new Folders(Application.dataPath);

        List<string> paths = new List<string>();

        CheckFolderName(folder, paths);

        foreach (var item in paths)
        {
            Debug.LogError(item);
        }

        Debug.Log("检查完成");
    }

    static void CheckFolderName(Folders folder, List<string> paths)
    {
        foreach (var item in folder.dicFileSystemInfo)
        {
            string name = item.Key;
            FileSystemInfo fileSystemInfo = item.Value;

            if (fileSystemInfo.Extension == ".meta" || fileSystemInfo.Extension == ".DS_Store")
            {
                continue;
            }

            bool isIllegal = IsIllegal(name, true, false);
            if (isIllegal == false)
            {
                paths.Add(fileSystemInfo.FullName);
            }
        }

        foreach (var item in folder.listFolder)
        {
            CheckFolderName(item, paths);
        }
    }

    static bool IsIllegal(string str, bool isDigit = true, bool isSpace = true)
    {
        for (int i = 0; i < str.Length; i++)
        {
            char c = str[i];

            if (char.IsLower(c) || char.IsUpper(c))
            {
                return true;
            }
            if (char.IsDigit(c) && isDigit == true)
            {
                return true;
            }
            if (char.IsWhiteSpace(c) && isSpace == true)
            {
                return true;
            }
        }
        return false;
    }


    /// <summary>
    /// 当前目录信息
    /// </summary>
    public DirectoryInfo currentDirectoryInfo;
    /// <summary>
    /// 当前文件夹中的文件夹
    /// </summary>
    public List<Folders> listFolder;
    /// <summary>
    /// 当前文件夹的文件
    /// </summary>
    public List<FileInfo> listFileInfo;
    /// <summary>
    /// 当前文件夹中所有文件
    /// </summary>
    public Dictionary<string, FileSystemInfo> dicFileSystemInfo;

    public Folders(string path)
    {
        currentDirectoryInfo = new DirectoryInfo(path);

        Init();
    }

    Folders(DirectoryInfo directoryInfo)
    {
        currentDirectoryInfo = directoryInfo;

        Init();
    }

    void Init()
    {
        listFileInfo = new List<FileInfo>();
        foreach (var item in currentDirectoryInfo.GetFiles())
        {
            listFileInfo.Add(item);
        }
        listFolder = new List<Folders>();
        foreach (var item in currentDirectoryInfo.GetDirectories())
        {
            listFolder.Add(new Folders(item));
        }

        dicFileSystemInfo = new Dictionary<string, FileSystemInfo>();
        foreach (var item in listFileInfo)
        {
            dicFileSystemInfo.Add(item.Name, item);
        }
        foreach (var item in listFolder)
        {
            dicFileSystemInfo.Add(item.currentDirectoryInfo.Name, item.currentDirectoryInfo);
        }
    }
}

```

