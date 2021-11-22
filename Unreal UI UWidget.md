## UWidget

- UPanelSlot* Slot;
  - 插槽 用来控制组件的坐标、大小、适配等功能
  - 它的实例类型由父级容器控制如：UCanvasPanelSlot、UScrollBoxSlot
  - 使用它需要转换为指定容器类型
- FText ToolTipText; 
  - 提示（用户将鼠标悬停在组件上显示的提示）
  - FGetText ToolTipTextDelegate;
  - void SetToolTipText(const FText& InToolTipText);
- UWidget* ToolTipWidget;
  - 提示（用户将鼠标悬停在组件上显示的提示）
  - FGetWidget ToolTipWidgetDelegate;
  - void SetToolTip(UWidget* Widget);
- FWidgetTransform RenderTransform; 
  - 渲染变换 （控制渲染偏移、缩放、切变（转换成平行四边形）、旋转角度）
  - void SetRenderTransform(FWidgetTransform InTransform);设置渲染变换
  - void SetRenderScale(FVector2D Scale);设置缩放
  - void SetRenderShear(FVector2D Shear);设置切变
  - float GetRenderTransformAngle() const;获取角度
  - void SetRenderTransformAngle(float Angle);设置角度
  - void SetRenderTranslation(FVector2D Translation);设置偏移
  - void UpdateRenderTransform();更新
- FVector2D RenderTransformPivot;
  - 渲染变换中心
  - void SetRenderTransformPivot(FVector2D Pivot);
- uint8 bIsVariable:1;
  - 是否为变量（该组件在蓝图中是否为变量 容器类型组件大部分默认不为变量 可以在编辑器中修改）
- uint8 bCreatedByConstructionScript:1;
  - 标记 Widget 是否是从蓝图创建的
- uint8 bIsEnabled:1; 
  - 设置组件是否可以由用户交互（点击之类）
  - FGetBool bIsEnabledDelegate; 是否启用委托
  - bool GetIsEnabled() const;获取
  - virtual void SetIsEnabled(bool bInIsEnabled);设置
- uint8 bOverride_Cursor : 1;
  - 覆盖光标（光标是否覆盖在该组件上）
  - void SetCursor(EMouseCursor::Type InCursor);设置光标
  - void ResetCursor();重置光标
- uint8 bOverrideAccessibleDefaults : 1; 
  - 覆盖可访问预设（覆盖此组件的所有默认可访问性行为和文本）
- uint8 bCanChildrenBeAccessible : 1;
  - 是否可以访问子组件（此组件的子组件是否可以显示为不同的可访问小组件）
- ESlateAccessibleBehavior AccessibleBehavior;
  - 访问行为（组件是否可访问，以及如何描述它。 如果设置为自定义，则会出现其他自定义选项）
  - FText AccessibleText;（当AccessibleBehavior设置为Custom时，这是将用于描述小部件的文本）
  - USlateAccessibleWidgetData::FGetText AccessibleTextDelegate;
- ESlateAccessibleBehavior AccessibleSummaryBehavior;
  - 访问总结行为（如何描述通过父小部件摘要显示的小部件。如果设置为自定义，将出现额外的自定义选项）
  - FText AccessibleSummaryText;
  - USlateAccessibleWidgetData::FGetText AccessibleSummaryTextDelegate;
- uint8 bHiddenInDesigner:1;
  - 是否隐藏在设计（如果小部件隐藏在设计器中，则存储设计时间标志设置）
- uint8 bExpandedInDesigner:1;
  - 是否展开设计（如果小部件在设计器内展开，则存储设计时标志设置）
- uint8 bLockedInDesigner:1;
  - 是否锁定设计（如果小部件在设计器中被锁定，则存储设计时标志设置）
- TEnumAsByte<EMouseCursor Type> Cursor;
  - 光标 (当鼠标悬停在组件上时显示的光标)
- EWidgetClipping Clipping;
  - 裁剪
  - EWidgetClipping GetClipping() const;
  - void SetClipping(EWidgetClipping InClipping);
- ESlateVisibility Visibility; 
  - 可见性（组件是否可见）
  - FGetSlateVisibility VisibilityDelegate;
  - bool IsVisible() const; 如果为Visible、HitTestInvisible、SelfHitTestInvisible返回true
  - ESlateVisibility GetVisibility() const;
  - virtual void SetVisibility(ESlateVisibility InVisibility);
  - ESlateVisibility 枚举
    - Visible 显示
    - SelfHitTestInvisible 显示 不接受点击（仅自身）
    - HitTestInvisible 显示 不接受点击（包含子组件）
    - Hidden 隐藏 占用布局空间（不常用 会导致排版不对）
    - Collapsed 隐藏 不占用布局空间
- float RenderOpacity;
  - 组件的不透明度
  - float GetRenderOpacity() const;
  - void SetRenderOpacity(float InOpacity);
- class UWidgetNavigation* Navigation;
  - 导航 (在编辑器中设置)
- EFlowDirectionPreference FlowDirectionPreference;
  - 流向方向设置
- TWeakObjectPtr<UObject> WidgetGeneratedBy;
  - 存储对负责此小部件构造的资产的引用
- TWeakObjectPtr<UClass> WidgetGeneratedByClass;
  - 存储对负责此小部件构造的类的引用
- bool IsLockedInDesigner() const
  - 此小部件是否已锁定在设计器 UI 中
- virtual void SetLockedInDesigner(bool NewLockedInDesigner)
  - bLockedInDesigner 应该锁定这个小部件
- virtual bool IsHovered() const;
  - 如果小部件当前正被指针设备悬停，则返回 true
- bool HasKeyboardFocus() const;
  - 检查此小部件当前是否具有键盘焦点
- bool HasMouseCapture() const;
  - 检查这个小部件是否是当前的鼠标捕获器
- bool HasMouseCaptureByUser(int32 UserIndex, int32 PointerIndex = -1) const;
  - 检查这个小部件是否是当前的鼠标捕获器
- void SetKeyboardFocus();
  - 将焦点设置到此小部件
- bool HasUserFocus(APlayerController* PlayerController) const;
  -  如果此小部件由特定用户关注，则返回 true
- bool HasAnyUserFocus() const;
  - 如果任何用户关注此小部件，则返回 true
- bool HasFocusedDescendants() const;
  - 如果任何用户关注任何后代小部件，则返回 true
- bool HasUserFocusedDescendants(APlayerController* PlayerController) const;
  - 如果特定用户关注任何后代小部件，则返回 true。
- void SetFocus();
  - 将焦点设置为拥有用户的此小部件
- void SetUserFocus(APlayerController* PlayerController);
  - 为特定用户设置此小部件的焦点（如果为拥有用户设置焦点，则更喜欢 SetFocus()
- void ForceLayoutPrepass();
  - 强制布局预通过
- void InvalidateLayoutAndVolatility();
  - 使布局和显示无效
- FVector2D GetDesiredSize() const;
  - 获取小部件所需的大小
- void SetAllNavigationRules(EUINavigationRule Rule, FName WidgetToFocus);
  - 设置所有方向的小部件导航规则。 这只能在小部件树中的小部件上调用
- void SetNavigationRule(EUINavigation Direction, EUINavigationRule Rule, FName WidgetToFocus);
  - 设置特定方向的小部件导航规则。 这只能在小部件树中的小部件上调用
- void SetNavigationRuleBase(EUINavigation Direction, EUINavigationRule Rule);
  - 设置特定方向的小部件导航规则。 这只能在小部件树中的小部件上调用。 这仅适用于非显式、非自定义和非自定义边界规则
- void SetNavigationRuleExplicit(EUINavigation Direction, UWidget* InWidget);
  - 设置特定方向的小部件导航规则。 这只能在小部件树中的小部件上调用。 这仅适用于显式规则
- void SetNavigationRuleCustom(EUINavigation Direction, FCustomWidgetNavigationDelegate InCustomDelegate);
  - 设置特定方向的小部件导航规则。 这只能在小部件树中的小部件上调用。 这仅适用于自定义规则
- void SetNavigationRuleCustomBoundary(EUINavigation Direction, FCustomWidgetNavigationDelegate InCustomDelegate);
  - 设置特定方向的小部件导航规则。 这只能在小部件树中的小部件上调用。 这仅适用于自定义边界规则
- class UPanelWidget* GetParent() const;
  - 获取父小部件
- virtual void RemoveFromParent();
  - 从其父小部件中删除小部件。 如果这个小部件被添加到玩家的屏幕或视口，它也会从这些容器中删除
- const FGeometry& GetCachedGeometry() const;
  - 获取缓存几何图形
- const FGeometry& GetTickSpaceGeometry() const;
- const FGeometry& GetPaintSpaceGeometry() const;
- TSharedRef<SWidget> TakeWidget();
  - 获取底层 slate 小部件
- TSharedRef<WidgetType> TakeDerivedWidget(ConstructMethodType ConstructMethod)
  - 获取底层 slate 小部件，如果不存在则构造它
- bool IsConstructed() const;
  - 是否构建
- UGameInstance* GetGameInstance() const;
  - 获取与此 UI 关联的游戏实例
  - TGameInstance* GetGameInstance() const
- virtual APlayerController* GetOwningPlayer() const;
  - 获取与此 UI 关联的播放器控制器
  - TPlayerController* GetOwningPlayer() const
- virtual ULocalPlayer* GetOwningLocalPlayer() const;
  - 获取与此 UI 关联的本地播放器
  - T* GetOwningLocalPlayer() const
- FText GetAccessibleText() const;
  - 获取可访问文本
- FText GetAccessibleSummaryText() const;
  - 获取可访问摘要文本
- virtual void SynchronizeProperties();
  - 同步属性到本地
- void BuildNavigation();
  - 构建导航
- FORCEINLINE bool IsDesignTime() const
  - 是否在设计时间状态（如果小部件当前正在设计器中显示，它可能想要显示不同的数据）
- FORCEINLINE bool HasAnyDesignerFlags(EWidgetDesignFlags FlagsToCheck) const
  - 测试此小组件上是否存在任何标志
- virtual void ValidateCompiledDefaults(class IWidgetCompilerLog& CompileLog) const {}
  - 在控件蓝图编译结束时调用
- virtual bool Modify(bool bAlwaysMarkDirty = true) override;
  - 将此对象标记为已修改，也将插槽标记为已修改
- bool IsChildOf(UWidget* PossibleParent);
  - 递归父列表，如果此小组件是可能父的后代，则返回 true
- FORCEINLINE bool CanSafelyRouteEvent()
  - 可以安全通讯事件
- FORCEINLINE bool CanSafelyRoutePaint()
  - 可以安全通讯绘画
- FORCEINLINE bool CanSafelyRouteCall() { }
  - 可以安全通讯调用
- virtual FString GetLabelMetadata() const;
  -  获取标签元数据，它可能像一些字符串数据一样简单，以帮助识别匿名文本块
- FText GetLabelText() const;
  - 获取要为此小组件显示给用户的标签
- FText GetLabelTextWithMetadata() const;
  - 获取要为此小组件显示给用户的标签，包括任何额外的元数据，如文本的文本字符串
- virtual const FText GetPaletteCategory();
  - 获取小组件的调色板类别
- virtual void OnCreationFromPalette() { }
  - 构建新的小组件后由调色板调用，允许小组件执行有趣的
- virtual const FSlateBrush* GetEditorIcon();
  - 获取编辑器图标
- virtual void ConnectEditorData() { }
  - 允许仅在编辑时使用的一般修正和连接
- bool IsVisibleInDesigner() const;
  - 小组件在设计器中是否可见？ 如果此小组件“隐藏在设计器中”或父部件是，则此小组件也将在此处返回 false。
- 设计器相关
  - void SelectByDesigner(); 设计器中选择
  - void DeselectByDesigner();设计器中取消选择
  - virtual void OnDesignerChanged(const FDesignerChangedEventArgs& EventArgs) { }
  - virtual void OnSelectedByDesigner() { }
  - virtual void OnDeselectedByDesigner() { }
  - virtual void OnDescendantSelectedByDesigner(UWidget* DescendantWidget) { }
  - virtual void OnDescendantDeselectedByDesigner(UWidget* DescendantWidget) { }
  - virtual void OnBeginEditByDesigner() { }
  - virtual void OnEndEditByDesigner() { }
- static EVisibility ConvertSerializedVisibilityToRuntime(ESlateVisibility Input);
- static ESlateVisibility ConvertRuntimeToSerializedVisibility(const EVisibility& Input);
- static FSizeParam ConvertSerializedSizeParamToRuntime(const FSlateChildSize& Input);
- static UWidget* FindChildContainingDescendant(UWidget* Root, UWidget* Descendant);
- static FString GetDefaultFontName();

## protected:

- uint8 bIsVolatile:1;
  - 是否稳定(如果为true，则阻止小部件或其子部件的几何或布局信息被缓存。如果这个小部件更改了每一帧，但您希望它仍然在失效面板中，您应该将它设置为volatile，而不是使它每一帧都失效，这将防止失效面板实际缓存任何东西)
  - void ForceVolatile(bool bForce);设置小部件的强制波动率
- TWeakPtr<SWidget> MyWidget;
  -  (底层的 SWidget)
- TWeakPtr<SObjectWidget> MyGCWidget; 
  - (SObjectWidget 中包含的底层 SWidget)
  - TSharedPtr<SWidget> GetCachedWidget() const;
- TArray<UPropertyBinding*> NativeBindings;
  - 本机属性绑定
  - bool AddBinding(FDelegateProperty* DelegateProperty, UObject* SourceObject, const FDynamicPropertyPath& BindingPath);
- static TArray<TSubclassOf<UPropertyBinding>> BinderClasses;
  - static TSubclassOf<UPropertyBinding> FindBinderClassForDestination(FProperty* Property);查找目标的 Binder 类

- EVisibility GetVisibilityInDesigner() const;
  - 设计器中显示和隐藏小组件
- virtual void OnBindingChanged(const FName& Property);
- UObject* GetSourceAssetOrClass() const;
- virtual TSharedRef<SWidget> RebuildWidget();
  - 重新构建小组件
- virtual void OnWidgetRebuilt();
  - 底层 SWidget 构建后调用的函数 
- TSharedRef<SWidget> BuildDesignTimeWidget(TSharedRef<SWidget> WrapWidget) {  }
  - 用于构建设计时包装器小组件的实用方法
- virtual TSharedRef<SWidget> RebuildDesignWidget(TSharedRef<SWidget> Content);
- TSharedRef<SWidget> CreateDesignerOutline(TSharedRef<SWidget> Content) const;
- void SynchronizeAccessibleData();
  - 将所有可访问的属性复制到 AccessibleWidgetData 对象
- EVisibility ConvertVisibility(TAttribute<ESlateVisibility> SerializedType) const
- TOptional<float> ConvertFloatToOptionalFloat(TAttribute<float> InFloat) const
- FSlateColor ConvertLinearColorToSlateColor(TAttribute<FLinearColor> InLinearColor) const
- void SetNavigationRuleInternal(EUINavigation Direction, EUINavigationRule Rule, FName WidgetToFocus = NAME_None, UWidget* InWidget = nullptr, FCustomWidgetNavigationDelegate InCustomDelegate = FCustomWidgetNavigationDelegate());
- virtual TSharedPtr<SWidget> GetAccessibleWidget() const;

## private:

- USlateAccessibleWidgetData* AccessibleWidgetData;
  - 可访问组件数据（此组件的一组自定义可访问性规则。 如果为 null，则使用小部件的默认规则）
- uint8 DesignerFlags;
  - 设计标记（设计者在编辑时使用的任何标志）
  - FORCEINLINE EWidgetDesignFlags GetDesignerFlags() const
  - virtual void SetDesignerFlags(EWidgetDesignFlags NewFlags);
- FString DisplayLabel;
  - 显示标签（在设计器和 BP 图中显示的此小部件的友好名称）
  - const FString& GetDisplayLabel() const
  - void SetDisplayLabel(const FString& DisplayLabel);
  - bool IsGeneratedName() const;
  - FText GetDisplayNameBase() const;
- TWeakPtr<SWidget> DesignWrapperWidget;
  - 设计包装组件（设计时包装器小部件的底层 SWidget）
- FString CategoryName;
  - 类别名称（小部件设计器中用于排序的类别名称） 
  - const FString& GetCategoryName() const;
  - void SetCategoryName(const FString& InValue);
- bool bRoutedSynchronizeProperties;
  - 我们是否路由了同步属性调用？
- TSharedRef<SWidget> TakeWidget_Private( ConstructMethodType ConstructMethod );
- void VerifySynchronizeProperties();
- FORCEINLINE void VerifySynchronizeProperties() { }