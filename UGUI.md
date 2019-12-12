#Canvas 画布
- Render Modes 渲染模式)
 - Screen Space - Overlay
   - UI将永远出现在所有摄像机的最前面
 - Screen Space - Camera **常用**
   - 允许UI界面前可以显示其他游戏对象，如为了增加特效而添加的粒子系统
   - 摄像机与UI之间的距离通过Plane Distance来进行设置
 - World Space
   - 将UI当3D对象来对待
- Pixel Prefect 像素完美点对点 让我们的像素和平面像素对应
- Sort Order 排序次序 一般情况下无需修改

#Canvas Scaler 画布缩放

- UIScaleMode UI缩放模式
 - Constant Pixel Size 固定像素尺寸
 - Scale With Screen Size 按照屏幕大小自适应 **常用**
   - Reference Resolution 相对分辨率
   - Screen Match Mode 屏幕匹配模式
     - Match Width Or Height 匹配宽度或高度 **常用**
 - Constant Physical Size 固定物理尺寸



#EventSystem UI的事件系统
- 控制UI界面总体的事件管理器，分别表示UI事件系统，输入模块系统，触摸输入系统

#RectTransform 矩形
- Pivot 中心轴
 - X,Y值范围是0到1的点
 - (0,0)表示左下角(1,1)表示右上角
- Anchor 锚点(锚框更合适)
 - X,Y值范围是0到1的点
 - min(X,Y)表示左下角锚点 Max(X,Y)表示右上角锚点
- Anchored Position 锚定位置
 - 小技巧按住Shift选择




#Text 文本
- Font 字体
- Font Style 字体类型
 - Normal 正常的
 - Bold 加粗
 - Italic 倾斜
 - Bold And Italic 加粗和倾斜
- Font Size 字体大小
- Line Spacing 多行状态的行距
- Alignment 对其规则
- Align By Geometry
- Horizontal Overflow 水平溢出
 - Wrap 截断
 - Overflow 溢出
- Horizontal Overflow 垂直溢出
 - Truncate 截断
 - Overflow 溢出
- Best Fit 最好的适应
- Color 字的颜色
- Material 材质
- Raycast Target 射线检测
 - True 一般按钮需要

#Image 图片
- Source Image 源图像
- Color 颜色
- Material 材质
- Raycast Target 射线检测
- Image Type 图片类型
 - Simple 简单
 - Sliced 切片
 - Tiled 平铺
 - Filled 填充

#Button 按钮
- Interactable 是否启用
- Transition 过渡
 - None 无
 - Color Tint 用颜色填
 - Sprite Swap 精灵交换
 - Animation 动画
   - Target Graphic 目标图像
   - Normal 正常
   - Highlighted 突出(鼠标经过)
   - Pressed 压(按下状态)
   - Disabled 失去能力(禁用的时候)
   - Color Multiplier 颜色倍数
   - Fade Duration 变化过程时间
- Navigation 导航(使用场景 多个按钮 方向键操作 点击第一个的时候会保持选择状态 通过方向键可以切换到其他按钮上)
 - None 无
 - Automatic 自动
 - Horizontal 水平
 - Vertical 垂直
 - Explicit 明确


#Toggle 开关
- 拥有Button所有选项
- Is On 开关
- Toggle Transition 开关过度
 - None 无
 - Fade 逐渐消失
- Graphic 图形(选择状态)
- Group 组(受Toggle Group组建控制)

#Toggle Group 开关组
- Allow Switch Off 允许关掉
 - True 允许所有的Toggle都为Flase
 - Flase 单选模式


#Slider 滑动条
- 拥有Button所有选项
- Fill Rect 填充矩形
- Handle Rect 滑杆矩形
- Direction 方向
	- Laft To Right 从左到右
	- Right To Laft 从右到左
	- Bottom To Top 从下到上
	- Top To Bottom 从上到下
- Min Value 最大值
- Max Value 最小值
- Whole Number 整数(是否只接收整数)
- Value 当前值

#Scrollbar 滚动条
- 拥有Button所有选项
- Handle Rect
- Direction 方向
	- Laft To Right 从左到右
	- Right To Laft 从右到左
	- Bottom To Top 从下到上
	- Top To Bottom 从上到下
- Value 当前值
- Size 大小(操作条矩形对应的缩放长度)
- Number Of Steps 阶梯数量(指定可滚动的位置数量)


#Scroll Rect 滚动矩形

- Content 内容(要滚动的对象)
- Horizontal 水平
- Vertical 垂直
- Movement Type 滑动选项
	- Unrestricted 无限制
	- Elastic 弹性
	- Clamped 夹紧的
- Inertia 惯性力
	- Deceleration Rate 减速的速度 (0~1之间 值越大则惯性力越大)
- Scroll Sensitivity 滚动的敏感性
- Viewport 视口
- Horizontal Scrollbar 水平滚动条
- Vertical Scrollbar 垂直滚动条
	- Visibility 可见性
		- Permanent 永久
		- Auto Hide 自动隐藏
		- Auto Hide And Expand Viewpord 自动隐藏和展开视窗
	- Spacing 间距

#Layout Element 布局元素

- Ignore Layout 忽略布局
- Minmum width 最小宽 (最先被分配 不带任何妥协) 
- Minmum height 最小高
- Preferred width 首选宽 (如果**父类容器**中仍有多余的空间)
- Preferred height 首选高
- Flexible width 灵活宽 (还有额外的空间)
- Flexible height 灵活高
- Layout Priority 布局优先

#Grid Layout Group
- Padding 填充
	- Laft
	- Right
	- Top
	- Bottom
- Cell Size 单元格大小
- Spacing 间隔
- Start Corner 开始角落
	- Upper Left 上左开始
	- Upper Right 上右开始
	- Lower Left 下左开始
	- Lower Right 下右开始
- Start Axis 开始轴
	- Horizontal 水平
	- Vertical 垂直
- Child Alignment 孩子对其
	- Upper Left 上左
	- Upper Center 上中心
	- Upper Right 上右
	- Middle Left 中间左
	- Middle Center 中间中心
	- Middle Right 中间右
	- Lower Left 下左
	- Lower Center 下中心
	- Lower Right 下右
- Constraint 约束
	- Fiexible 活动 
	- Fixed Column Count 固定列数
	- Fixed Row Count 固定行数


#Horizontal Layout 水平布局
- 相对于Grid Layout Group新增以下
- Child Force Expand 孩子能力扩展(自动调整子物体之间的距离以适应父物体的宽和高) 
- Control Child Size 控制子集大小




