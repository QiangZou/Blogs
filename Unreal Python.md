# Unreal Python 菜单扩展

- 使用Python脚本化运行虚幻编辑器 https://dev.epicgames.com/documentation/zh-cn/unreal-engine/scripting-the-unreal-editor-using-python

- API https://dev.epicgames.com/documentation/en-us/unreal-engine/PythonAPI

- 生成unreal.py脚本

  - Project Settings -> Plugings -> python -> Developer Mode(all users)  勾选

- vscode 添加unreal.py脚本

  - vscode打开setting.json配置 添加

  - ```json
    "python.analysis.extraPaths": [
            "XXX\\Intermediate\\PythonStub"
        ],
    ```

- 显示UI扩展名

  -  **Editor Preferences**->**Display UI Extension Points** 勾选

- 使用Python脚本注册菜单

  - ```python
    import unreal
    
    
    def RegisterMenus(name: str, command: str, tool_tip: str = ""):
        tool_menus = unreal.ToolMenus.get()
    
        # 先尝试移除已经存在的菜单项，避免重复调用添加
        tool_menus.remove_entry(unreal.Name("LevelEditor.MainMenu.Python"), unreal.Name(name), unreal.Name(name))
    
        tool_menu = tool_menus.find_menu(unreal.Name("LevelEditor.MainMenu"))
        main_menu = tool_menu.add_sub_menu(unreal.Name("LevelEditor.MainMenu"), unreal.Name("Python"), unreal.Name("Python"), unreal.Name("Python"), unreal.Name("Python Menu"))
    
        entry = unreal.ToolMenuEntry(name=unreal.Name(name), type=unreal.MultiBlockType.MENU_ENTRY)
        entry.set_label(unreal.Text(name))
        entry.set_tool_tip(unreal.Text(tool_tip))
        entry.set_string_command(unreal.ToolMenuStringCommandType.COMMAND, unreal.Name(""), command)
    
        main_menu.add_menu_entry(unreal.Name(name), entry)
    
        tool_menus.refresh_all_widgets()
    
    
    RegisterMenus("Clear Log", "py clear_log.py", "Clear the log")
    ```

## 终结

- 生成的unreal.py脚本有80万行 导致代码提示 跳转很不好用
- 同一个类型 很多C++有的 python不提供
- 功能缺失严重 稍微复杂一点的功能需要在C++提供接口
- 菜单扩展这种功能 还不如直接用插件方式