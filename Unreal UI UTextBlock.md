# UTextBlock 组件 相关类

## UTextLayoutWidget 文本布局组件基类

- ShapedTextOptions
  - 控制文本阅读方向
  - 控制文本显示方向
- Justification
  - 设置文本对齐方式 左对齐、居中对齐、右对齐
- AutoWrapText
  - 开启自动换行
- WrapTextAt
  - 设置文本换行宽度
  - 如果设置为0或负值则不换行
  - 一般不用这个参数 开启AutoWrapText
- WrappingPolicy
  - 换行策略 
  - DefaultWrapping 单词超过行宽不换行
  - AllowPerCharacterWrapping 单词超过行宽拆分单词
- Margin
  - 文本区域边缘留下的空白空间量
- LineHeightPercentage
  - 每行高度的缩放量

## UTextBlock 文本块

- Text 显示的文本
  - TextDelegate
  - GetText
  - SetText
  - GetDisplayText

- ColorAndOpacity 颜色和透明 （和画笔设置独立）

  - ColorAndOpacityDelegate 修改委托
  - SetColorAndOpacity
  - SetOpacity 只设置透明
- Font 字体
  - SetFont
  - GetDynamicFontMaterial 获取动态字体材质
  - GetDynamicOutlineMaterial 获取动态描边材质
- StrikeBrush 文本上的删除线
  - SetStrikeBrush
- ShadowOffset 阴影投射的方向
  - SetShadowOffset
- ShadowColorAndOpacity 阴影的颜色
  - ShadowColorAndOpacityDelegate 修改委托
  - SetShadowColorAndOpacity
- MinDesiredWidth 文本最小尺寸
  - 当勾选Size To Content时防止文本内容太少导致排版问题
- bWrapWithInvalidationPanel
  -  如果为 true，它将使用失效面板自动包装此文本小部件
- TextTransformPolicy 文本策略全大写、全小写
  - SetTextTransformPolicy

## STextBlock

- BoundText 此文本块中显示的文本
  - SetText
  - GetText
- TextLayoutCache 此文本块的包装布局
  - OnPaint 绘制
  - ComputeDesiredSize 计算所需大小
  - SetTextShapingMethod 设置文本阅读方向
  - SetTextFlowDirection 设置文本流方向
- TextStyle 默认文本样式 字体 颜色 阴影啥的
- Font 字体
  - GetFont
  - SetFont
- StrikeBrush 删除线
  - GetStrikeBrush
  - SetStrikeBrush
  - GetComputedTextStyle
- ColorAndOpacity 颜色和透明
  - GetColorAndOpacity
  - SetColorAndOpacity
- ShadowOffset 阴影投射的方向
  - GetShadowOffset
  - SetShadowOffset
- ShadowColorAndOpacity 阴影的颜色
  - GetShadowColorAndOpacity
  - SetShadowColorAndOpacity
- HighlightColor 高亮颜色
  - GetHighlightColor
- HighlightShape  高亮形状
  - GetHighlightColor
- HighlightText 高亮文本
  - SetHighlightText
  - ComputeDesiredSize
- WrapTextAt 大于0时 超过此宽度换行
  - SetHighlightText
  - ComputeDesiredSize
- AutoWrapText 文本自动换行
  - OnPaint
  - ComputeDesiredSize
  - SetAutoWrapText
- WrappingPolicy 换行策略 
  - ComputeDesiredSize
  - SetWrappingPolicy
- TransformPolicy 文本策略全大写、全小写
  - ComputeDesiredSize
  - SetTransformPolicy
- Margin 文本空隙
  - ComputeDesiredSize
  - SetMargin
- Justification 设置文本对齐方式 左对齐、居中对齐、右对齐
  - SetJustification
- LineHeightPercentage 每行高度的缩放量
  - ComputeDesiredSize
  - SetLineHeightPercentage
- MinDesiredWidth 最小想要的宽度 对齐用
  - ComputeDesiredSize
  - SetMinDesiredWidth
- bSimpleTextMode 简单文本模式 更快的文本对齐测量 适用于全部为纯ascll表中的字符
  - InvalidateText
  - OnPaint
  - ComputeDesiredSize
  - SetTextShapingMethod
  - SetTextFlowDirection
- CachedSimpleDesiredSize 缓存简单模式文本期望大小
  - InvalidateText
  - ComputeDesiredSize

  
