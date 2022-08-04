# Unreal 本地化

- 官网文档：[https://docs.unrealengine.com/4.27/zh-CN/ProductionPipelines/Localization/Overview/](https://docs.unrealengine.com/4.27/zh-CN/ProductionPipelines/Localization/Overview/)
- 国际化 internationalization 缩写“I18N”，18为首尾字母中间的字符数
- 本地化 localization 缩写“L10N”，10 为首尾字母中间的字符数
- 在Unreal4.26.2环境下实现

## 那类文本能被收集
- 蓝图中的变量 需要是FText文本类型（FString字符串类型不行）
- C++代码中使用宏包含的
  - NSLOCTEXT(InNamespace, InKey, InTextLiteral)
  - LOCTEXT(InKey, InTextLiteral) 
  - 建议使用 NSLOCTEX InNamespace用通用的
- 如果项目中使用Lua 也可以使用LOCTEXT函数包住文本
  - 写个收集LOCTEXT中的文本自动化脚本 处理成csv文件再导入工程为StringTable类型

## 创建本地化语言并收集文本流程

- 开启本地化工具 
  - **编辑->编辑器偏好设置**（Unreal4.26 默认是开启的 如果第一次打开需要重启Unreal）
- 打开本地化窗口 
  - **窗口->本地化控制板**
- 设置收集文本目录
  - 勾选从文本文件收集 添加目录Source
  - 勾选从包收集 添加目录Content
- 添加新语言并设置默认语言
- 收集文本
- 导出文本 生成po文件（建议使用Content/Localization/Game目录 方便版本控制和自动导入）
- 导入文本 导入po文件
- 编译文本 生成二进制文件运行时使用


## 生成本地化文件解析

- 执行收集、导出文本、编译文本后会在Content下生成目录
- Localization 引擎自动创建
  - Game 本地化目标命名（默认是Game 也可以新增  但是没必要）
    - en 英文语言标签
      - Game.archive JSON文件引擎使用不要手动修改
      - Game.locres 二进制文件Unreal运行时加载用的
      - Game.po 通用翻译文件 给翻译厂商翻译使用
    - zh-Hans 中文简体语言标签
    - Game.csv 
    - Game.locmeta
    - Game.manifest
    - Game_Conflicts.txt 冲突文件 （一般是拷贝蓝图资源拷贝导致蓝图中的文本翻译Key相同 找到冲突位置 重新生成Key就可以）

## 资源本地化
- 贴图、字体等资源如果需要按地区展示不同的效果
- 在内容浏览器 选中资源 右键 -> 资产本地化 -> 新建本地化资产 -> 选中对应的地区
- 会在内容浏览器 L10N/地区代码 相同目录生成一个对应资源 修改该资源即可
- 建议图集中如果有带中文的图片 单拎出来 作为Texture使用 创建本地化资源


## 如何预览多语言
- 前提条件 需要导入翻译好的PO文件 执行编译文本
- UMG预览在编辑窗口 设计视图右上角选择对应地区
- 直接启动预览 编辑 -> 编辑器偏好设置 -> 区域和语言 -> 预览游戏语言
- 完整预览 使用 **独立进程游戏** 模式 (可以预览资源本地化)
  - 运行按钮下拉菜单 高级设置 额外启动参数设置为 -culture=en 即可预览英文效果

## API
- UKismetInternationalizationLibrary::GetCurrentCulture 获取当前文化
- FInternationalization::SetCurrentCulture 设置当前文化

