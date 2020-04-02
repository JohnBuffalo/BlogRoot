# 从零搭建手游xlua环境

本文涉及到的xlua知识点都可以在[xlua教程]("https://github.com/Tencent/xLua")找到.我希望通过这篇文章,能够给出一个通用的模板来搭建手游xlua开发环境.环境搭建大致可分为以下几个步骤:

1. 导入xlua
2. 自定义lualoader
3. 增删第三方库
4. 打包环境配置

## 1. 导入xlua
>1. 从[xlua](https://github.com/Tencent/xLua)下载工程文件.
>2. 新建一个空的unity工程,将xlua工程导入
## 2. 自定义lualoader
> 这个loader的作用是,当调用require方法时,从哪个路径去加载module.
> 每个项目中都应该去定义这个loader,因为如果不自定义这个loader的话,xlua只能从同目录下的Resource文件夹加载lua文件,同时由于Resource只能支持有限的数据格式,所以需要在lua文件添加.txt后缀.
>
xlua定义loader很简单,只涉及到一个简单的接口:
```c#
public delegate byte[] CustomLoader(ref string filepath);
public void LuaEnv.Addloader(CustomLoader loader);
//example
LuaEnv luaenv = new LuaEnv();
LuaLoader lualoader = new LuaLoader();
luaenv.Addloader(lualoader.LoadLuaScript);
//example-lualoader
readonly string PATH_LUA_TEXT = Application.dataPath + "/Scripts/Lua";
public byte[] LoadLuaScript(ref string filepath){
    byte[] luaBytes = null;
    string luaPath;
    //根据平台的不同,需要定义不同的路径
    #if UNITY_EDITOR_OSX
        luaPath = PATH_LUA_TEXT+"/"+filepath+".lua";
        if(File.Exists(luaPath)){
            luaBytes = File.ReadAllBytes(luaPath);
        }
    #endif
    return luaBytes;
}
```
## 3 增删第三方库
1. 准备库的源文件
>- 完成xlua导入后,在与Assets同级的目录下找到build文件夹
>- 本文以xlua官方教程,导入rapidjson为例
>- 将rapidjson的头文件放在 $UnityProj\build\lua-rapidjson\include文件夹
>- 将rapidjson的源代码放在 $UnityProj\build\lua-rapidjson\source文件夹
2. 在CMakeLists.txt加入扩展
>- xlua的各平台Plugins编译使用cmake编译,好处是所有平台的配置都在统一的文件中,且大部分逻辑都是跨平台的
>- xlua配套的CMakeLists.txt为第三方扩展提供了扩展点(都是list)
>- THIRDPART_INC：第三方扩展的头文件搜索路径。
>- THIRDPART_SRC：第三方扩展的源代码。
>- THIRDPART_LIB：第三方扩展依赖的库。
```cmake
 #begin lua-rapidjson
 set (RAPIDJSON_SRC lua-rapidjson/source/rapidjson.cpp)
 set_property(
     SOURCE ${RAPIDJSON_SRC}
     APPEND
     PROPERTY COMPILE_DEFINITIONS
     LUA_LIB
 )
 list(APPEND THIRDPART_INC  lua-rapidjson/include)
 set (THIRDPART_SRC ${THIRDPART_SRC} ${RAPIDJSON_SRC})
 #end lua-rapidjson
```
3. 使用命令行运行xlua自带的编译脚本
>需要注意的是, 每个平台都需要单独编译,而且编译的版本应该是lua53,而非luajit,因为luajit已经停止维护了.
>以编译osx平台为例, 在$xlua路径下输入以下命令即可开始编译
```ninux
$ ./make_osx_lua53.sh
```
4. 编译成功后,会生成一个xlua.bundle文件,路径为 $ build/plugin_lua53/Plugins, 用这个文件替换掉工程中 Assets/Plugins/xlua.bundle文件即可
5. IOS的编译和osx类似,只是执行的脚本为 make_ios_lua53.sh

# 先留一个坑给android的编译, 还在下载android studio

6. c#侧集成
所有lua的C扩展库都会提供个luaopen_xxx的函数，xxx是动态库的名字，比如lua-rapidjson库该函数是luaopen_rapidjson
```cpp
extern "C" {
LUALIB_API int luaopen_rapidjson(lua_State* L)
{
        lua_newtable(L); // [rapidjson]

        luax::setfuncs(L, methods); // [rapidjson]

        lua_pushliteral(L, "rapidjson"); // [rapidjson, name]
        lua_setfield(L, -2, "_NAME"); // [rapidjson]

        lua_pushliteral(L, LUA_RAPIDJSON_VERSION); // [rapidjson, version]
        lua_setfield(L, -2, "_VERSION"); // [rapidjson]

        lua_getfield(L, -1, "null"); // [rapidjson, json.null]
        values::nullref = luaL_ref(L, LUA_REGISTRYINDEX); // [rapidjson]


        createSharedMeta(L, "json.object", "object");
        createSharedMeta(L, "json.array", "array");

        Userdata<Document>::luaopen(L);
        Userdata<SchemaDocument>::luaopen(L);
        Userdata<SchemaValidator>::luaopen(L);

        return 1;
    }
}
```
由于ios的限制,我们无法加载动态库,所以需要直接编译到进程里面去.
为此,LuaEnv中提供了一个API:
```c#
public void AddBuildin(string name, LuaCSFunction initer)
{
    if (!Utils.IsStaticPInvokeCSFunction(initer))
    {
        throw new Exception("initer must be static and has MonoPInvokeCallback Attribute!");
    }
    buildin_initer.Add(name, initer);
}
```
>- @param name : Lua代码中require的module名
>- @param initer : 原型 public delegate int lua_CSFunction(IntPtr L); 必须是静态函数,且带有MonoPInvokeCallbackAttribute属性修饰
新建一个脚本,扩展LuaDLL.Lua类,用来定义访问第三方库的静态函数
```c#
namespace LuaDLL
{ 
    public partial class Lua
    { 
        [DllImport(LUADLL, CallingConvention = CallingConvention.Cdecl)]
        public static extern int luaopen_rapidjson(System.IntPtr L);

        [MonoPInvokeCallback(typeof(LuaDLL.lua_CSFunction))]
        public static int LoadRapidJson(System.IntPtr L)
        {
            return luaopen_rapidjson(L);
        }
    }
}
```
然后调用AddBuildin
```c#
LuaEnv luaenv = new LuaEnv();
luaenv.AddBuildin("rapidjson", LuaDLL.Lua.LoadRapidJson);
```
这里的技巧是使用partial关键字,在你新建的脚本,比如就叫LuaThirdLibExtension.cs中来扩展Lua类,更有利于管理第三方库的增删可debug.
> DllImport的使用
>- Why: .net framework 会将源码先编译为IL/MSIL, 然后在CLR中通过JIT compiler编译为native code执行. 我们游戏中通过c#编写的源码就属于这一类, 也就是所谓的托管代码(managed code) 而我们将要导入的lua库是由c/c++编写的,他们属于非托管代码(unmanaged code),因为在预编译阶段CLR只会拷贝编译托管代码,所以我们需要为非托管代码添加DllImport属性,使其不被裁剪.
>- How: DllImport(dll名字), CallingConvention = CallingConvention.Cdecl; Cdecl: 调用者清空栈(the caller cleans the stack.Enables calling functions with varargs) 这一块不懂.
> 方法声明需要用到static,extern关键字,方法名需要和dll中的方法名一致.约定的命名规则是luaopen_xxx
> 在初始化环节,lua调用c#(LoadRapidJson),再从c#调用c的dll里的方法(luaopen_rapidjson).为了从lua调用c#代码,需要使用MonoPInvokeCallback属性.注：MonoPInvokeCallback标签可以保证在禁止了JIT的环境下也能运行
- 在上述工作全部完成的情况下,调用require('rapidjson')时报了如下的错误
> EntryPointNotFoundException: luaopen_rapidjson
> 有两种情况可能出现这个报错
>- 导入xlua.bundle后没有重启unity
>- 编译失败

今天在这个问题上卡了很久,最后发现是cmakelist.txt配置错误导致没有编译rapidjson

之所以没有编译到,是因为没有认真阅读cmakelist.txt里的代码,将rapidjson的编译配置(如下)写在了cmakelist.txt的末尾,此时已经结束了编译......(汗)
```cmake
 #begin lua-rapidjson
 set (RAPIDJSON_SRC lua-rapidjson/source/rapidjson.cpp)
 set_property(
     SOURCE ${RAPIDJSON_SRC}
     APPEND
     PROPERTY COMPILE_DEFINITIONS
     LUA_LIB
 )
 list(APPEND THIRDPART_INC  lua-rapidjson/include)
 set (THIRDPART_SRC ${THIRDPART_SRC} ${RAPIDJSON_SRC})
 #end lua-rapidjson
```
- 