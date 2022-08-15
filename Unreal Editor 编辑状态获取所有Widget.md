# Unreal Editor 编辑状态获取所有Widget

## 问题由来
- 需要编辑状态修改Widget
- UWidgetBlueprint引用自定义的UWidgetBlueprint
- UUserWidget的UWidgetTree变量在编辑状态为空 导致获取不到子对象

## 解决流程
- 尝试了很多方法
- 通过断点查看发现Widget的基类UObjectBase有ClassPrivate这个变量包含UUserWidget的UWidgetTree数据
- ClassPrivate可以通过GetClass()获取

## 解决方法

```c++
UPackage* Package = xxx;
UObject* Asset = Package->FindAssetInPackage();
UWidgetBlueprint* WidgetBlueprint = Cast<UWidgetBlueprint>(Asset);
TArray<UWidget*> ChildWidgets = WidgetBlueprint->GetAllSourceWidgets();

for (UWidget* ChildWidget : ChildWidgets)
{
    UWidgetBlueprintGeneratedClass* WidgetBlueprintGeneratedClass = Cast<UWidgetBlueprintGeneratedClass>(ChildWidget->GetClass());
    if (WidgetBlueprintGeneratedClass)
    {
        TArray<UWidget*> DescendantsWidgets;
        WidgetBlueprintGeneratedClass->GetWidgetTreeArchetype()->GetAllWidgets(DescendantsWidgets);
        for (UWidget* DescendantsWidget : DescendantsWidgets)
        {
        }
    }
}
```
