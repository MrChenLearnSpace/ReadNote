

Lua 是通过使用基于寄存器的虚拟机解释字节码来运行的脚本语言，可与c语言程序进行相互调用，创建共享语法框架的定制编程语言

# 值和类型
Lua 中有八种基本类型：nil、布尔值、数字、字符串、函数、用户数据、线程和表。
## nil 和 boolean
nil是值类型，代表该类型无，“无”和“假”都使条件为假;任何其他值都使它为真。
## number
包含整数和浮点数，浮点数精度取决于编译的lua位数，32位或者64位
## string
字符串可以包含任何 8 位值，包括嵌入的零 （' ' \0 ）。Lua 也是与编码无关的;它不对字符串的内容做出任何假设。
## userdata
提供 userdata 类型是为了允许将任意 C 数据存储在 Lua 变量中，用户数据值表示原始内存块，有**两种类型**的用户数据：**full userdata**，它是一个具有由Lua管理的内存块的对象； **light userdata**，它只是一个C指针值。通过**使用元表**，程序员可以定义完整用户数据值的操作。**用户数据值不能在Lua中创建或修改**，只能通过C API。这保证了主机程序拥有的数据的完整性。
## thread
Lua 线程与操作系统线程无关。Lua 支持所有系统上的协程，即使是那些本身不支持线程的系统
## table
实现关联数组，即不仅可以将数字作为索引的数组，还可以将**除 nil 和 NaN 之外的任何 Lua 值**作为索引。任何值为 nil 的键都不被视为表的一部分。相反，不属于表的任何键都具有关联的值 nil。表是 Lua 中唯一的数据结构化机制;它们可用于表示普通数组、列表、符号表、集合、记录、图形、树等。为了表示记录，Lua 使用字段名称作为索引。该语言通过为 a.name 提供语法 a["name"] 糖来支持这种表示形式。
## 引用类型
表、函数、线程和（完整）userdata 值都是对象：变量实际上不包含这些值，只包含对它们的引用。赋值、参数传递和函数返回始终操作对此类值的引用;这些操作并不意味着任何类型的副本。
# 全局环境和环境
在 Lua 中，全局变量 _G 使用相同的值进行初始化
# 错误处理
如果需要在 Lua 中捕获错误，可以在保护模式下使用 pcall 或 xpcall 调用给定函数。
例如
## pcall
```lua
--pcall（执行函数， [, arg1, ···]）
function f(n)
    n = n / 1
end
status = pcall(f,1) 
print(status)
--out
true
```

```lua
function f(n)
    n = n / 1
end
status = pcall(f) 
print(status)
--out
false
```
## xpcall
```lua
--xpcall（执行函数，处理函数 [, arg1, ···]）
function f(n)
    n = n / 1
end
function h(err)
    print("err ",err)
end
status = xpcall(f,h)
print(status)

--out
err     main.lua:2: attempt to perform arithmetic on a nil value (local 'n')
false
```
# 元表和元方法
Lua 中的每个值都可以有一个元表。此元表是一个普通的 Lua 表，它定义了原始值在某些特殊操作下的行为。您可以通过在元表中设置特定字段来更改对值的操作行为的多个方面。
**元表中每个事件的键是一个字符串**
## 元方法
**__index** ：索引访问操作 table[key] 。当不是表或 中 table 不存在时 table ， key 会发生此事件。元方法在 中 table 查找。
**__newindex** ：索引分配 table[key] = value 。与索引事件一样，当不是表或 key 中 table 不存在时，会发生 table 此事件。元方法在 中 table 查找。
**__call** ：调用操作 func(args) 。当 Lua 尝试调用非函数值（即 func 不是函数）时，会发生此事件。元方法在 中 func 查找。如果存在，则调用元方法作为其第一个参数 func ，后跟原始调用 （ ） 的参数 args 。调用的所有结果都是操作的结果。（这是允许多个结果的唯一元方法。
**剩余的运算元方法**，如果加法的任何操作数不是数字（也不是可强制到数字的字符串），Lua 将尝试调用元方法。首先，Lua 将检查第一个操作数（即使它是有效的）。如果该操作数未定义 的 __add 元方法，则 Lua 将检查第二个操作数。如果 Lua 可以找到元方法，它会以两个操作数作为参数调用元方法，调用的结果（调整为一个值）就是操作的结果。否则，它会引发错误。
## 使用lua实现面向对象
新建对象
```lua
--新建类表
man = {name = "", age =0}
function man:new (_name, _age, _new)
    _new =_new or {}
    setmetatable(_new,self)
    self.__index = self
    self.name = _name
    self.age = _age
    return _new
end
--创建新对象
a = man:new("a",10)
b = man:new("b",10)
print(a.age + b.age)
```
## 继承

```lua
woman = man:new ()
function woman:new (_name, _age, _new)
    _new =_new or man:new(_name, _age, _new)--注意有修改
    setmetatable(_new,self)
    self.__index = self
    self.name = _name
    self.age = _age+1
    return _new
end
c = woman:new("c",10)
d = woman:new("d",20)
print(c.age + d.age)
```
# 垃圾回收
Lua 执行自动内存管理。这意味着您不必担心为新对象分配内存或在不再需要对象时释放内存。Lua 通过运行垃圾回收器来收集所有死对象（即无法再从 Lua 访问的对象），从而自动管理内存。Lua 使用的所有内存都受到自动管理：字符串、表、用户数据、函数、线程、内部结构等。

## 具体回收过程
Lua 实现了增量标记和扫描收集器。它使用两个数字来控制其垃圾收集周期：垃圾收集器暂停和垃圾收集器步进乘数。两者都使用百分点作为单位（例如，值 100 表示内部值 1）。垃圾回收器暂停控制回收器在开始新循环之前等待的时间。值越大，收集器的攻击性越小。小于 100 的值表示收集器不会等待开始新的循环。值 200 表示收集器在开始新周期之前等待使用的总内存翻倍。垃圾收集器步进乘数控制收集器相对于内存分配的相对速度。较大的值使收集器更具侵略性，但也会增加每个增量步骤的大小。不应使用小于 100 的值，因为它们会使收集器太慢，并可能导致收集器永远不会完成一个循环。默认值为 200，这意味着收集器以内存分配速度的“两倍”运行。如果将步进乘法器设置为非常大的数字（大于程序可能使用的最大字节数的 10%），则收集器的行为类似于停止世界收集器。如果随后将暂停设置为 200，则收集器的行为与旧 Lua 版本中的行为相同，每次 Lua 的内存使用量翻倍时都会执行一个完整的收集。
## 手动回收
调用 lua_gc 在 C 或 collectgarbage 在 Lua中，您还可以使用这些功能直接控制收集器（例如，停止并重新启动它），这些元方法也称为终结器。终结器允许您将 Lua 的垃圾回收与外部资源管理（例如关闭文件、网络或数据库连接或释放您自己的内存）协调起来。
对于要在收集时完成的对象（表或用户数据），必须将其标记为完成。当您设置对象的元表时，您可以标记要完成的对象，并且元表具有由字符串 “ __gc ” 索引的字段。请注意，如果设置一个没有字段的元表，然后稍后在元表中创建该 __gc 字段，则不会将对象标记为定型。
## 垃圾回收元方法 __gc
您还可以使用这些功能直接控制收集器（例如，停止并重新启动它）,自定义修改gc方法
## 弱表
弱表是其元素是弱引用的表。垃圾回收器将忽略弱引用。如果对对象的唯一引用是弱引用，则垃圾回收器将收集该对象。
**弱表可以具有弱键和/或弱值。具有弱值的表允许收集其值，但阻止收集其键。**
表的弱点由其元表的 __mode 字段控制。如果 __mode 字段是包含字符 'k' 的字符串，则表中的键较弱。如果包含 ' v ' ，则 __mode 表中的值较弱。
**具有弱键和强值的表也称为星历表。在星历表中，仅当值的键可访问时，该值才被视为可访问**
特别是，如果对键的唯一引用来自其值，则会删除该对。
**只有具有显式构造的对象才会从弱表中删除**。值（如数字和轻 C 函数）不受垃圾回收的影响，因此不会从弱表中删除（除非收集其关联值）。尽管字符串受垃圾回收的影响，但它们没有显式构造，因此不会从弱表中删除。
**复活的对象（即，正在完成的对象和只能通过正在完成的对象访问的对象）在弱表中具有特殊行为。**它们在运行终结器之前从弱值中删除，但只有在运行终结器后的下一个集合中，当这些对象实际被释放时，才会从弱键中删除。此行为允许终结器通过弱表访问与对象关联的属性。如果弱表在收集周期中复活的对象中，则在下一个循环之前可能无法正确清除该表。
# 协程
第一创建协程
coroutine.create(func) 参数一个函数
```lua
function foo (a)
    print("foo", a)
    return coroutine.yield(2 * a)
end

co = coroutine.create(function(a, b)
    print("co-boby",a,b)
    local r = foo(a + 1)
    print("co-boby",r)
    local r,s = coroutine.yield(a+b,a-b)
    print("co-body",r,s)
    return b, "end"
end)
```
yield 挂起跳出,包括嵌套函数中也能跳出
```lua
coroutine.yield(2 * a)
```
coroutine.resume(co, ...)
如果函数是挂起跳出则将从挂起跳出的位置继续执行函数，塞入的额外参数将会通过yield赋值传入函数；如果函数未跳出过将从头执行。
```lua
print("main",coroutine.resume(co, 1, 10))
print("main",coroutine.resume(co, "r"))

result 
co-boby 1       10
foo     2
main    true    4
co-boby r
main    true    11      -9
```
返回是否无异常， yield()内的参数，若出错将返回报错信息
```lua
print("main",coroutine.resume(co, "x", "y"))
print("main",coroutine.resume(co, "x", "y"))

result
co-body x       y
main    true    10      end
main    false   cannot resume dead coroutine
```



