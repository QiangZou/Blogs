# Unity 自动编译后启动Unity

## 需求由来

- 因为项目特别大所以关闭了Auto Refresh自动刷新
  - 防止代码还没写完Unity就自动加载
- 每次写完代码都需要
  - 手动Ctrl+R刷新资源（耗时1分钟左右）
  - 手动运行Unity
  - 需要2个操作  中间容易出小差
- 系统响应时间太长 效率很低

## 解决方案

- 主动刷新资源后 监测资源加载完成  启动Unity

## 使用方式

- 快捷键：Ctrl+Alt+R
- 菜单栏：ZQFramwork/自动编译后启动Unity

## 源码

```c#
using UnityEditor;
using UnityEngine;

public class AutoCompilePlay : EditorWindow
{
    [MenuItem("ZQFramwork/自动编译后启动Unity %&r", false, 0)]
    public static void Open()
    {
        AutoCompilePlay me = GetWindow<AutoCompilePlay>();
        me.titleContent = new GUIContent("自动启动工具");
        me.minSize = new Vector2(200, 100);
        me.maxSize = me.minSize;

        EditorApplication.isPlaying = false;//停止运行
        AssetDatabase.Refresh();//刷新资源
    }

    //每秒10帧调用
    private void OnInspectorUpdate()
    {
        Repaint();//重绘
    }

    private void OnGUI()
    {
        EditorGUILayout.Space();
        EditorGUILayout.Space();
        EditorGUILayout.Space();
        EditorGUILayout.Space();
        EditorGUILayout.Space();

        if (EditorUtility.scriptCompilationFailed)
        {
            Debug.LogError("编译报错");
            Close();
            return;
        }

        if (EditorApplication.isCompiling)
        {
            EditorGUILayout.LabelField("正在编译");
            return;
        }

        if (Application.isPlaying == false)
        {
            EditorGUILayout.LabelField("正在启动");
            EditorApplication.isPlaying = true;
        }
        else if (Application.isPlaying == true)
        {
            Close();
        }
    }
}
```

