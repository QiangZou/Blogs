# Lua

## 起点

### Chunks

- 一行一个语句
- 分号（;）可选


### 全局变量

- 不需要声明
- 赋值为nil即删除


### 词法约定

- 不能用保留字
- 大小写敏感
- 单行注释--
- 多行注释：--[[ --]] 

### 命令行方式

## 类型和值

### Nil

- 特殊类型
- 变量没赋值前为nil

### Booleans

- false和nil为假其他都为真
- 0和空字符串为真

### Numbers

- 实数
- 小于100,000,000,000,000没误差

### Strings

- 8字节
- 不可修改
- 单引号（''）双引号（""）都行
- 多行用[[...]]
- 字符串连接符(..)
- 转义符\

```
\a		响铃（电脑“嘀”声）
\b		后退
\f		换页
\n		换行
\r		回车
\t		制表
\v		tab
\\		"\"
\"		双引号
\'		单引号
\[		左中括号
\]		 右中括号
```


### Functions

- 第一类值
- Lua标准库都是C实现的


### Userdata and Threads


## 表达式


### 算术运算符

- 一元运算符：- (负值)
- 二元运算符：+ - * / ^ (加减乘除幂) 
- 运算符的操作数都是实数
- 运算string 和 numbers自动类型转换


### 关系运算符

```
< > <= >= == ~= 
```
- nil只和自己相等
- tables、userdata、functions比较引用
- 字母顺序依赖本地环境



### 逻辑运算符

```
and or not 
```
- and 的优先级比 or 高
- false和nil为假其他都为真
- and 和 or 的运算结果不是 true 和 false（特别注意）
```
a and b -- 如果 a 为 false，则返回 a，否则返回 b
a or b -- 如果 a 为 true，则返回 a，否则返回 b 
例如
print(4 and 5) --> 5 
print(4 or 5) --> 4 
```



### 连接运算符

```
.. 		--两个点
```

### 优先级

```
^
not - (unary)
* /
+ -
..
< > <= >= ~= ==
and
or
```


### 表的构造

- 最简单的构造函数{}
- 默认从1开始
- 构造函数中可以用逗号(,)可以用分号(;)


## 基本语法

###赋值语句

- 可以同时多个赋值
- 赋值个数少用nil补


### 局部变量与代码模块

- local创建
- 一个控制结构内，一个函数体，一个chunk为代码块


### 控制结构语句

- if语句
```
if conditions then
	then-part
elseif conditions then
	elseif-part
.. --->多个 elseif
else
	else-part
end;
```

- while 语句
```
while condition do
	statements;
end; 
```

- repeat-until 语句
```
repeat
	statements;
until conditions; 
```

- for 语句有两大类
 - 数值 for 循环
```
for var=exp1,exp2,exp3 do
		loop-part
end
-- exp1为初始值
-- exp2为终止值
-- exp3为增加值 可省略 默认为+1
```
 - 范型 for 循环
```
for i,v in ipairs(a) do 
		print(v) 
end
```


### break 和 return 语句


- break退出当前循环
- return返回函数结果
- 提前调用使用do return end(待测试)

## 函数

### 返回多个结果值

- return x,y 返回多个参数
- 可以使用圆括号强制使调用返回一个值
- unpack接受一个table作为参数,并默认从下标1开始返回数组的所有元素
```
local info={1,2,3,nil,5,p=6}
local a,b,c,d,e,f=unpack(info)
print(a,b,c,d,e,f)
输出结果：1   2   3   nil    5   nil	
```

### 可变参数

- 可变参数（...）
- 可变参数存在arg表中
- arg表中有一个域n表示参数的个数



### 命名参数

- 额 好像就是用table作为参数


## 再论函数

- 第一类值指：在 Lua 中函数和其他值（数值、字符串）一样，函数可以被存放在变量中，也可以存放在表中，可以作为函数的参数，还可以作为函数的返回值
- 词法定界指：被嵌套的函数可以访问他外部函数中的变量。这一特性给 Lua 提供了强大的编程能力

### 闭包

- upvalue：外部的局部变量
- 闭包：带upvalue的特殊函数
- 还没完全理解



### 非全局函数

- 函数作为table的域
- 函数和表语法：
  - 表和函数放一起
```
Lib = {}
Lib.foo = function (x,y) return x + y end
Lib.goo = function (x,y) return x - y end
```
  - 使用表构造函数
```
Lib = {
	foo = function (x,y) return x + y end,
	goo = function (x,y) return x - y end
} 
```
  - Lua 提供另一种语法方式
```
Lib = {}
function Lib.foo (x,y)
	return x + y
end
function Lib.goo (x,y)
	return x - y
end 
```


### 正确的尾调用（Proper Tail Calls）

- 函数最后一个动作是调用另一个函数
- 尾调用不需要使用栈空间
- 常用于状态机


## 迭代器与泛型 for



### 迭代器与闭包

- 实现一个迭代器
```
function list_iter (t)
	local i = 0
	local n = table.getn(t)
	return function ()
 		i = i + 1
 		if i <= n then 
			return t[i] 
		end
	end
end 
```
- 使用for语句循环
```
t = {10, 20, 30}
	for element in list_iter(t) do
		print(element)
end 
```

### 范性 for 的语义

```
for var_1, ..., var_n in explist do block end 
```
等价于

```
do
	local _f, _s, _var = explist
	while true do
		local var_1, ... , var_n = _f(_s, _var)
		_var = var_1
		if _var == nil then break end
		block
	end
end 
```

### 无状态的迭代器
```
a = {"one", "two", "three"}
for i, v in ipairs(a) do
	print(i, v)
end 
```
也可以这样实现
```
function iter (a, i)
	i = i + 1
	local v = a[i]
	if v then
 		return i, v
	end 
end

function ipairs (a)
	return iter, a, 0
end 
```
Lua 库中实现的 pairs 是一个用 next 实现原始方法：
```
function pairs (t)
	return next, t, nil
end 
```


### 多状态的迭代器



### 真正的迭代器

## 编译·运行·调试

- dofile
 - 功能：载入文件并执行代码块，对于相同的文件每次都会执行
 - 定位：内置操作，辅助函数

- loadfile
 - 功能：载入文件但不执行代码块，对于相同的文件每次都会执行。只是编译代码，然后将编译结果作为一个函数返回

- loadstring
 - 功能：与loadfile类似，loadstring是从一个字符串中读取代码
 - 定位：执行外部代码，如：用户的输入

- dostring
 - 功能：类似dofile:加载并运行



### require 函数


- 不重复加载文件
- 自动搜索目录
 - 全局变量 LUA_PATH 是否为一个字符串
 - 环境变量 LUA_PATH 的值
 - 固定的路径（典型的"?;?.lua"）


### C Packages

 - loadlib 函数（不调用初始化函数方式加载 返回一个函数）
```
local path = "/usr/local/lua/lib/libluasocket.so"
local f = assert(loadlib(path, "luaopen_socket")) 
```


### 错误

 - 调用 error 函数显示的抛出错误
 - 内置函数 assert
  - 检查第一个参数是否返回错误
  - 第二个参数抛出错误信息
```
n = io.read()
assert(tonumber(n), "invalid input: " .. n .. " is not a number") 
```




### 异常和错误处理


- 使用 pcall 函数封装你的代码
- 没有异常和错误，返回 true 和调用返回的任何值；
- 有异常和错误，返回 nil 加错误信息



### 错误信息和回跟踪（Tracebacks）

##协同程序


#第二篇 tables 与 objects

##第 11 章 数据结构


###11.1 数组

- 默认下标用1开始



###11.2 阵和多维数组

- 数组的数组表示(n 行 m 列)
```
array = {}
for i=1,n do
   array[i] = {}
      for j=1,m do
         array[i][j] = i*j
      end
end
```
- 将行和列组合起来作为索引下标


###11.2 链表

- 实现一个简单的链表
```
list = {next = list, value = v} 
```
- 遍历
```
local l = list
while l do
	print(l.value)
	l = l.next
end 
```



###11.4 队列和双端队列


- insert 和 remove针对大数据效率太低
- 使用两个索引下标
```
List = {}
function List.new ()
return {first = 0, last = -1}
end 
```
-实现双端队列方法
```
function List.pushleft (list, value)
local first = list.first - 1
	list.first = first
	list[first] = value
end
--
function List.pushright (list, value)
local last = list.last + 1
	list.last = last
	list[last] = value
end
--
function List.popleft (list)
local first = list.first
if first > list.last then error("list is empty") end
local value = list[first]
	list[first] = nil -- to allow garbage collection
	list.first = first + 1
return value
end
--
function List.popright (list)
local last = list.last
if list.first > last then error("list is empty") end
local value = list[last]
	list[last] = nil -- to allow garbage collection
	list.last = last - 1
return value
end 
```

###11.5 集合和包

- 将元素作为下标存放在一个table中判断标识符

###11.5 字符串缓冲

- 拼接很多个字符串使用 table.concat


##第 12 章 数据文件与持久化


###12.1 序列化