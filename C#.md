# C#

## 基础

### 面向对象的三大特点

> **封装**：将数据和行为相结合，通过行为约束代码修改数据的程度，增强数据的安全性，属性是C#封装实现的最好体现。就是将一些复杂的逻辑经过包装之后给别人使用，使用者不需要了解内部的实现逻辑，只需要传入所需的参数就可以得到想要的结果。封装的意义在于保护或者防止代码被无意破坏。
>
> **继承**：提高代码重用率，增强软件可维护性的重要手段，符合开闭原则。继承最主要的作用就是把子类的公共属性集合起来，便于共同管理，使用起来也更加方便。在C#中，只支持单继承，不支持多继承（多个同级父类）。
>
> **多态**：同名的方法在不同环境下，自适应地做出不同的表现。

### 重载和重写的区别

> 1. 所处位置不同，重载在同类中，重写在父子类中
> 2. 定义方式不同，重载方法名相同，参数列表不同，重写方法名和参数列表都相同
> 3. 调用方式不同，重载使用相同对象以不同参数调用，重写用不同对象以相同参数调用
> 4. 多态时机不同，重载是编译时多态，重写是运行时多态

### 值类型和引用类型的区别

> **值类型**：整型、浮点型、bool、char、struct、enum。继承自System.ValueType
>
> **引用类型**：string、object、class、interface、delegate、array。继承自System.Object
>
> |                  | 值类型                             | 引用类型                               |
> | ---------------- | ---------------------------------- | -------------------------------------- |
> | 存储             | 存储在栈上                         | 实际数据存储在堆上，堆的地址存储在栈上 |
> | 存取速度（相对） | 快                                 | 慢                                     |
> | 回收机制         | 压栈自动分配内存，出栈自动释放内存 | .NET的GC                               |
> | 赋值操作         | 深拷贝（也浅不了）                 | 浅拷贝                                 |
> | 参数传递         | 复制一份实例                       | 复制一份地址，指向同一实例             |
> | ref和out         | 引用传递，改本身                   | 直接操作地址                           |
>

### struct和class的区别

> |              | struct                                                       | class                                   |
> | ------------ | ------------------------------------------------------------ | --------------------------------------- |
> | 数据类型     | 值类型                                                       | 引用类型                                |
> | 默认成员公开 | public                                                       | private                                 |
> | 是否可继承   | 不可继承，不接受virtual、protected、sealed等关键字           | 可继承                                  |
> | 构造函数     | 不能定义默认的构造器（无参构造函数），只能定义带参构造函数，且带参构造函数需初始化所有字段 | class都可以定义，且不需要初始化所有字段 |
> | 析构函数     | 没有                                                         | 有                                      |
> | 字段         | 不能在声明时初始化                                           | 可以在声明时初始化                      |
> | 属性         | 可以在声明时初始化                                           | 可以在声明时初始化                      |
> | 可空         | 可以作为可空类型（Nullable<T>）的元素                        | 不可以作为可空类型的元素                |

### public, private, protected, internal的区别

> **public**：对任何类和成员公开，无限制访问
>
> **private**：仅对该类公开
>
> **protected**：对该类和派生类公开
>
> **internal**：只能在包含该类的程序集中访问该类
>
> **protected internal**：使一个成员在当前程序集以及派生类中可见。具体来说，它可以被以下几种对象访问：
>
> - 程序集内的任何类型
> - 从包含该成员的类派生的任何类型（无论程序集内还是外部）
> - 从包含该成员的程序集派生的任何类型

### ArrayList和List的主要区别

> ArrayList：非泛型，存储的类型默认是object，需要装箱和拆箱
>
> List：泛型，不需要进行装箱和拆箱，性能更优
>
> 都继承了IList接口
>
> ```c#
> ArrayList list = new ArrayList(6);
> Debug.Log(list.Capacity);
> for(int i = 0; i < 10; ++i)
> {
>     list.Add(i);
> }
> Debug.Log(list.Capacity);
> /**
> 6
> 12
> */
> 
> int[] arr = { 1, 2, 3, 4, 5 };
> ArrayList list = new ArrayList(arr);
> Debug.Log(list.Capacity);
> /**
> 5
> */
> ```

### Array

> ```c#
> int[] list = {34,72,13,44,25,30,10};
> Array.Sort(list);
> Array.Reverse(list);
> ```

### GC产生的原因以及如何避免

> what：垃圾回收，扫描那些没有被引用的对象（或者超出作用域）并回收内存。
>
> why：避免内存溢出。
>
> prevent：少new就好了。具体一点的措施是：
>
> - 减少new对象的次数
> - 使用静态成员
> - 用StringBuilder而不是string

### 接口（Interface）和抽象类的区别

> 1. 接口不是类，不能实例化，抽象类可以间接实例化
> 2. 接口是完全抽象，抽象类为部分抽象
> 3. 接口可以多继承，抽象类只能单继承

### Sealed关键字的作用

> 在类声明时可以防止类被继承，在方法中声明则可以防止派生类重写此方法。
>
> ```c#
> sealed class a{
>     sealed public void fun(){
>         
>     }
> };
> ```

### 反射的实现原理

> 在加载程序运行时，动态获取和加载程序集，并且可以获取到程序集的信息反射即在运行期动态获取类、对象、方法、对象数据等的一种重要手段，主要使用的类库：System.Reflection
>
> 核心类：
>
> 1. Assembly描述程序集
> 2. Type描述类型
> 3. ConstructorInfo描述构造函数
> 4. MethodInfo描述所有的方法
> 5. FieldInfo描述类的字段
> 6. PropertyInfo描述类的属性
>
> 通过以上核心类可在运行时动态获取程序集中的类，并执行类构造产生类对象，动态获取对象的字段或属性值，更可以动态执行类方法和实例方法等。
>
> ```c#
> 实现步骤：
> /*1. 导⼊*/using System.Reflection;
> /*2. */Assembly.Load("程序集") // 加载程序集,返回类型是⼀个Assembly
> /*3.  */foreach (Type type in assembly.GetTypes())
>          {
>             string t = type.Name;
>          }
> // 得到程序集中所有类的名称
> /*4. */Type type = assembly.GetType("程序集.类名"); //获取
> //当前类的类型
> /*5. */Activator.CreateInstance(type);// 创建此类型实例
> /*6. */MethodInfo mInfo = type.GetMethod("⽅法名");//获取
> //当前⽅法
> /*7. */mInfo.Invoke(null,/*⽅法参数*/null);
> ```

### Unity中C#的底层原理

> unity在运行C#程序时有两种机制，一种是Mono，另一种是IL2CPP。
>
> #### IL：
>
> > IL全称是Intermediate Language，但很多时候我们看到的是CIL（Common Intermediate Language，特指在.NET平台下的IL标准），其实大部分文章提到的IL和CIL表示的是同一个东西，即中间语言。
> >
> > IL是一种低阶的人类可读的编程语言。我们可以将通用语言翻译成IL，然后汇编成字节码，最后运行在虚拟机上，也可以把IL看作一个面向对象的汇编语言，只是它必须运行在虚拟机上，而且是完全基于堆栈的语言。也就是说，C#、VB、J#这种遵循CLI规范的高级语言，会被各自的编译器编译成中间语言IL（CIL），当需要运行它们时就会实时地加载到运行时库，由虚拟机动态地编译成汇编代码并执行。
> >
> > 其中IL有三种转译模式：
> >
> > 1. Just-in-time（JIT）编译：在程序运行过程中将CIL转译为机器码。
> > 2. Ahead-of-Time（AOT）编译：将IL转译成机器码并存储在文件中，此文件并不能完全独立运行。通常这种模式可产生出绝大部分JIT模式所产生的机器码，只有部分例外，例如trampolines或是控管监督相关的代码仍旧需要JIT来运行。
> > 3. 完全静态编译：这个模式只支持少数平台，它基于AOT编译模式更进一步产生所有的机器码。完全静态编译模式可以让程序在运行期完全不需要用到JIT，这个做法适用于iOS操作系统、PlayStation 3以及XBox 360等不允许使用JIT的操作系统。
> >
> > Unity在打包iOS操作系统的时候就是用了第三种方式，而在Android和Windows上则使用JIT实时编译来运行代码。
>
> #### Mono：
>
> > unity中使用C#语言的一个重要好处就是编译快且开发效率高。但是.NET只能在Windows上运行，主要是因为微软没有考虑跨平台且没有将其开源。后来微软公司向ECMA申请将C#作为一种标准。C#在2003年成为了一个ISO标准（ISO/IEC 23270）。这意味着只要是遵守CLI（Common Language Infrastructure）的第三方就可以将任何一种语言实现到.NET平台上。Mono就在这种环境下诞生了，Mono的目标就是达成跨平台.NET4.0的完整功能支持。
> >
> > Mono内包含三类组件，分别是核心组件、Mono/Linux/GNOME开发堆栈、微软兼容堆栈。
> >
> > - 核心组件包含C#编译器、Common Language Infrastructure虚拟机，以及核心类别程序库。
> > - Mono/Linux/GNOME开发堆栈提供了工具用于开发应用软件。这些工具使用了既有的GNOME以及自由且开放源代码的程序库，它们包含了针对图形用户界面开发的Gtk#、可套用Gecko rendering engine的Mozilla程序库、Unix集成程序库（Mono.Posix）、安全堆栈，以及XML schema语言RelaxNG。
> > - 微软兼容堆栈提供了一种方式以使Windows .NET应用程序可以被移植到GNU/Linux上，堆栈包含ADO.NET、ASP.NET以及Windows Forms等。
> >
> > Mono使用垃圾回收机制来管理内存，应用程序向垃圾回收器申请内存，最终由垃圾回收器决定是否回收。当我们向垃圾回收器申请内存时，如果发现内存不足，就会自动触发垃圾回收，或者也可以主动触发垃圾回收，垃圾回收器此时会遍历内存中所有对象的引用关系，如果没有被任务对象引用则会释放内存。
> >
> > 在3.1.1版之后，Mono正式将Simple Generational GC（SGen-GS）设置为默认的垃圾回收器。SGen-GC的主要思想是：
> >
> > 将对象分为两个内存池，一个较新，一个较老，那些存活时间长的对象都会被转移到较老的内存池。这是基于一个事实：程序经常会申请一些小的临时对象，用完了马上就释放。而如果某个对象一段时间没被释放，往往很长时间不会释放。

### 静态构造函数

> 静态构造函数是C#的一个新特性，用于初始化一些静态变量。在创建第一个实例或应用任何静态成员之前，由.NET自动调用，只会执行一次。
>
> 1. 静态构造函数既没有访问修饰符，也没有参数。因为是.NET调用的，所以像public和private等修饰符是没有意义的。
> 2. 在创建第一个类实例或任何静态成员被引用时，.NET将自动调用静态构造函数进行初始化。我们无法直接调用，也无法控制什么时候执行静态构造函数。
> 3. 一个类只能有一个静态构造函数
> 4. 无参数的构造函数可以与静态构造函数共存。尽管参数列表相同，但一个属于类，一个属于实例，所以不会冲突
> 5. 最多只允许一次
> 6. 静态构造函数不可以被继承
> 7. 如果没有写静态构造函数，而类中包含带有初始值设定的静态成员，那么编译器会自动生成默认的静态构造函数
>
> 
>
> 类的静态构造函数也叫类型构造器，静态构造器，他调用的时刻由CLR来控制：
>
> CLR会选择如下时间之一来调用静态构造函数：
>
> 1. 在类型的第一个实例创建之前，或类型的非继承字段或成员第一次访问之前。这里的“之前”，代表前后衔接的意思。这里的时刻是精确的！
> 2. 在非继承的静态字段或成员第一次访问之前的某个时刻，具体时刻不定！
>
> 由于调用的时刻不确定，所以我们最好不要编写依赖于特定的静态构造函数的执行顺序的代码，这样很容易产生不可预料的后果！

### string和stringBuilder对比

> 处理字符串的话，string中的方法每次都需要创建一个新的字符串对象并且分配新的内存地址，而stringBuilder是在原来的内存里对字符串进行修改，所以stringBuilder表现好一点。
>
> string是不可变的，天然线程同步。
>
> stringBuilder可变，非线程同步。

### Lambda表达式

> ```c#
> Func<int, string, bool> func1 = (idx, str) => {
>     return false;
> };
> 
> Action<int, string> func2 = (idx, str) => {
>     
> };
> ```

### unsafe关键字

> 托管代码：执行过程交由运行时管理的代码。在这种情况下，相关的运行时称为公共语言运行时（CLR，Common Language Running），不管使用的是哪种实现。CLR负责提取托管代码、将其编译成机器代码，然后运行它。除此之外，运行时还提供多个重要服务，例如自动内存管理、安全边界和类型安全。
>
> 非托管代码：依赖于平台和语言，需要自己提供安全检测、垃圾回收等操作。
>
> 非托管代码需要unsafe关键字，一般用在带指针操作的场合。

### ref和out

> ref修饰引用参数。参数必须赋值，带回返回值。
>
> out修饰输出参数。参数可以不赋值，带回返回值之前必须明确赋值。

### for、foreach、Enumerator.MoveNext

> for循环通过索引一次进行遍历。
>
> foreach和Enumerator.MoveNext通过迭代的方式进行遍历，此外foreach时，不能增加或修改元素（有版本记录，version，Add或者Clear之后会version++），同时也不能在循环内修改元素（只读）。
>
> 能用foreach遍历访问的对象需要实现IEnumrable接口或GetEnumerator方法。

### string的+=有什么问题

> 可能会产生大量的内存碎片，string内每次都是新new一块地方存放+=后的结果。
>
> 据某些博客称，现在的编译器已经把string的+操作优化成stringBuilder了，所以一般用string就可以了。（不知道怎么打印地址，没试）

### 委托和事件

> **委托delegate**：
>
> 所有的函数指针功能都以委托的方式来完成。委托可以视为一个更高级的函数指针，它不仅能把地址指向另一个函数，而且还能传递参数、获得返回值等多个信息。
>
> 我们在创建委托时，其实就是创建一个delegate类实例，这个delegate委托类继承了System.MulticastDelegate类，类实例里有BeginInvoke()、EndInvoke()、Invoke()这三个函数，分别表示异步开始调用，结束异步调用及直接调用。
>
> delegate委托类还重写了+=、-=这两个操作符，对应MulticastDelegate类的Combine()和Remove()方法，当对函数进行+=和-=操作时，相当于把函数地址推入链表尾部或者移除链表。
>
> **事件event**：
>
> 事件在委托的基础上做了一次封装，限制用户直接操作delegate实例中变量的权限。
>
> 封装后，用户不能再直接用赋值操作来改变委托变量，只能通过注册或者注销委托的方法来增减委托函数的数量。即event声明的委托不再提供赋值操作符，但仍有“+=”和“-=”操作符可供操作。
>
> ```c#
> public delegate int DelegateExample(int target);
> public event DelegateExample EventExample;
> 
> int OnTargetChanged(int target)
> {
> }
> 
> EventExample += OnTargetChanged;
> ```
>
> C#的三个预定义委托：
>
> ```c#
> // 接受0-16个输入，不带返回值
> Action<int, string> myAction = (x, y)=>{
> 
> };
> myAction(10, "Hello");
> 
> // 接受0-16个输入，带一个返回值
> Func<int, int, int> myFunc = (x, y)=>{
>     return x+y;
> }
> int z = myFunc(10, 20);
> 
> // 只接受一个输入返回一个bool
> Predicate<int> myPredicate = (x)=>{
>     return x > 0;
> }
> bool isPositive = myPredicate(1);
> ```

### 委托和函数指针的区别

> 委托对象是真正的对象，函数指针只是函数的入口地址。
>
> 委托是类型安全的，函数指针不是。

### 委托和接口的区别，应用场景

> 接口（interface）是约束类应该具备的功能集合，约束了类应该具备的功能，使类从千变万化的具体逻辑中解脱出来，便于类的管理和扩展，同时又解决了类的单继承问题。适用于无法使用继承的场合；完全抽象的场合以及多人协作的场合。
>
> 委托是约束方法集合的一个类，可以边接地使用委托对这个方法集合进行操作。多用于事件处理中。

### C#泛型

> 参考C++模板，但不支持特化和偏特化。
>
> 可以将泛型看作是一种增强程序功能的技术，泛型类和泛型方法兼具可重用性、类型安全性和效率，这是非泛型类和非泛型方法无法实现的。
>
> ```c#
> using System;
> using System.Collections;
> namespace c.biancheng.net
> {
>     // 定义泛型类
>     class GenericClass<T>{
>         // 泛型方法
>         public GenericClass(T msg){
>             Console.WriteLine(msg);
>         }
>     }
>     class Demo
>     {
>         static void Main(string[] args){
>             GenericClass<string> str_gen = new GenericClass<string>("abcd");
>             GenericClass<int> int_gen = new GenericClass<int>(1234567);
>             GenericClass<char> char_gen = new GenericClass<char>('C');
>             Console.ReadKey();
>         }
>     }
> }
> ```
>
> 泛型委托：
>
> ```c#
> using System;
> using System.Collections.Generic;
> namespace c.biancheng.net
> {
>     class Demo
>     {
>         delegate T NumberChanger<T>(T n);
>         static int num = 10;
>         public static int AddNum(int p){
>             num += p;
>             return num;
>         }
>         public static int MultNum(int q){
>             num *= q;
>             return num;
>         }
>         public static int getNum(){
>             return num;
>         }
>         static void Main(string[] args){
>             // 创建委托实例
>             NumberChanger<int> nc1 = new NumberChanger<int>(AddNum);
>             NumberChanger<int> nc2 = new NumberChanger<int>(MultNum);
>             // 使用委托对象调用方法
>             nc1(25);
>             Console.WriteLine("Num 的值为: {0}", getNum());
>             nc2(5);
>             Console.WriteLine("Num 的值为: {0}", getNum());
>             Console.ReadKey();
>         }
>     }
> }
> ```

### C#和C++的区别

> |              | C#                                                           | C++                                         |
> | ------------ | ------------------------------------------------------------ | ------------------------------------------- |
> | 继承         | 单继承                                                       | 多继承                                      |
> | 指针         | 托管代码不支持指针，需要unsafe                               | 支持指针操作                                |
> | switch..case | 支持string，除非case语句后是空格，否则执行完一个case语句后就算没有break也会停止执行后面的case语句 | 不支持string，没有break的话就会一直往下执行 |
> | 内存管理     | 托管代码有gc                                                 | 自己new自己释放                             |
> | 委托         | 有                                                           | 没有                                        |
> | 函数指针     | 没有                                                         | 有                                          |
> | 装箱拆箱     | 有                                                           | 没有                                        |
> | 速度         | 慢点，因为使用了很多重型库                                   | 快点，不使用重型库                          |