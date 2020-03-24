# 获取窗口

最近在做技能编辑器,其中用到了Timeline来生成时间轴,但是当需要强制刷新Timeline窗口时候出现了无法找到窗口实例的error.
会出错的代码如下:
```c#
void RefreshTimelineWindow(){           
    var windowType = typeof(UnityEditor.Editor).Assembly.GetType("UnityEditor.Timeline.TimelineWindow");
    var window = EditorWindow.GetWindow(windowType);
    if(window)
        window.Repaint();
    }
```
报错信息如下:即找不到该Type导致GetWindow获取实例失败.
> FindAllObjectsOfType: Invalid Type
UnityEditor.EditorWindow:GetWindow(Type)
>NullReferenceException: Object reference not set to an instance of an object

首先我怀疑是Timeline窗口的类名写错了,所以使用反射打印出了所有所有继承自EditorWindow的类,代码如下
```c#
var result = new System.Collections.Generic.List<System.Type>();
System.Reflection.Assembly[] AS = System.AppDomain.CurrentDomain.GetAssemblies();
System.Type editorWindow = typeof(EditorWindow);
foreach (var A in AS)
{
    System.Type[] types = A.GetTypes();
    foreach (var T in types)
    {
        if (T.IsSubclassOf(editorWindow))
            result.Add(T);
    }
}
result.ToArray();
```
打印出了167个继承自EditorWindow的类,包含Timeline名字的类只有一个,正是"UnityEditor.Timeline.TimelineWindow".
不知道这是unity自己的bug还是我使用有误会,于是更换了思路.直接获取当前开启的所有窗口,在里面找出Timeline窗口然后调用Repaint()方法即可.代码如下:
```c#
EditorWindow[] allWindows = Resources.FindObjectsOfTypeAll<EditorWindow>();
foreach (var window in allWindows)
{
    if(window.GetType().Name == "TimelineWindow"){
        window.Repaint();
        break;
    }
}
```