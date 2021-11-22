# Unreal Engine 实现英雄联盟手游 备注名文本超框自动截断

## 效果如图

![](https://github.com/QiangZou/Blogs/blob/master/Images/image-20211111220249266.png?raw=true)

## 需求由来

- 策划要求

## 实现流程

- UTextBlock 新增一个超框截断功能开关
- 获取TextBlock的Size X长度
- 获取STextBlock文本渲染需要长度
- 判断文本长度是否超过设计长度
- 循环减少字符直至满足设计长度

## 关键代码(这里只展示修改源码情况 也可以使用继承 复写)

- UTextBlock.h

  ```
  public:
     /** 超框自动截断 填补"..." */
     UPROPERTY(EditAnywhere, BlueprintReadOnly, Category=Appearance, AdvancedDisplay)
     bool bAutoCut;
     
  protected:
     virtual TSharedRef<SWidget> RebuildWidget() override;
  ```

- UTextBlock.cpp

  ```
  TSharedRef<SWidget> UTextBlock::RebuildWidget()
  {
     if (bWrapWithInvalidationPanel && !IsDesignTime())
     {
        TSharedPtr<SWidget> RetWidget = SNew(SInvalidationPanel)
        [
           SAssignNew(MyTextBlock, STextBlock)
           .SimpleTextMode(bSimpleTextMode)
        ];
        return RetWidget.ToSharedRef();
     }
     else
     {
        MyTextBlock =
           SNew(STextBlock)
           .SimpleTextMode(bSimpleTextMode);
  
        MyTextBlock->SetAutoCut(bAutoCut);
  
        return MyTextBlock.ToSharedRef();
     }
  }
  ```

- STextBlock.h

  ```
  private:
      /** 超框自动截断 填补"..." */
      bool bAutoCut = false;
  public:
  	void SetAutoCut(bool AutoCut);
  ```

- STextBlock.cpp

  ```
  void STextBlock::SetAutoCut(bool AutoCut)
  {
     bAutoCut = AutoCut;
  }
  FVector2D STextBlock::ComputeDesiredSize(float LayoutScaleMultiplier) const
  {
  	SCOPE_CYCLE_COUNTER(Stat_SlateTextBlockCDS);
  
  	if (bSimpleTextMode)
  	{
  		const FVector2D LocalShadowOffset = GetShadowOffset();
  
  		const float LocalOutlineSize = GetFont().OutlineSettings.OutlineSize;
  
  		// Account for the outline width impacting both size of the text by multiplying by 2
  		// Outline size in Y is accounted for in MaxHeight calculation in Measure()
  		const FVector2D ComputedOutlineSize(LocalOutlineSize * 2, LocalOutlineSize);
  		const FVector2D TextSize = FSlateApplication::Get().GetRenderer()->GetFontMeasureService()->Measure(GetText(), GetFont()) + ComputedOutlineSize + LocalShadowOffset;
  
  		CachedSimpleDesiredSize = FVector2D(FMath::Max(MinDesiredWidth.Get(0.0f), TextSize.X), TextSize.Y);
  		return CachedSimpleDesiredSize.GetValue();
  	}
  	else
  	{
  		if (bAutoCut != true)
  		{
  			// ComputeDesiredSize will also update the text layout cache if required
  			const FVector2D TextSize = TextLayoutCache->ComputeDesiredSize(
  				FSlateTextBlockLayout::FWidgetArgs(BoundText, HighlightText, WrapTextAt, AutoWrapText, WrappingPolicy, TransformPolicy, Margin, LineHeightPercentage, Justification),
  				LayoutScaleMultiplier, GetComputedTextStyle()
  			);
  
  			return FVector2D(FMath::Max(MinDesiredWidth.Get(0.0f), TextSize.X), TextSize.Y);
  		}
  		else
  		{
  			TAttribute<FText> DisplayText = BoundText;
  			const FString BoundString = BoundText.Get(FText::GetEmpty()).ToString();
  			int Lenght = 0;
  			do
  			{
  				const FVector2D TextSize = TextLayoutCache->ComputeDesiredSize(
  					FSlateTextBlockLayout::FWidgetArgs(DisplayText, HighlightText, WrapTextAt, AutoWrapText, WrappingPolicy, TransformPolicy, Margin, LineHeightPercentage, Justification),
  					LayoutScaleMultiplier, GetComputedTextStyle()
  				);
  				
  				const FVector2D DisplayTextSize = FVector2D(FMath::Max(MinDesiredWidth.Get(0.0f), TextSize.X), TextSize.Y);
  				//容器宽度大于文本显示宽度
  				if (GetTickSpaceGeometry().GetLocalSize().X >= DisplayTextSize.X)
  				{
  					return DisplayTextSize;
  				}
  
  				Lenght++;
  
  				//处理截取到最后一个字符 还超框的情况
  				if (Lenght == BoundString.Len())
  				{
  					return DisplayTextSize;
  				}
  		
  				FString NewBoundString = BoundString.Left(BoundString.Len() - Lenght) + "...";
  		
  				FText NewText = FText::FromString(NewBoundString);
  		
  				DisplayText = TAttribute<FText>::Create(TAttribute<FText>::FGetter::CreateLambda(
  					[NewText]()
  					{
  						return NewText;
  					}));
  			}
  			while (true);
  		}
  	}
  }
  ```