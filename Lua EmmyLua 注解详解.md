# Lua EmmyLua 注解详解

## **Why**

- 为了使IDE编码体验和强语言相近
- 让IDE提前发现编码错误
- BUG查找更方便
- 代码阅读更方便

##  建议

- 明确字段**类型**
- 明确字段访问**修饰符**
- 明确方法参数**类型**
- **善用	":" 继承	"|" 或	","多个** 

## 支持格式

```lua
--类            ---@class MY_TYPE[:PARENT_TYPE] [@comment]
--类型          ---@type MY_TYPE[|OTHER_TYPE] [@comment]
--别名          ---@alias NEW_NAME TYPE
--参数          ---@param param_name MY_TYPE[|other_type] [@comment]
--返回值        ---@return MY_TYPE[|OTHER_TYPE] [@comment]
--字段          ---@field [public|protected|private] field_name FIELD_TYPE[|OTHER_TYPE] [@comment]
--泛型          ---@generic T1 [: PARENT_TYPE] [, T2 [: PARENT_TYPE]]
--不定参数      ---@vararg TYPE
--内嵌语言      ---@language LANGUAGE_ID
--数组          ---@type MY_TYPE[]
--字典          ---@type table<KEY_TYPE, VALUE_TYPE>
--函数          ---@type fun(param:MY_TYPE):RETURN_TYPE
```

## 官网完整例子

```lua
---@class Transport @parent class
---@field public name string
local transport = {}

function transport:move() end

---@class Car : Transport @Car extends Transport
local car = {}
function car:move() end

---@class Ship : Transport @Ship extends Transport
local ship = {}

---@param type number @parameter type
---@return Car|Ship @may return Car or Ship
local function create(type)
    -- ignored
end

local obj = create(1)
---now you can see completion for obj

---@type Car
local obj2
---now you can see completion for obj2

local list = { obj, obj2 }
---@param v Transport
for _, v in ipairs(list) do
    ---not you can see completion for v
end
```

## 自己验证例子

```lua
---@class TestBase @基类
---@field protected key number @基类字段

---@class Test : TestBase @测试类
---@field bool boolean @boolean类型字段
---@field numberArray number[] @数组
---@field numberDictionary table<number,number> @字典

---@type Test
local Test = {}


---@type number @number类型字段（后期扩展字段 IDE不能识别注释）
Test.num = 0


---方法1
function Test:Func1()
    --字段测试
    self.key = 0 --能跳转基类
    self.bool = false
    self.num = 1 --IDE不能识别注释
    for i, v in ipairs(self.numberArray) do end
    for k, v in pairs(self.numberDictionary) do end --遍历能识别 k v 类型

    --方法测试
    self:Func2("张三")
    local tempFunc3 = self:Func3("李四")
    local tempFunc4A, tempFunc4B = self:Func4("王五", false)
    local tempFunc5 = self:Func5(false)
end

---方法2 有参数
---@param name string @名字
function Test:Func2(name)
end

---方法3 有返回值
---@return string @返回类型
function Test:Func3(name)
    return name
end

---方法4 多参数 多返回值
---@param name string @名字
---@param sex boolean @性别
---@return string , number @返回类型
function Test:Func4(name, sex)
    return name, sex
end

---方法5 参数多类型 返回值多类型
---@param sex string | boolean @性别
---@return string | boolean @返回类型
function Test:Func5(sex)
    return sex
end

---方法6 参数为方法
---@param func fun(key:number):string @函数
function Test:Func6(func)
    return func(1)
end


--使用 see 注解来标注一个引用
---@see Test#Func1




--下面的不常用


--不定参数注解
---@vararg string
---@return string
local function format(...)
    local tbl = { ... } -- inferred as string[]
end

--泛型
--几乎不用 C#用是因为用 object 作为参数 有装箱拆箱消耗 lua语言天然不需要

---@class Goods @物品基类
---@field public price number @价格

---@class Food : Goods @食物
---@field public cal number @卡路里

---@class Phone : Goods @手机
---@field public battery number @电量

---@generic T : Goods
---@param object T
function Test:GetPrice(object)
    return object.price
end

--内嵌语言
---@language JSON
local jsonText = [[{
    "name":"Emmy"
}]]


return Test
```

