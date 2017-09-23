---
layout: post
title:  "[C#]Learning hard C#学习笔记"
date:   2016-04-25 02:55:10
categories: 41---【C#】
comments: true
---

委托：
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace DelegateUse  
{  
    // 委托使用演示  
    class Program  
    {  
        // 1. 使用delegate关键字来定义一个委托类型  
        delegate void MyDelegate(int para1, int para2);  
        
        static void Main(string[] args)  
        {  
            // 2. 声明委托变量d  
            MyDelegate d;  
            // 3. 实例化委托类型,传递的方法也可以为静态方法，这里传递的是实例方法  
            d = new MyDelegate(new Program().Add);  
  
            // 4. 委托类型作为参数传递给另一个方法  
            MyMethod(d);  
            Console.Read();  
        }  
  
        // 该方法的定义必须与委托定义相同，即返回类型为void, 两个int类型的参数  
        void Add(int para1, int para2)  
        {  
            int sum = para1 + para2;  
            Console.WriteLine("两个数的和为:"+sum);  
        }  
  
        // 方法的参数是委托类型  
        private static void MyMethod(MyDelegate mydelegate)  
        {  
            // 5.在方法中调用委托  
            mydelegate(1,2);  
        }  
    }  
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace DelegateOrigin  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 引入委托之后  
            // 初始化对象  
            Program p = new Program();  
            p.Greeting("李志", p.ChineseGreeting);  
            p.Greeting("Tommy Li", p.EnglishGreeting);  
            Console.Read();  
        }  
        #region 引入委托之后  
        // 定义委托类型  
        public delegate void GreetingDelegate(string name);  
        // 有了委托之后，可以像如下实现打招呼方法  
        public void Greeting(string name, GreetingDelegate callback)  
        {  
            // 调用委托  
            callback(name);  
        }  
  
        // 英国人打招呼方法  
        public void EnglishGreeting(string name)  
        {  
            Console.WriteLine("Hello,  " + name);  
        }  
  
        // 中国人打招呼方法  
        public void ChineseGreeting(string name)  
        {  
            Console.WriteLine("你好, " + name);  
        }  
        public void JapaneseGreeting(string name)  
        {  
            Console.WriteLine("こんにちは, "+name);  
        }  
        #endregion   
 
        #region 没有委托的情况  
        //// 不使用委托实现打招呼方法  
        //public void Greeting(string name,string language)  
        //{  
        //    switch (language)  
        //    {  
        //        case "zh-cn":  
        //            ChineseGreeting(name);  
        //            break;  
        //        case "en-us":  
        //            EnglishGreeting(name);  
        //            break;  
        //        default:  
        //            EnglishGreeting(name);  
        //            break;  
        //    }  
        //}  
  
        //// 英国人打招呼方法  
        //public void EnglishGreeting(string name)  
        //{  
        //    Console.WriteLine("Hello,  "+name);  
        //}  
  
        //// 中国人打招呼方法  
        //public void ChineseGreeting(string name)  
        //{  
        //    Console.WriteLine("你好, "+name);  
        //}  
        #endregion  
    }   
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace MulticastDelegates  
{  
    class Program  
    {  
        // 声明一个委托类型  
        public delegate void DelegateTest();  
        static void Main(string[] args)  
        {  
            // 用静态方法来实例化委托  
            DelegateTest dtstatic = new DelegateTest(Program.method1);  
  
            DelegateTest dtinstance= new DelegateTest(new Program().method2);  
              
            // 定义一个委托对象，一开始初始化为null，就是不代表任何方法（我就是我，我不代表任何人）  
            DelegateTest delegatechain = null;  
  
            // 使用+符号链接委托，链接多个委托后就成为委托链了  
            delegatechain += dtstatic;  
            delegatechain += dtinstance;  
  
            // 使用-运算符把dtstatic委托从委托链中移除  
            delegatechain -= dtstatic;  
  
            // 调用委托链  
            delegatechain();  
            Console.Read();  
        }  
  
        // 静态方法  
        private static void method1()  
        {  
            Console.WriteLine( "这是静态方法");  
        }  
  
        // 实例方法  
        private void method2()  
        {  
            Console.WriteLine( "这是实例方法");  
        }  
    }  
}  
```

事件：
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace EventUse  
{  
    // 新郎官类，充当事件发布者角色  
    public class Bridegroom  
    {  
        // 自定义委托  
        public delegate void MarryHandler(string msg);  
  
        // 使用自定义委托类型定义事件，事件名为BirthdayEvent  
        public event MarryHandler MarryEvent;  
  
        // 发出事件  
        public void OnBirthdayComing(string msg)  
        {  
            // 判断是否绑定了事件处理方法  
            if (MarryEvent != null)  
            {  
                // 触发事件  
                MarryEvent(msg);  
            }  
        }  
  
        static void Main(string[] args)  
        {  
            //初始化新郎官对象  
            Bridegroom bridegroom =new Bridegroom();  
  
            // 实例化朋友对象  
            Friend friend1 = new Friend("张三");  
            Friend friend2= new Friend("李四");  
            Friend friend3 = new Friend("王五");  
  
            // 使用+=订阅事件  
            bridegroom.MarryEvent+=new MarryHandler(friend1.SendMessage);  
            bridegroom.MarryEvent += new MarryHandler(friend2.SendMessage);  
  
            // 发出通知,此时订阅了事件的对象才能收到通知  
            bridegroom.OnBirthdayComing("朋友们，我结婚了，到时候准时参加婚礼!");  
            Console.WriteLine("------------------------------------");  
  
            // 使用-=取消订阅事件,此时李四将收不到通知  
            bridegroom.MarryEvent -= new MarryHandler(friend2.SendMessage);  
  
            // 使用+=订阅事件，此时王五可以收到通知  
            bridegroom.MarryEvent += new MarryHandler(friend3.SendMessage);  
  
            // 发出通知  
            bridegroom.OnBirthdayComing("朋友们，我结婚了，到时候准时参加婚礼!");  
            Console.Read();  
        }  
    }  
  
    // 朋友类  
    public class Friend  
    {  
        // 字段  
        public string Name;  
  
        // 构造函数  
        public Friend(string name)  
        {  
            Name = name;  
        }  
  
        // 事件处理函数,该函数需要符合MarryHandler委托的定义  
        public void SendMessage(string mssage)  
        {  
            // 输出通知信息  
            Console.WriteLine(mssage);  
  
            // 对事件做出处理  
            Console.WriteLine(this.Name + "收到了，到时候准时参加");  
        }  
    }  
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace EventUseInNET  
{  
    // 新郎官类，充当事件发布者角色  
    public class Bridegroom  
    {  
        // 使用.NET类库中预定义的委托类型EventHandler来定义事件  
        public event EventHandler MarryEvent;  
  
        // 发出事件  
        public void OnBirthdayComing(string msg)  
        {  
            // 判断是否绑定了事件处理方法  
            if (MarryEvent != null)  
            {  
                Console.WriteLine(msg);  
  
                // 触发事件  
                MarryEvent(this,new EventArgs());  
            }  
        }  
  
        static void Main(string[] args)  
        {  
            //初始化新郎官对象  
            Bridegroom bridegroom = new Bridegroom();  
  
            // 实例化朋友对象  
            Friend friend1 = new Friend("张三");  
            Friend friend2 = new Friend("李四");  
            Friend friend3 = new Friend("王五");  
  
            // 使用+=订阅事件  
            bridegroom.MarryEvent += new EventHandler(friend1.SendMessage);  
            bridegroom.MarryEvent += new EventHandler(friend2.SendMessage);  
  
            // 发出通知,此时订阅了事件的对象才能收到通知  
            bridegroom.OnBirthdayComing("朋友们，我结婚了，到时候准时参加婚礼!");  
            Console.WriteLine("------------------------------------");  
  
            // 使用-=取消订阅事件,此时李四将收不到通知  
            bridegroom.MarryEvent -= new EventHandler(friend2.SendMessage);  
  
            // 使用+=订阅事件，此时王五可以收到通知  
            bridegroom.MarryEvent += new EventHandler(friend3.SendMessage);  
  
            // 发出通知  
            bridegroom.OnBirthdayComing("朋友们，我结婚了，到时候准时参加婚礼!");  
            Console.Read();  
        }  
    }  
  
    // 朋友类  
    public class Friend  
    {  
        // 字段  
        public string Name;  
  
        // 构造函数  
        public Friend(string name)  
        {  
            Name = name;  
        }  
  
        // 事件处理函数，该函数需要符合EventHandler委托的定义  
        public void SendMessage(object s, EventArgs e)  
        {  
            // 对事件做出处理  
            Console.WriteLine(this.Name + "收到了，到时候准时参加");  
        }  
    }  
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace EventArgsExt  
{  
    // 自定义事件类，使其带有事件数据  
    public class MarryEventArgs : EventArgs  
    {  
        public string Message;  
        public MarryEventArgs(string msg)  
        {  
            Message = msg;  
        }  
    }  
  
    // 新郎官类，充当事件发布者角色  
    public class Bridegroom  
    {  
        // 自定义委托类型，委托具有两个参数  
        public delegate void MarryHandler(object sender, MarryEventArgs e);  
  
        // 定义事件  
        public event MarryHandler MarryEvent;  
  
        // 发出事件  
        public void OnBirthdayComing(string msg)  
        {  
            // 判断是否绑定了事件处理方法  
            if (MarryEvent != null)  
            {  
                // 触发事件  
                MarryEvent(this, new MarryEventArgs(msg));  
            }  
        }  
  
        static void Main(string[] args)  
        {  
            //初始化新郎官对象  
            Bridegroom bridegroom = new Bridegroom();  
  
            // 实例化朋友对象  
            Friend friend1 = new Friend("张三");  
            Friend friend2 = new Friend("李四");  
            Friend friend3 = new Friend("王五");  
  
            // 使用+=订阅事件  
            bridegroom.MarryEvent += new MarryHandler(friend1.SendMessage);  
            bridegroom.MarryEvent += new MarryHandler(friend2.SendMessage);  
  
            // 发出通知,此时订阅了事件的对象才能收到通知  
            bridegroom.OnBirthdayComing("朋友们，我结婚了，到时候准时参加婚礼!");  
            Console.WriteLine("------------------------------------");  
  
            // 使用-=取消订阅事件,此时李四将收不到通知  
            bridegroom.MarryEvent -= new MarryHandler(friend2.SendMessage);  
  
            // 使用+=订阅事件，此时王五可以收到通知  
            bridegroom.MarryEvent += new MarryHandler(friend3.SendMessage);  
  
            // 发出通知  
            bridegroom.OnBirthdayComing("朋友们，我结婚了，到时候准时参加婚礼!");  
            Console.Read();  
        }  
    }  
  
    // 朋友类  
    public class Friend  
    {  
        // 字段  
        public string Name;  
  
        // 构造函数  
        public Friend(string name)  
        {  
            Name = name;  
        }  
  
        // 事件处理函数，该函数需要符合MarryHandler委托的定义  
        public void SendMessage(object s, MarryEventArgs e)  
        {  
            // 输出通知信息  
            Console.WriteLine(e.Message);  
  
            // 对事件做出处理  
            Console.WriteLine(this.Name + "收到了，到时候准时参加");  
        }  
    }  
}  
```

泛型：
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace _11._3__3  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 每个封闭泛型类型都会调用其构造函数  
            GenericClass<int>.Print();  
            GenericClass<string>.Print();  
  
            // 对于非泛型类型，其构造函数只会调用一次  
            NonGenericClass.Print();  
            NonGenericClass.Print();  
            Console.Read();  
        }  
    }  
  
    // 泛型类  
    public static class GenericClass<T>  
    {  
        // 静态构造函数  
        static GenericClass()  
        {  
            Console.WriteLine("泛型的静态构造函数被调用了,实际类型为："+typeof(T));  
        }  
  
        // 静态方法  
        public static void Print()  
        {  
        }  
    }  
    // 非泛型类  
    public static class NonGenericClass  
    {  
        static NonGenericClass()  
        {  
            Console.WriteLine("非泛型的静态构造函数被调用了");  
        }  
  
        public static void Print()  
        {  
        }  
    }  
}  
```

可空类型、匿名方法、迭代器：
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace _12._1  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 下面代码也可以这样子定义int? value=1;  
            Nullable<int> value = 1;  
  
            Console.WriteLine("可空类型有值的输出情况：");  
            Display(value);  
            Console.WriteLine();  
            Console.WriteLine();  
  
            value = new Nullable<int>();  
            Console.WriteLine("可空类型没有值的输出情况：");  
            Display(value);  
            Console.Read();  
        }  
  
        // 输出方法，演示可空类型中的方法和属性的使用  
        private static void Display(int? nullable)  
        {  
            // HasValue 属性代表指示可空对象是否有值  
            Console.WriteLine("可空类型是否有值：{0}", nullable.HasValue);  
            if (nullable.HasValue)  
            {  
                Console.WriteLine("值为: {0}", nullable.Value);  
            }  
  
            // GetValueOrDefault(代表如果可空对象有值,就用它的值返回,如果可空对象不包含值时,使用默认值0返回)相当与下面的语句  
            // if (!nullable.HasValue)  
            // {  
            //    result = d.Value;  
            // }  
  
            Console.WriteLine("GetValueorDefault():{0}", nullable.GetValueOrDefault());  
  
            // GetValueOrDefault(T)方法代表如果 HasValue 属性为 true，则为 Value 属性的值；否则为 defaultValue 参数值,即2。  
            Console.WriteLine("GetValueorDefalut重载方法使用：{0}", nullable.GetValueOrDefault(2));  
  
            // GetHashCode()代表如果 HasValue 属性为 true，则为 Value 属性返回的对象的哈希代码；如果 HasValue 属性为 false，则为零  
            Console.WriteLine("GetHashCode()方法的使用：{0}", nullable.GetHashCode());  
        }  
    }  
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace _12._2  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            Console.WriteLine("??运算符的使用如下：");  
            NullcoalescingOperator();  
            Console.Read();  
        }  
        // ??运算符的演示  
        private static void NullcoalescingOperator()  
        {  
            int? nullable = null;  
            int? nullhasvalue = 1;  
  
            // ??和三目运算符的功能差不多的  
            // 所以下面代码等价于：  
            // x=nullable.HasValue?b.Value:12;  
            int x = nullable ?? 12;  
  
            // 此时nullhasvalue不能null,所以y的值为nullhasvalue.Value,即输出1  
            int y = nullhasvalue ?? 123;  
            Console.WriteLine("可空类型没有值的情况：{0}", x);  
            Console.WriteLine("可空类型有值的情况：{0}", y);  
  
            // 同时??运算符也可以用于引用类型， 下面是引用类型的例子  
            Console.WriteLine();  
            string stringnotnull = "123";  
            string stringisnull = null;  
  
            // 下面的代码等价于：  
            // (stringnotnull ==null)? "456" :stringnotnull  
            // 也也等价于：  
            // if(stringnotnull==null)  
            // {  
            //      return "456";  
            // }  
            // else  
            // {  
            //      return stringnotnull;  
            // }  
  
            // 从上面的等价代码可以看出，有了??运算符之后可以省略大量的if—else语句，这样代码少了， 自然可读性就高了  
            string result = stringnotnull ?? "456";  
            string result2 = stringisnull ?? "12";  
            Console.WriteLine("引用类型不为null的情况：{0}", result);  
            Console.WriteLine("引用类型为null的情况：{0}", result2);  
        }  
    }  
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace _12._3  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            Console.WriteLine("可空类型的装箱和拆箱的使用如下：");  
            BoxedandUnboxed();  
            Console.Read();  
        }  
  
        // 可空类型装箱和拆箱的演示  
        private static void BoxedandUnboxed()  
        {  
            // 定义一个可空类型对象nullable  
            Nullable<int> nullable = 5;  
            int? nullablewithoutvalue = null;  
             
            Console.Write(nullablewithoutvalue.Value);  
            // 获得可空对象的类型，此时返回的是System.Int32,而不是System.Nullable<System.Int32>,这点大家要特别注意下的  
            Console.WriteLine("获取不为null的可空类型的类型为：{0}", nullable.GetType());  
  
            // 对于一个为null的类型调用方法时出现异常，所以一般对于引用类型的调用方法前，最好养成习惯先检测下它是否为null  
            //Console.WriteLine("获取为null的可空类型的类型为：{0}", nullablewithoutvalue.GetType());  
  
            // 将可空类型对象赋给引用类型obj,此时会发生装箱操作，大家可以通过IL中的boxed 来证明  
            object obj = nullable;  
  
            // 获得装箱后引用类型的类型，此时输出的仍然是System.Int32,而不是System.Nullable<System.Int32>  
            Console.WriteLine("获得装箱后obj 的类型：{0}", obj.GetType());  
  
            // 拆箱成非可空变量  
            int value = (int)obj;  
            Console.WriteLine("拆箱成非可空变量的情况为：{0}", value);  
  
            // 拆箱成可空变量  
            nullable = (int?)obj;  
            Console.WriteLine("拆箱成可空变量的情况为：{0}", nullable);  
  
            // 装箱一个没有值的可空类型的对象  
            obj = nullablewithoutvalue;  
            Console.WriteLine("对null的可空类型装箱后obj 是否为null：{0}", obj == null);  
  
            // 拆箱成非可空变量,此时会抛出NullReferenceException异常,因为没有值的可空类型装箱后obj等于null,引用一个空地址  
            // 相当于拆箱后把null值赋给一个int 类型的变量,此时当然就会出现错误了  
            //value = (int)obj;  
            //Console.WriteLine("一个没有值的可空类型装箱后，拆箱成非可空变量的情况为：{0}", value);  
  
            // 拆箱成可空变量  
            nullable = (int?)obj;  
            Console.WriteLine("一个没有值的可空类型装箱后，拆箱成可空变量是否为null：{0}", nullable == null);  
        }  
    }  
}  
```
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
using System.Collections;  
  
namespace _12._9  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 创建一个对象  
            Friends friendcollection = new Friends();  
            // Friends实现了IEnumerable所以可以使用foreach语句进行遍历  
            foreach (Friend f in friendcollection)  
            {  
                Console.WriteLine(f.Name);  
            }  
  
            Console.Read();  
        }  
  
        // 朋友类  
        public class Friend  
        {  
            private string name;  
            public string Name  
            {  
                get { return name; }  
                set { name = value; }  
            }  
            public Friend(string name)  
            {  
                this.name = name;  
            }  
        }  
  
        // 朋友集合  
        public class Friends : IEnumerable  
        {  
            private Friend[] friendarray;  
  
            public Friends()  
            {  
                friendarray = new Friend[]  
            {  
                new Friend("张三"),  
                new Friend("李四"),  
                new Friend("王五")  
            };  
            }  
  
            // 索引器  
            public Friend this[int index]  
            {  
                get { return friendarray[index]; }  
            }  
  
            public int Count  
            {  
                get { return friendarray.Length; }  
            }  
  
            // 实现IEnumerable<T>接口方法  
            public IEnumerator GetEnumerator()  
            {  
                return new FriendIterator(this);  
            }  
        }  
  
        //  C#1.0中实现迭代器代码——必须实现 IEnumerator接口  
        public class FriendIterator : IEnumerator  
        {  
            private readonly Friends friends;  
            private int index;  
            private Friend current;  
            internal FriendIterator(Friends friendcollection)  
            {  
                this.friends = friendcollection;  
                index = 0;  
            }  
  
            // 实现IEnumerator接口中的方法  
            public object Current  
            {  
                get  
                {  
                    return this.current;  
                }  
            }  
  
            public bool MoveNext()  
            {  
                if (index + 1 > friends.Count)  
                {  
                    return false;  
                }  
                else  
                {  
                    this.current = friends[index];  
                    index++;  
                    return true;  
                }  
            }  
  
            public void Reset()  
            {  
                index = 0;  
            }  
        }  
    }  
}  
```

Lambda表达式：
```java
using System;  
using System.Collections.Generic;  
using System.Linq;  
using System.Text;  
  
namespace _14._1  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // Lambda表达式的演变过程  
            // 下面是C# 1中创建委托实例的代码  
            Func<string, int> delegatetest1 = new Func<string, int>(Callbackmethod);  
  
            //                                  ↓  
            // C# 2中用匿名方法来创建委托实例，此时就不需要额外定义回调方法Callbackmethod  
            Func<string, int> delegatetest2 = delegate(string text)  
            {  
                return text.Length;  
            };  
            //                      ↓  
            // C# 3中使用Lambda表达式来创建委托实例  
            Func<string, int> delegatetest3 = (string text) => text.Length;  
  
            //                                  ↓  
            // 可以省略参数类型string,把上面代码再简化为：  
            Func<string, int> delegatetest4 = (text) => text.Length;  
  
            //                                  ↓  
            // 此时可以把圆括号也省略,简化为：  
            Func<string, int> delegatetest = text => text.Length;  
  
            // 调用委托  
            Console.WriteLine("使用Lambda表达式返回字符串的长度为： " + delegatetest("learning hard"));  
            Console.Read();  
        }  
  
        // 回调方法  
        private static int Callbackmethod(string text)  
        {  
            return text.Length;  
        }  
    }  
}  
```
```java
using System;  
using System.Windows.Forms;  
  
namespace _14._2  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 新建一个button实例  
            Button button1 = new Button();  
            button1.Text = "点击我";  
  
            // C# 2中使用匿名方法来订阅事件  
            button1.Click += delegate(object sender, EventArgs e)  
            {  
                ReportEvent("Click事件", sender, e);  
            };  
            button1.KeyPress += delegate(object sender, KeyPressEventArgs e)  
            {  
                ReportEvent("KeyPress事件，即键盘按下事件", sender, e);  
            };  
  
            // C# 3之前初始化对象时使用下面代码  
            Form form = new Form();  
            form.Name = "在控制台中创建的窗体";  
            form.AutoSize = true;  
            form.Controls.Add(button1);  
            // 运行窗体  
            Application.Run(form);  
        }  
  
        // 记录事件的回调方法  
        private static void ReportEvent(string title, object sender, EventArgs e)  
        {  
            Console.WriteLine("发生的事件为：{0}", title);  
            Console.WriteLine("发生事件的对象为：{0}", sender);  
            Console.WriteLine("发生事件参数为： {0}", e.GetType());  
            Console.WriteLine();  
            Console.WriteLine();  
        }  
    }  
}  
```
```java
using System;  
// 引入Expression<TDelegate>类的命名空间  
using System.Linq.Expressions;  
  
namespace _14._6  
{  
    class Program  
    {  
        // 构造a+b表达式树结构  
        static void Main(string[] args)  
        {  
            // 表达式的参数  
            ParameterExpression a = Expression.Parameter(typeof(int), "a");  
            ParameterExpression b = Expression.Parameter(typeof(int), "b");  
              
            // 表达式树主体部分  
            BinaryExpression be = Expression.Add(a, b);  
  
            // 构造表达式树  
            Expression<Func<int, int, int>> expressionTree = Expression.Lambda<Func<int, int, int>>(be, a, b);  
  
            // 分析树结构，获取表达式树的主体部分  
            BinaryExpression body = (BinaryExpression)expressionTree.Body;  
  
            // 左节点,每个节点本身就是一个表达式对象  
            ParameterExpression left = (ParameterExpression)body.Left;  
  
            // 右节点  
            ParameterExpression right = (ParameterExpression)body.Right;  
  
            // 输出表达式树结构  
            Console.WriteLine("表达式树结构为："+expressionTree);  
            // 输出  
            Console.WriteLine("表达式树主体为：");  
            Console.WriteLine(expressionTree.Body);  
            Console.WriteLine("表达式树左节点为：{0}{4} 节点类型为：{1}{4}{4} 表达式树右节点为：{2}{4} 节点类型为：{3}{4}", left.Name, left.Type, right.Name, right.Type, Environment.NewLine);  
              
            Console.Read();  
        }  
    }  
}  
```
```java
using System;  
// 引入Expression<TDelegate>类的命名空间  
using System.Linq.Expressions;  
  
namespace _14._4  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 将lambda表达式构造成表达式树  
            Expression<Func<int, int, int>> expressionTree = (a, b) => a + b;  
  
            // 获得表达式树参数  
            Console.WriteLine("参数1：{0},参数2: {1}", expressionTree.Parameters[0], expressionTree.Parameters[1]);  
            // 获取表达式树的主体部分  
            BinaryExpression body = (BinaryExpression)expressionTree.Body;  
  
            // 左节点,每个节点本身就是一个表达式对象  
            ParameterExpression left = (ParameterExpression)body.Left;  
  
            // 右节点  
            ParameterExpression right = (ParameterExpression)body.Right;  
  
            Console.WriteLine("表达式树主体为：");  
            Console.WriteLine(expressionTree.Body);  
            Console.WriteLine("表达式树左节点为：{0}{4} 节点类型为：{1}{4}{4} 表达式树右节点为：{2}{4} 节点类型为：{3}{4}", left.Name, left.Type, right.Name, right.Type, Environment.NewLine);  
            Console.Read();  
        }  
    }  
}  
```
```java
using System;  
// 引入Expression<TDelegate>类的命名空间  
using System.Linq.Expressions;  
  
namespace _14._5  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            // 将lambda表达式构造成表达式树  
            Expression<Func<int, int, int>> expressionTree = (a, b) => a + b;  
            // 通过调用Compile方法来生成Lambda表达式的委托  
            Func<int,int,int> delinstance =expressionTree.Compile();  
            // 调用委托实例获得结果  
            int result = delinstance(2, 3);  
            Console.WriteLine("2和3的和为：" + result);  
            Console.Read();  
        }  
    }  
}  
```

动态类型：
```java
using System;  
using System.Dynamic;  
  
namespace _18._1  
{  
    class Program  
    {  
        static void Main(string[] args)  
        {  
            dynamic expand = new ExpandoObject();  
            // 动态绑定成员  
            expand.Name = "Learning Hard";  
            expand.Age = 24;  
            expand.Addmethod = (Func<int, int>)(x => x + 1);  
            // 调用类型成员  
            Console.WriteLine("expand类型的姓名为：" + expand.Name + " 年龄为： " + expand.Age);  
            Console.WriteLine("调用expand类型的动态绑定的方法：" + expand.Addmethod(5));  
            Console.Read();  
        }  
    }  
}  
```