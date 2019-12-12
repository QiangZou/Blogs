

## 效果展示 ##
一个很简单的组件脚本

![](https://i.imgur.com/2bPg0DI.png)

运行状态在Inspector面板可以随便修改字段和调用方法

![](https://i.imgur.com/8jEI97d.png)

方法调用日志

![](https://i.imgur.com/8V64zbM.png)

## 设计由来 ##

- 最近在学习反射
- 结合游戏开发过程遇到比较难受的事情

## 应用场景 ##

- 游戏特别庞大、电脑特别垃圾、重新运行一次Unity需要等待几十秒的情况下

- 你需要修改一个组件字段或者调用一个方法展示一个动画等等

- 这个时候你肯定渴望可以直接修改字段或者直接调用某个方法

- 反射就可以实现
 - 修改实例对象所有的字段包括私有字段
 - 调用实例对象所有的方法包括私有方法




- 总结一下：就是可以瞎几把修改组件字段和调用组件方法 

## 反射可以做哪些事 ##

- 获取类的所有字段属性方法包括私有的

- 修改类的所有字段包括私有的

- 修改类的所有属性包括私有的

- 调用类的所有方法包括私有的

- 实例化一个类（这个工具用不到）

## 制作流程 ##

- 新建一个继承MonoBehaviour的类 ReflectionMonoBehaviour
 - 在类的Start方法获取挂在游戏对象上的其他组件实例


- 新建一个的Editor类 ReflectionMonoBehaviourEditor
 - 自定义编辑脚本Inspector面板
 - 通过上面获取的组件实例
 - 反射出所有的字段、属性、方法
 - 然后在Inspector面板显示出来
 - 通过在Inspector面板修改、点击按钮
 - 使用反射修改、调用组件实例

## 使用到的反射方法 ##


```C++

using System.Reflection;

//获取实例组件的Type
Type type = 实例对象.GetType();

//获取实例组件的所有字段（BindingFlags限制枚举）
FieldInfo[] allFieldInfo = type.GetFields(BindingFlags.NonPublic | BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.Public | BindingFlags.DeclaredOnly | BindingFlags.Static);

//获取实例组件的所有方法（BindingFlags限制枚举）
MethodInfo[] allMemberInfo = type.GetMethods(BindingFlags.NonPublic | BindingFlags.NonPublic | BindingFlags.Instance | BindingFlags.Public | BindingFlags.DeclaredOnly | BindingFlags.Static);

//获取字段的类型（int string float 等等）
FieldInfo.FieldType

//获取方法的所有参数
MethodInfo.GetParameters()

//获取参数的类型
ParameterInfo.ParameterType

//修改实例组件的字段
Field.fsieldInfo.SetValue(实例对象, 值)

//方法实例组件的方法
Method.methodInfo.Invoke(实例对象, 所有参数);

```

## 工具待完善 ##

目前只支持一些类型的字段修改和调用方法
如果有需要可以自己修改  很简单的


## 源码例子地址 ##
源码：[https://github.com/QiangZou/ZouQiang/tree/master/Assets/ZouQiang/Tool/ReflectionMonoBehaviour](https://github.com/QiangZou/ZouQiang/tree/master/Assets/ZouQiang/Tool/ReflectionMonoBehaviour "https://github.com/QiangZou/ZouQiang/tree/master/Assets/ZouQiang/Tool/ReflectionMonoBehaviour")
例子：[https://github.com/QiangZou/ZouQiang/tree/master/Assets/ZouQiangExample/ReflectionMonoBehaviour](https://github.com/QiangZou/ZouQiang/tree/master/Assets/ZouQiangExample/ReflectionMonoBehaviour "https://github.com/QiangZou/ZouQiang/tree/master/Assets/ZouQiangExample/ReflectionMonoBehaviour")