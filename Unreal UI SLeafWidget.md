# SLeafWidget 

- 是一种没有子插槽的 Widget

- SLeafWidget()
- virtual void SetVisibility( TAttribute<EVisibility> InVisibility ) override final;
- virtual int32 OnPaint()提供自己的视觉表示
- virtual FVector2D ComputeDesiredSize()应仅根据其视觉表示计算其
- virtual FChildren* GetChildren() 
- virtual void OnArrangeChildren()
- static FNoChildren NoChildrenInstance;所有没有孩子的小部件的 FNoChildren 共享实例。

