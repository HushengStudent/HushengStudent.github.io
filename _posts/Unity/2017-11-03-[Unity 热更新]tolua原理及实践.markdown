---
layout: post
title:  "[Unity 热更新]tolua原理及实践"
date:   2017-11-03 13:25:10
categories: Unity
comments: true
---

#### 一、概论
1、tolua相比ulua的优势：

①效率更高。

②更加稳定。

③不支持使用反射，只支持wrap方式。

#### 二、基础
[DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]

其中LUADLL对应的字符串就是tolua，在不同的平台上mono会去加载对应的tolua.dll或者tolua.so等文件并调用对应的函数。

①LuaAttribute.cs

在tolua#生成绑定代码时做一些标示使用。

②LuaBaseRef.cs

Lua中对象对应C#中对象的一个基类，主要作用是有一个reference指向lua里面的对象，引用计数判断两个对象是否相等等。

比如LuaFunction里面的reference是指向lua里面的一个闭包的，而LuaTable的reference是指向lua中的一个table的。

③LuaDll.cs

这个类的主要作用就是实现了C#调用原生代码的功能。

④LuaState.cs

这里面是对真正的lua_State的封装，包括初始化lua路径，加载相应的lua文件，注册我们前面生成的绑定代码以及各种辅助函数。

⑤ObjectTranslator.cs

主要意义就是给lua中对C#对象的交互提供了基础，简单来说就是C#中的对象在传给lua时并不是直接把对象暴露给了lua，而是在这个OjbectTranslator里面注册并返回一个索引（可以理解为windows编程中的句柄），并把这个索引包装成一个userdata传递给lua，并且设置元表。具体可以查看tolua_pushnewudata代码。

[参考一](http://www.cnblogs.com/ghl_carmack/p/6720500.html)

弱引用可以让您保持对对象的引用，同时允许GC在必要时释放对象，回收内存。对于那些创建便宜但耗费大量内存的对象，即希望保持该对象，又要在应用程序需要时使用，同时希望GC必要时回收时，可以考虑使用弱引用。

    Object obj = new Object();
    WeakReference wref = new WeakReference( obj );
    obj = null;

第一行代码新建了一个新的对象，这里叫它对象A，obj是对对象A的强引用。接着第二行代码新建了一个弱引用对象，参数就是对象A的强引用，第三行代码释放掉对对象A的强引用。这时如果GC进行回收，对象A就会被回收。

怎样在取得对象A的强引用呢？很简单，请看代码2：

    Object obj2 = wref.Target;
    if( obj2 != null )
    {
       // 做你想做的事
    }
    else
    {
    // 对象已经被回收，如果要用必须新建一个
    }


1、提供Lua-c#值类型、对象类型转化操作交互层。（ObjectTranslator.cs、LuaFunction.cs、LuaTable.cs、ToLua.cs等）。

2、提供Lua虚拟机创建、启动、销毁，Require、DoFile、DoString、Traceback等相关支持。（LuaState.cs、LuaStatic.cs）。

3、提供导出工具，利用c#反射，对指定的c#类生成对应的wrap文件，启动后将所有wrap文件注册到lua虚拟机中。（ToLuaMenu.cs、ToLuaExport.cs、ToLuaTree.cs、LuaBinder.cs、CustomSetting.cs等）。

4、提供c#对象和lua userdata对应关系，使该userdata能访问对应c#对象属性，调用对应c#对象函数。lua支持一定的面向对象(类、继承)。管理这些对象的内存分配与生命周期、GC。(LuaState.cs）。

5、提供支持功能Lua Coroutine、反射等，Lua层重写部分性能有问题对象如Vector系列。（Vector3.lua等）。

LuaState继承自LuaStatePtr，该类包含一个System.IntPtr L指针，即lua虚拟机栈，并提供了一系列LuaDLL API的封装，可以认为是LuaDLL的升级版。而成员属性中比较重要的有ObjectTranslator和LuaReflection，暂时我们只用关注ObjectTranslator。

1、new LuaState()总结

①初始化了LuaState成员属性ObjectTranslator和LuaReflection。

②LuaState构造开始。

③构造中先初始化了LuaException用于异常时抛出，提供了出错信息堆栈的格式化显示。

④以标准的LuaDLL.luaL_newstate语句正式启动lua虚拟机。

⑤luaL_openlibs开启lua基本库，额外地在c层初始了tolua的一些相关内容。

⑥OpenToLuaLibs向lua虚拟机注册ToLua.Print、ToLua.DoFile、Panic等基础c函数。

⑦OpenBaseLibs先向虚拟机注册了一些System和Unity命名空间下的基础类，接着初始化了Mathf、Layer和一些反射相关。

⑧InitLuaPath在LuaFileUtils中存储了lua中package.path、LuaConst中一些路径搜索路径。用于查找文件。

⑨LuaState构造结束。

LuaState.Start()先是DoFile(“tolua.lua”)，而tolua.lua中require了所有tolua重写的一些Unity性能有问题的类型，例如Vector3、Vector2、Bound等。接着在C#缓存了这些类型的一些lua方法。

LuaBinder.cs这个文件是ToLua导出自动生成的，而Bind函数是向lua虚拟机注册所有的wrap类。 

lua调用c#比c#调用lua效率要高。Lua 调用 Wrap , Wrap调用C#的模式，实现了Lua调用C#。

尽最多可能使用lua中的容器table取代c#中的所有容器，因为lua的table是c语言实现，效率比c#最快的容器Dictionary还要快。

安卓平台如果使用luajit的话，记得在lua最开始执行的地方请开启 jit.off()，性能会提升N倍。 

loadassembly  反射

require   非反射

[参考二](http://blog.csdn.net/lodypig/article/details/60160020)

#### 三、示例
    public class HelloWorld : MonoBehaviour
    {
        void Awake()
        {
            LuaState lua = new LuaState();  //创建Lua虚拟机;
            lua.Start();                    //初始化;
            string hello =                  //Lua代码;
                @"                
                    print('hello tolua#')                                  
                ";
            
            lua.DoString(hello, "HelloWorld.cs"); //执行;
            lua.CheckTop();
            lua.Dispose();                  //回收;
            lua = null;
        }
    }


loadfile，加载文件，编译文件，并且返回一个函数，不运行。

dofile，其实就是包装了Loadfile，根据loadfile的返回函数运行一遍。

require，加载文件的时候，不用带目录，有lua自己的搜索加载目录的路径，并且会判断文件是否加载过，加载过则不加载。

dofile与require都是会执行里面的代码，区别是require只加载一次，dofile每次加载,loadfile只加载文件而不执行。

在lua文件里面，推荐的是requrie加载文件。

#### 四、LuaFramework
启动游戏：

加载Manager；

```csharp
AppFacade.Instance.AddManager<LuaManager>(ManagerName.Lua);
AppFacade.Instance.AddManager<PanelManager>(ManagerName.Panel);
AppFacade.Instance.AddManager<SoundManager>(ManagerName.Sound);
AppFacade.Instance.AddManager<TimerManager>(ManagerName.Timer);
AppFacade.Instance.AddManager<NetworkManager>(ManagerName.Network);
AppFacade.Instance.AddManager<ResourceManager>(ManagerName.Resource);
AppFacade.Instance.AddManager<ThreadManager>(ManagerName.Thread);
AppFacade.Instance.AddManager<ObjectPoolManager>(ManagerName.ObjectPool);
AppFacade.Instance.AddManager<GameManager>(ManagerName.Game);
```

LuaManager：
```lua
--主入口函数。从这里开始lua逻辑
function Main()					
	print("logic start")--[LuaManager 开始执行Main.Lua--]
end

--场景切换通知
function OnLevelWasLoaded(level)
	collectgarbage("collect")   
	Time.timeSinceLevelLoad = 0
end
```

collectgarbage("collect")：运行一个完整的垃圾回收周期。

collectgarbage("count")：返回当前程序使用的内存总量，以 KB 为单位。

collectgarbage("restart")：如果垃圾回收器停止，则重新运行它。

collectgarbage("setpause")：设置垃圾收集暂停时间变量的值，值由第二个参数指出（第二参数的值除以 100 后赋予变量）。稍后，我们将
详细讨论它的用法。

collectgarbage("setsetmul")：设置垃圾收集器步长倍增器的值，第二个参数的含义与上同。

collectgarbage("step")：进行一次垃圾回收迭代。第二个参数值越大，一次迭代的时间越长；如果本次迭代是垃圾回收的最后一次迭代则此函数返回 true。

collectgarbage("stop")：停止垃圾收集器运行。

GameManager：

```csharp
LuaManager.DoFile("Logic/Game");            //加载游戏
LuaManager.DoFile("Logic/Network");         //加载网络
NetManager.OnInit();                        //初始化网络

Util.CallMethod("Game", "OnInitOK");          //初始化完成
```

Game.Lua
```lua
--初始化完成，发送链接服务器信息--
function Game.OnInitOK()
    AppConst.SocketPort = 2012;
    AppConst.SocketAddress = "127.0.0.1";
    networkMgr:SendConnect();

    --注册LuaView--
    this.InitViewPanels();

    --测试第三方库功能--
    this.test_class_func();
    this.test_pblua_func();
    this.test_cjson_func();
    this.test_pbc_func();
    this.test_lpeg_func();
    this.test_sproto_func();
    coroutine.start(this.test_coroutine);

    CtrlManager.Init();  --初始化CtrlManager--
    local ctrl = CtrlManager.GetCtrl(CtrlNames.Prompt); --获取Prompt--
    if ctrl ~= nil and AppConst.ExampleMode == 1 then
        ctrl:Awake();          --调用ctrl的Awake方法--
    end
    logWarn('LuaFramework InitOK--->>>');
end
```

相关引用在define.lua中：
```lua

CtrlNames = {
	Prompt = "PromptCtrl",
	Message = "MessageCtrl"
}

PanelNames = {
	"PromptPanel",	
	"MessagePanel",
}

--协议类型--
ProtocalType = {
	BINARY = 0,
	PB_LUA = 1,
	PBC = 2,
	SPROTO = 3,
}
--当前使用的协议类型--
TestProtoType = ProtocalType.BINARY;

Util = LuaFramework.Util;
AppConst = LuaFramework.AppConst;
LuaHelper = LuaFramework.LuaHelper;
ByteBuffer = LuaFramework.ByteBuffer;

resMgr = LuaHelper.GetResManager();
panelMgr = LuaHelper.GetPanelManager();
soundMgr = LuaHelper.GetSoundManager();
networkMgr = LuaHelper.GetNetManager();

WWW = UnityEngine.WWW;
GameObject = UnityEngine.GameObject;

```

我们可以看到，networkMgr是通过LuaHelper注册的wrap文件执行LuaHelper中的方法获得的。

PanelManager：

```csharp
IEnumerator StartCreatePanel(string name, LuaFunction func = null) {
            AssetBundle bundle = ResManager.LoadBundle(name);

            name += "Panel";
            GameObject prefab = null;
#if UNITY_5
            prefab = bundle.LoadAsset(name, typeof(GameObject)) as GameObject;
#else
            prefab = bundle.Load(name, typeof(GameObject)) as GameObject;
#endif
            yield return new WaitForEndOfFrame();

            if (Parent.FindChild(name) != null || prefab == null) {
                yield break;
            }
            GameObject go = Instantiate(prefab) as GameObject;
            go.name = name;
            go.layer = LayerMask.NameToLayer("Default");
            go.transform.parent = Parent;
            go.transform.localScale = Vector3.one;
            go.transform.localPosition = Vector3.zero;

            yield return new WaitForEndOfFrame();
            go.AddComponent<LuaBehaviour>().OnInit(bundle); //加上LuaBehaviour组件，执行对于模块的Awake，Start...

            if (func != null) func.Call(go);  //完成加载并传递回Lua
            Debug.Log("StartCreatePanel------>>>>" + name);
        }
```