# Unity 查看Unity所有的默认UI效果 GUIStyle

## 效果图

![](E:\GitHub\Blogs\Unity 查看Unity所有的默认UI效果 GUIStyle\TIM截图20200515143703.png)

## 需求由来

- 因为编辑器需要做一个搜索的功能
- 想到搜索框怎么用Unity自带的UI效果实现
- 最后查到 输入框GUIStyle "ToolbarSeachTextField" 删除按钮 GUIStyle "ToolbarSeachCancelButton"
- 然后又查看可以获取所有的GUIStyle API
- 最后想到做个显示所有GUIStyle的窗口 方便以后查看

## 使用方式

- 左上角菜单栏 ZQFramwork -> 工具 -> 查看所有GUIStyle

## 源码

```c#
using UnityEngine;
using UnityEditor;

namespace ZQFramwork
{
    public class ShowAllGUIStyle : EditorWindow
    {
        private Vector2 scrollVector2 = Vector2.zero;

        [MenuItem("ZQFramwork/工具/查看所有GUIStyle", false)]
        static void OpenWindow()
        {
            EditorWindow window = GetWindow(typeof(ShowAllGUIStyle));
            window.minSize = new Vector2(300, 900);
        }

        string search = string.Empty;

        void OnGUI()
        {
            EditorGUILayout.Space();

            EditorGUILayout.BeginHorizontal();

            search = EditorGUILayout.TextField("", search, "ToolbarSeachTextField");

            if (GUILayout.Button("", "ToolbarSeachCancelButton"))
            {
                search = string.Empty;
            }

            EditorGUILayout.EndHorizontal();

            EditorGUILayout.Space();

            scrollVector2 = GUILayout.BeginScrollView(scrollVector2);

            foreach (GUIStyle style in GUI.skin.customStyles)
            {
                if (search == string.Empty || style.name.Contains(search))
                {
                    DrawStyleItem(style);
                }

            }

            GUILayout.EndScrollView();
        }

        void DrawStyleItem(GUIStyle style)
        {
            EditorGUILayout.BeginVertical("box");

            EditorGUILayout.SelectableLabel(style.name);

            GUILayout.Button("", style);

            EditorGUILayout.EndVertical();
        }
    }
}
```

