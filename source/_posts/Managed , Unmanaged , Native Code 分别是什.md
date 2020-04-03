# Managed , Unmanaged , Native Code 分别是什么
参考文章:
[1.Managed, Unmanaged, Native: What Kind of Code Is This?]("https://www.developer.com/net/cplus/article.php/2197621/Managed-Unmanaged-Native-What-Kind-of-Code-Is-This.htm")

## Managed Code(托管代码)
- .NET and C# 编译器产生的代码, 源码会被编译为中间语言(intermediate language(IL)), 而不是计算机可直接执行的机器码(machine code)
- 中间语言(IL)以及一份metadata被保存在一个被称为assembly的文件中,metadata中记录了classes,methods,attributes
- assembly可以被拷贝到其他平台上使用,不需要在那台电脑上重新编译源码
- 共用语言运行时(Common Language Runtime(CLR))负责运行Managed Code,并提供了诸多服务
- 代码的运行通常是这样的:
> 1. CLR加载assembly并验证IL是否正确
> 2. 当某个methods被调用时,CLR将会将相关的IL代码编译为当前运行平台的机器码(machine code)
> 3. 机器码会被缓存(cached)以便下一次调用
> 4. 这种方式通常被称作just in time / JIT compiling / Jitting
> 5. 伴随该assembly运行的全生命周期,CLR全程提供安全检测,内存管理,线程管理, 这就好像这个应用被CLR“托管”了一样
