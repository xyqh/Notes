# Lua

## 表

### pairs和ipairs的区别

> ipairs
> 遍历的索引的特点
> 1.必须是从1开头的 int类型的连续整数 1 2 3 4 5 6 7 8 9 …………………..
> 2.索引不能断开，断开则终止遍历（当存在nil 类型的数据）
> 终止时机
> 1.索引断开
> 2.下一个索引不存在
>
> pairs
> 遍历的索引的特点
> 1.遍历的顺序是随机的，但是一定会遍历整个表
> 2.pairs是先按照索引值打印（数字型key，可以用[ ]访问），然后打印哈希（键值对）
> 终止时机
> 1.所有元素遍历完毕
>
> ```lua
> print('test pairs and ipairs')
> local t =
> {
>     [1] = 1,
>     2,
>     [3] = 3,
>     4,
>     [5] = 5,
>     [6] = 6
> }
> 
> print('ipairs  ')
> for i, v in ipairs(t) do
>     print(v)
> end
> 
> print('pairs  ')
> for i, v in pairs(t) do
>     print(v)
> end
> 
> -- 输出是：
> -- test pairs and ipairs
> -- ipairs  
> -- 2
> -- 4
> -- 3
> -- pairs  
> -- 2
> -- 4
> -- 3
> -- 6
> -- 5
> ```
>
> 数据在表 t 中的存储方式：
> 1）根据元素类别分别存进哈希表与数组：
> 哈希表：{ [1]=1，[3]=3，[5]=5，[6]=6 }；
> 数组：{ 2，4 }
> 2）将数组中的元素放入哈希表：
> 当数组中的元素转移到哈希表时，数组中的元素变为[1]=2与[2]=4；而此时哈希表中已有键值对 [1]=1，发生冲突，会将新值2重新与键1匹配，即得到新的键值对[1]=2，此时的哈希表元素为：
> { [1]=2，[2]=4，[3]=3，[5]=5，[6]=6 }

### 深拷贝

> ```lua
> function clone(obj)
>     function _clone(obj)
>         if type(obj) ~= "table" then
>             return obj
>         end
>         
>         local newTable = {}
>         for i, v in pairs(obj) do
>             newTable[_clone(i)] = _clone(v)
>         end
>         return setmetatable(newTable, getmetatable(obj))
>     end
>     
>     return _clone(obj)
> end
> ```

### 面向对象实现

> 通过元表和元方法来实现类及继承
>
> ```lua
> require 'Class'
> 
> --声明了新的属性Z
> SubClass = {z = 0}
> --设置类型是Class
> setmetatable(SubClass, Class)
> --还是和类定义一样，表索引设定为自身
> SubClass.__index = SubClass
> --这里是构造方法
> function SubClass:new(x,y,z)
>    local t = {}             --初始化对象自身
>    t = Class:new(x,y)       --将对象自身设定为父类，这个语句相当于其他语言的super 或者 base
>    setmetatable(t, SubClass)    --将对象自身元表设定为SubClass类
>    t.z= z                   --新的属性初始化，如果没有将会按照声明=0
>    return t
> end
> 
> --定义一个新的方法
> function SubClass:go()
>    self.x = self.x + 10
> end
> 
> --重定义父类的方法，相当于override
> function SubClass:test()
>      print(self.x,self.y,self.z)
> end
> ```

### 元方法

> ```lua
> A = {}
> B = {}
> setmetatable(
>     A,
>     {
>         __index = B, --索引查询
>         __newindex = B, --索引更新
>         __add = B, --加法
>         __sub = B, --减
>         __mul = B, --乘
>         __div = B, --除
>         __mod = B, --取模
>         __pow = B, --乘幂
>         __unm = B, --取反
>         __concat = B, --连接
>         __len = B, --长度
>         __eq = B, --相等
>         __lt = B, --小于
>         __le = B, --小于等于
>         __call = B, --执行方法调用
>         __tostring = B, --字符串输出
>         __metatable = B	 --保护元表
>     }
> )
> ```





## 交互

### lua和C的交互

> C和Lua之间的差异
> 1.Lua有垃圾回收机制，C需要显示释放内存
> 2.Lua是动态类型，弱类型语言【运行时确认】，C是静态类型，强类型语言。【编译时确认】
>
> C与Lua的通信使用了虚拟栈结构！！！
> 以下是简单的虚拟栈概念！
> 将2个数据压入虚拟栈
>
> 当使用正数索引时，表示从栈底开始，一直到栈顶 ，使用负数索引时表示从栈顶开始，一直到栈底。
> 通过指定索引来出栈和入栈
>
> C#调用Lua是依靠C作为中间语言，通过C#调用C，C再调用Lua实现的 而框架中的tolua.dll等也就是借助LuaInterface封装的C语言动态库
>
> Lua调用C
> 调用之前需要注册，将函数地址告知Lua
> LuaFramework的框架中Lua要调用Unity自带的API或者我们自己写的脚本之前要先生成对应的XXXWrap文件，就是如上面例子一样，需要在lua里进行注册。

### lua和C#的交互

> #### C#调用lua：
>
> > 过程：
> >
> > C#文件先调用Lua的解析器底层的dll库（C语言编写），再由DLL文件执行相应的Lua文件
> >
> > 原理：
> >
> > C#先将数据放入栈中，然后Lua去栈中获取数据，然后返回数据对应的值到栈顶，再由栈顶返回至C#
> >
> > 代码解释：
> >
> > C#生成Bridge文件，Bridge调dll文件（dll是用C写的库），先调用lua中dll文件，由dll文件执行lua代码
> > C#->Bridge->dll->Lua 或 C#->dll->Lua
>
> #### lua调用C#：
>
> > 过程：
> >
> > 1.Wrap方式：首先生成C#源文件对应的Wrap文件，Lua文件会调用生成的Wrap文件，再由Wrap文件去调用C#文件。
> > 2.反射方式：当索引系统API、DLL库或者第三方库，如果无法将代码具体实现进行代码生成，可通过反射来获取，执行效率较低。
> >
> > 原理：
> >
> > C#源文件生成Wrap文件、或C#源文件生成C模块，将Wrap文件和C模块注册到Lua的解析器中，最后再由Lua去调用这个模块的函数~
> >
> > 代码解释：
> >
> > lua->wrap->C#
> > 先生成Wrap文件（中间文件/适配文件），wrap文件把字段方法，注册到lua虚拟机中（解释器luajit），然后lua通过wrap就可以调C#了、或者在config文件中添加相应类型也可以





## 其他

### 闭包

> 闭包=函数+引用环境
> 子函数可以使用父函数中的局部变量，这种行为可以理解为闭包！
>
> 1、闭包的数据隔离
> 不同实例上的两个不同闭包，闭包中的upvalue变量各自独立，从而实现数据隔离
>
> 2、闭包的数据共享
> 两个闭包共享一份变量upvalue，引用的是更外部函数的局部变量（即Upvlaue）,变量是同一个，引用也指向同一个地方，从而实现对共享数据进行访问和修改。
>
> 3、利用闭包实现简单的迭代器
> 迭代器只是一个生成器，他自己本身不带循环。我们还需要在循环里面去调用它才行。
> 1）while…do循环，每次调用迭代器都会产生一个新的闭包，闭包内部包括了upvalue(t,i,n)，闭包根据上一次的记录，返回下一个元素，实现迭代
> 2）for…in循环，只会产生一个闭包函数，后面每一次迭代都是使用该闭包函数。内部保存迭代函数、状态常量、控制变量。
>
> https://www.cnblogs.com/msxh/p/8283865.html

### lua如何实现热更

> Lua的模块加载机制
> 热更的核心就是替换Package.loaded表中的模块
>
> 导出函数require（mode_name）
> 查询全局缓存表package.loaded
> 通过package.searchers查找加载器
>
> package.loaded
> 存储已经被加载的模块：当require一个mode_name模块得到的结果不为假时，require返回这个存储的值。require从package.loader中获得的值仅仅是对那张表（模块）的引用，改变这个值并不会改变require使用的表（模块）。
>
> package.preload
> 保存一些特殊模块的加载器：这里面的值仅仅是对那张表（模块）的引用，改变这个值并不会改变require使用的表（模块）。
>
> package.searchers
> require查找加载器的表：这个表内的每一项都是一个查找器函数。当加载一个模块时，require按次序调用这些查找器，传入modname作为唯一参数。此方法会返回一个函数（模块的加载器）和一个传给这个加载器的参数。或返回一个描述为什么没有找到这个模块的字符串或者nil。
>
> https://blog.csdn.net/qq_28180261/article/details/100137777?ops_request_misc=%257B%2522request%255Fid%2522%253A%2522161789702516780264057865%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=161789702516780264057865&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~baidu_landing_v2~default-1-100137777.pc_search_result_hbase_insert&utm_term=Lua%E7%83%AD%E6%9B%B4%E7%9A%84%E5%8E%9F%E7%90%86

### gc

> Lua的GC使用了标记清除算法Mark and Sweep
>
> 标记：每一次执行GC前，从根节点开始遍历每一个相关节点，进行标记
> 清除：标记完成后，遍历对象链表，然后对需要执行清除标记的对象，进行清除
>
> 使用三色法：白，灰，黑，作为对象的三种状态
> 新白：可以回收的对象；新创建的对象，初始状态是新白，但不会被清除
> 旧白：可以回收的对象；lua只会清除旧白，GC后，会更新新白
> 灰色：等待回收的对象：该对象已被GC访问过，但该对象引用的其它对象还未标记
> 黑色：不可回收的对象
>
> 简单流程：
> 1.根对象开始标记，将白色对象重置为灰色对象，加入灰色链表
> 2.如果灰色链表不为空，取出一个对象，重置为黑色，并遍历相关引用的对象，重置为黑色
> 3.如果灰色链表为空，清除一次灰色链表
> 4.根据不同类型对象分布回收，类型的存储表
> 5.判断是否遍历到链表尾
> 6.判断对象是否为白色
> 7.将对象重置为白色
> 8.释放资源
>
> 缺点：
>
> 假设A已经扫描完成且为黑，正在扫描B的引用节点C，此时D断开与B的引用关系，转而被A引用，那么D就不会被扫描，从而被销毁。
>
> 核心问题为：
>
> 1. 灰色和白色断开
> 2. 黑色和白色建立引用
>
> 解决方法：
>
> 1. 原始快照（SATB）：将灰色对象断开到白色对象的引用关系记录下来，扫描结束后根据记录以记录的灰色对象为根重新扫描一次
> 2. 增量更新（Incremental Update）：增量更新是将黑色和白色建立引用的过程记录下来，根据这些记录以被记录的黑色对象为根重新扫描一次，用的是写屏障技术(write barrier)