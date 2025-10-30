# Unreal 细节窗口 自定义属性 扩展按钮 修改属性 Details Window IDetailCustomization PropertyEditor

## 需求

- UI设计有一套规范 例如
  - 标题：粗体 32 大小
  - 内容：细体 28 大小
  - 按钮：粗体 30 大小
  - 金黄：\#FFD700
  - 褐色：\#D2B48C
- 还原UI效果的时候经常需要设置这些数据
- 为了提高效率和规范风格 
- 所以打算扩展文本组件细节窗口 点击按钮就可以设置 并且这些需要可配置化

## 实现

- 使用插件扩展 不需要修改源码和组件

- XXX.uplugin 插件需要设置 方便存放配置DataTabel

  - ```
    "CanContainContent": true,
    ```

- 定义配置结构体

  - ```c++
    #pragma once
    
    #include "CoreMinimal.h"
    
    #include "CustomStyle.generated.h"
    
    USTRUCT(BlueprintType)
    struct FCustomColor: public FTableRowBase
    {
        GENERATED_BODY()
        UPROPERTY(EditAnywhere)
        FString Name;
        
        UPROPERTY(EditAnywhere)
        FSlateColor Color;
    };
    
    USTRUCT(BlueprintType)
    struct FCustomFont: public FTableRowBase
    {
        GENERATED_BODY()
        UPROPERTY(EditAnywhere)
        FString Name;
        
        UPROPERTY(EditAnywhere)
        FSlateFontInfo Font;
    };
    ```

- 插件启动时注册扩展类型

  - ```c++
    void FMyAssetToolsModule::StartupModule()
    {
        FPropertyEditorModule& PropertyModule = FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");
        PropertyModule.RegisterCustomClassLayout(UTextLayoutWidget::StaticClass()->GetFName(), FOnGetDetailCustomizationInstance::CreateStatic(&FTextLayoutWidgetDetailCustomization::MakeInstance));
    }
    
    void FMyAssetToolsModule::ShutdownModule()
    {
        if (UObjectInitialized() && FModuleManager::Get().IsModuleLoaded("PropertyEditor"))
        {
           FPropertyEditorModule& PropertyModule = FModuleManager::LoadModuleChecked<FPropertyEditorModule>("PropertyEditor");
           PropertyModule.UnregisterCustomClassLayout(UTextLayoutWidget::StaticClass()->GetFName());
        }
    }
    ```

- 具体的实现逻辑

  - FTextLayoutWidgetDetailCustomization.h

  - ```C++
    #pragma once
    
    #include "CustomStyle.h"
    #include "IDetailCustomization.h"
    
    
    /**
     * 
     */
    class FTextLayoutWidgetDetailCustomization : public IDetailCustomization
    {
    public:
        static TSharedRef<IDetailCustomization> MakeInstance();
    
        virtual void CustomizeDetails(IDetailLayoutBuilder& DetailBuilder) override;
    private:
    
        void AddSyncBrowserToObjectsButton(const TSharedPtr<SHorizontalBox>& HorizontalBox, const FAssetData& AssetData) const;
        
        bool IsCustomColorEnabled(const FCustomColor Value) const;
        FReply OnSetColorAndOpacity(const FCustomColor Value) const;
    
        bool IsCustomFontEnabled(const FCustomFont Value) const;
        FReply OnSetFont(const FCustomFont Value) const;
        static void OnSetFontPropertyHandle(const TSharedRef<IPropertyHandle>& PropertyHandle,const FCustomFont Value);
        
        IDetailLayoutBuilder* SavedLayoutBuilder;
    };
    ```

  - FTextLayoutWidgetDetailCustomization.cpp

  - ```c++
    #include "FTextLayoutWidgetDetailCustomization.h"
    #include "CustomStyle.h"
    #include "DetailCategoryBuilder.h"
    #include "DetailLayoutBuilder.h"
    #include "DetailWidgetRow.h"
    #include "Components/MultiLineEditableText.h"
    #include "Components/MultiLineEditableTextBox.h"
    #include "Components/RichTextBlock.h"
    #include "Components/TextBlock.h"
    
    
    TSharedRef<IDetailCustomization> FTextLayoutWidgetDetailCustomization::MakeInstance()
    {
        return MakeShareable(new FTextLayoutWidgetDetailCustomization());
    }
    
    void FTextLayoutWidgetDetailCustomization::CustomizeDetails(IDetailLayoutBuilder& DetailBuilder)
    {
        SavedLayoutBuilder = &DetailBuilder;
    
    
        const TSharedPtr<SHorizontalBox> HorizontalBoxCustomColor = SNew(SHorizontalBox);
    
        const UDataTable* DataTableCustomColor = LoadObject<UDataTable>(nullptr,TEXT("/MyAssetTools/CustomColor.CustomColor"));
        DataTableCustomColor->ForeachRow<FCustomColor>("FCustomColor", [&](const FName& Key, const FCustomColor& Value)
        {
           HorizontalBoxCustomColor->AddSlot()
                                   .AutoWidth()
                                   .HAlign(HAlign_Fill)
           [
              SNew(SButton)
              .Text(FText::FromString(Value.Name))
              .IsEnabled(this, &FTextLayoutWidgetDetailCustomization::IsCustomColorEnabled, Value)
              .OnClicked(this, &FTextLayoutWidgetDetailCustomization::OnSetColorAndOpacity, Value)
           ];
        });
        AddSyncBrowserToObjectsButton(HorizontalBoxCustomColor, FAssetData(DataTableCustomColor));
    
        const TSharedPtr<SHorizontalBox> HorizontalBoxCustomFont = SNew(SHorizontalBox);
        const UDataTable* DataTableCustomFont = LoadObject<UDataTable>(nullptr,TEXT("/MyAssetTools/CustomFont.CustomFont"));
        DataTableCustomFont->ForeachRow<FCustomFont>("FCustomFont", [&](const FName& Key, const FCustomFont& Value)
        {
           HorizontalBoxCustomFont->AddSlot()
                                  .AutoWidth()
                                  .HAlign(HAlign_Fill)
           [
              SNew(SButton)
              .Text(FText::FromString(Value.Name))
              .IsEnabled(this, &FTextLayoutWidgetDetailCustomization::IsCustomFontEnabled, Value)
              .OnClicked(this, &FTextLayoutWidgetDetailCustomization::OnSetFont, Value)
           ];
        });
        AddSyncBrowserToObjectsButton(HorizontalBoxCustomFont, FAssetData(DataTableCustomFont));
    
        auto& DefaultCategory = DetailBuilder.EditCategory(TEXT("Appearance"));
        DefaultCategory.AddCustomRow(INVTEXT("CustomColor"))
                       .NameContent()
           [
              SNew(STextBlock)
              .Text(INVTEXT("Custom Color"))
              .Font(FCoreStyle::GetDefaultFontStyle("Regular", 8))
           ]
           .ValueContent()
           [
              HorizontalBoxCustomColor.ToSharedRef()
           ];
        DefaultCategory.AddCustomRow(INVTEXT("CustomFont"))
                       .NameContent()
           [
              SNew(STextBlock)
              .Text(INVTEXT("Custom Font"))
              .Font(FCoreStyle::GetDefaultFontStyle("Regular", 8))
           ]
           .ValueContent()
           [
              HorizontalBoxCustomFont.ToSharedRef()
           ];
    }
    
    void FTextLayoutWidgetDetailCustomization::AddSyncBrowserToObjectsButton(const TSharedPtr<SHorizontalBox>& HorizontalBox, const FAssetData& AssetData) const
    {
        HorizontalBox->AddSlot()
                     .AutoWidth()
                     .HAlign(HAlign_Fill)
                     .VAlign(VAlign_Center)
        [
           SNew(SButton)
           .ButtonStyle(FAppStyle::Get(), "SimpleButton")
           .OnClicked_Lambda([=, this]
           {
              TArray<FAssetData> Objects;
              Objects.Add(AssetData);
    
              GEditor->SyncBrowserToObjects(Objects);
              return FReply::Handled();
           })
           [
              SNew(SImage)
              .Image(FAppStyle::GetBrush("Icons.BrowseContent"))
              .ColorAndOpacity(FSlateColor::UseForeground())
           ]
        ];
    }
    
    bool FTextLayoutWidgetDetailCustomization::IsCustomColorEnabled(const FCustomColor Value) const
    {
        TArray<TWeakObjectPtr<UObject>> ObjectsBeingCustomized;
        SavedLayoutBuilder->GetObjectsBeingCustomized(ObjectsBeingCustomized);
        for (const TWeakObjectPtr<UObject>& Object : ObjectsBeingCustomized)
        {
           if (const UTextBlock* CustomObject = Cast<UTextBlock>(Object.Get()))
           {
              if (CustomObject->GetColorAndOpacity() != Value.Color)
              {
                 return true;
              }
           }
           if (const URichTextBlock* CustomObject = Cast<URichTextBlock>(Object.Get()))
           {
              if (CustomObject->GetDefaultTextStyle().ColorAndOpacity != Value.Color)
              {
                 return true;
              }
           }
           if (const UMultiLineEditableText* CustomObject = Cast<UMultiLineEditableText>(Object.Get()))
           {
              if (CustomObject->WidgetStyle.ColorAndOpacity != Value.Color)
              {
                 return true;
              }
           }
           if (const UMultiLineEditableTextBox* CustomObject = Cast<UMultiLineEditableTextBox>(Object.Get()))
           {
              if (CustomObject->WidgetStyle.TextStyle.ColorAndOpacity != Value.Color)
              {
                 return true;
              }
           }
        }
        return false;
    }
    
    FReply FTextLayoutWidgetDetailCustomization::OnSetColorAndOpacity(const FCustomColor Value) const
    {
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("ColorAndOpacity", UTextBlock::StaticClass()); PropertyHandle->IsValidHandle())
        {
           PropertyHandle->SetValueFromFormattedString(Value.Color.GetSpecifiedColor().ToString());
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("bOverrideDefaultStyle", URichTextBlock::StaticClass()); PropertyHandle->IsValidHandle())
        {
           PropertyHandle->SetValue(true);
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("DefaultTextStyleOverride.ColorAndOpacity", URichTextBlock::StaticClass()); PropertyHandle->IsValidHandle())
        {
           PropertyHandle->SetValueFromFormattedString(Value.Color.GetSpecifiedColor().ToString());
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("WidgetStyle.ColorAndOpacity", UMultiLineEditableText::StaticClass()); PropertyHandle->IsValidHandle())
        {
           PropertyHandle->SetValueFromFormattedString(Value.Color.GetSpecifiedColor().ToString());
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("WidgetStyle.TextStyle.ColorAndOpacity", UMultiLineEditableTextBox::StaticClass()); PropertyHandle->IsValidHandle())
        {
           PropertyHandle->SetValueFromFormattedString(Value.Color.GetSpecifiedColor().ToString());
        }
        return FReply::Handled();
    }
    
    bool FTextLayoutWidgetDetailCustomization::IsCustomFontEnabled(const FCustomFont Value) const
    {
        TArray<TWeakObjectPtr<UObject>> ObjectsBeingCustomized;
        SavedLayoutBuilder->GetObjectsBeingCustomized(ObjectsBeingCustomized);
        for (const TWeakObjectPtr<UObject>& Object : ObjectsBeingCustomized)
        {
           if (const UTextBlock* CustomObject = Cast<UTextBlock>(Object.Get()))
           {
              if (CustomObject->GetFont().FontObject != Value.Font.FontObject || CustomObject->GetFont().Size != Value.Font.Size)
              {
                 return true;
              }
           }
           if (const URichTextBlock* CustomObject = Cast<URichTextBlock>(Object.Get()))
           {
              if (CustomObject->GetDefaultTextStyle().Font.FontObject != Value.Font.FontObject || CustomObject->GetDefaultTextStyle().Font.Size != Value.Font.Size)
              {
                 return true;
              }
           }
           if (const UMultiLineEditableText* CustomObject = Cast<UMultiLineEditableText>(Object.Get()))
           {
              if (CustomObject->WidgetStyle.Font.FontObject != Value.Font.FontObject || CustomObject->WidgetStyle.Font.Size != Value.Font.Size)
              {
                 return true;
              }
           }
           if (const UMultiLineEditableTextBox* CustomObject = Cast<UMultiLineEditableTextBox>(Object.Get()))
           {
              if (CustomObject->WidgetStyle.TextStyle.Font != Value.Font || CustomObject->WidgetStyle.TextStyle.Font.Size != Value.Font.Size)
              {
                 return true;
              }
           }
        }
        return false;
    }
    
    FReply FTextLayoutWidgetDetailCustomization::OnSetFont(const FCustomFont Value) const
    {
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("Font", UTextBlock::StaticClass()); PropertyHandle->IsValidHandle())
        {
           OnSetFontPropertyHandle(PropertyHandle, Value);
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("DefaultTextStyleOverride.Font", URichTextBlock::StaticClass()); PropertyHandle->IsValidHandle())
        {
           OnSetFontPropertyHandle(PropertyHandle, Value);
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("WidgetStyle.Font", UMultiLineEditableText::StaticClass()); PropertyHandle->IsValidHandle())
        {
           OnSetFontPropertyHandle(PropertyHandle, Value);
        }
        if (const TSharedRef<IPropertyHandle> PropertyHandle = SavedLayoutBuilder->GetProperty("WidgetStyle.TextStyle.Font", UMultiLineEditableTextBox::StaticClass()); PropertyHandle->IsValidHandle())
        {
           OnSetFontPropertyHandle(PropertyHandle, Value);
        }
        return FReply::Handled();
    }
    
    void FTextLayoutWidgetDetailCustomization::OnSetFontPropertyHandle(const TSharedRef<IPropertyHandle>& PropertyHandle, const FCustomFont Value)
    {
        if (const TSharedPtr<IPropertyHandle> ChildPropertyHandle = PropertyHandle->GetChildHandle(GET_MEMBER_NAME_CHECKED(FSlateFontInfo, FontObject)))
        {
           ChildPropertyHandle->SetValue(Value.Font.FontObject);
        }
        if (const TSharedPtr<IPropertyHandle> ChildPropertyHandle = PropertyHandle->GetChildHandle(GET_MEMBER_NAME_CHECKED(FSlateFontInfo, Size)))
        {
           ChildPropertyHandle->SetValue(Value.Font.Size);
        }
    }
    ```