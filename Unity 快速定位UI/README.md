# Unity 快速定位UI

## 问题由来

- 当项目UI层级特别多的时候

- 想找快速定位UI的位置非常麻烦 

## 使用方式

- 运行状态下

- 鼠标移动到指定UI位置

- 快捷键Ctrl+F

## 源码

```c#
using System;
using System.Collections.Generic;
using UnityEditor;
using UnityEngine;
using UnityEngine.EventSystems;

public class QuickPositioningUITool : Editor
{
    [MenuItem("ZQFramwork/快速定位UI %f", false, 0)]
    public static void QuickPositioning()
    {
        if (Application.isPlaying == false)
        {
            return;
        }

        //使焦点移动到Game视图
        Type gameViewType = typeof(Editor).Assembly.GetType("UnityEditor.GameView");
        EditorWindow window = EditorWindow.GetWindow(gameViewType);
        window.Focus();


        PointerEventData pointerEventData = new PointerEventData(EventSystem.current)
        {
            position = Input.mousePosition
        };

        List<RaycastResult> raycastResults = new List<RaycastResult>();

        //获取鼠标位置所有碰撞对象
        EventSystem.current.RaycastAll(pointerEventData, raycastResults);

        if (raycastResults.Count > 0)
        {
            //选择第一个对象
            Selection.activeGameObject = raycastResults[0].gameObject;

            EditorGUIUtility.PingObject(raycastResults[0].gameObject);
        }
    }
}

```

