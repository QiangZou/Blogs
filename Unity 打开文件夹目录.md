# Unity 打开文件夹目录 

## 解决方案

- 调用API：EditorUtility.RevealInFinder(“路径”);

## 问题由来

- 最近在写一个编辑器 需要这个功能

## 解决流程

- 搜索：Unity 打开文件夹目录

- 看了几个方案都是window平台用cmd Mac平台用Shell

- 测试都还可以  但打开目录后不会选择指定文件

- 想到Unity右键文件菜单Show in Explorer选项就换了搜索关键字

- 搜索：Unity Show in Explorer

- 然后就找到了上面的API

  要多看看API啊啊啊啊啊啊

