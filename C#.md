C#

1. C#事件和委托的区别：事件基于委托，提供了一种发布/订阅机制，只提供了+=和-=操作。事件声明是：

   ```c#
   public delegate int EventHandler(int x);
   public event EventHandler Event;
   ```

2. C#的三个预定义委托：

   ```c#
   // 接受0-16个输入，不带返回值
   Action<int, string> myAction = (x, y)=>{
   
   };
   myAction(10, "Hello");
   
   // 接受0-16个输入，带一个返回值
   Func<int, int, int> myFunc = (x, y)=>{
       return x+y;
   }
   int z = myFunc(10, 20);
   
   // 只接受一个输入返回一个bool
   Predicate<int> myPredicate = (x)=>{
       return x > 0;
   }
   bool isPositive = myPredicate(1);
   ```

3. 反射：

   1. 把程序集加载到应用程序域中：
      1. assembly Assembly.LoadFrom
      2. Assembly.LoadFile
      3. Assembly.Load
   2. 获取类：
      1. Type[] types = assembly.GetTypes();
      2. Type type = assembly.GetType("...");
   3. 获取方法：
      1. MethodInfo method = type.GetMethod("...");
   4. 获取成员：
      1. FieldInfo right = type.GetField("...");
   5. 实例化：
      1. Object obj = Activator.CreateInstance(type);