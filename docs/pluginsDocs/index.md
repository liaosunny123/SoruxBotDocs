# SoruxBot 插件概述

SoruxBot 插件开发有两种模式：
- 基于框架使用C#接入开发
- 基于框架扩展使用第三方语言进行开发

SoruxBot插件的目录结构：

```bash
Plugins ##表示Wrapper下插件的存储/运行/配置的目录
  Bin  ##表示插件的存储目录
  Config  ##表示插件的配置目录
  Data  ##表示插件的数据目录
```


## 基于C#接入开发

### 开始

- 下载[SoruxBot.Interface](https://www.github.com/liaosunny123/soruxbot)的源码
- 建立你的工作区，并将SoruxBot.Interface添加到你的解决方案中
- 创建你的项目，并选型为C#类库

### 创建项目结构

插件应当包含一个Register类作为向框架注册信息的类，和多个请求事件的Controller类。

#### Register

在Register类中，通过引入命名空间`using Sorux.Framework.Bot.Core.Interface.PluginsSDK.Register`并使类至少继承一个`IBasicInformationRegister`接口

你的代码可能是这个样子：
```csharp
using Sorux.Framework.Bot.Core.Interface.PluginsSDK.Register;

public class Register : IBasicInformationRegister 
{  
    public string GetAuthor() => "SoruxBot";  
  
    public string GetDescription() => "SoruxBot框架调试插件";  
  
    public string GetName() => "SoruxBotDebugHelper";  
  
    public string GetVersion() => "1.0.0-release";  
  
    public string GetDLL() => "soruxbot.pluginshelper.debughelper";   
}
```

插件通过继承Interface.Register命名空间下的若干接口来向框架注册对应的服务，以下是这些接口的具体含义：
- IBasicInformationRegister:必须继承，表示注册插件的基本信息
- IPermissionRegister:可选继承，表示是否请求权限托管服务，继承后可以使用权限管理特性
- IPluginsUUIDRegister:可选继承，表示插件是否向SoruxBot插件市场注册，并提供UUID
::: warning
不经过继承对应的接口表明插件特性，可能会导致在使用接口提供的特性的时候出现无法预知的错误
:::

#### Controller

在Controller类中：
Controller类表示插件的控制类，可以以MVC中的Controller来进行理解。
::: tip
最佳实践：Controller类虽然没有明确的限制，但是建议在开发时每个功能模块自成一个Controller
:::

##### 构造函数

Controller构造函数中可以使用Interface中提供的参数，以下有：
- ILoggerService:日志模块
- IBasicAPI:API实例
- BotContext:机器人上下文模块【部分安全模式下框架可能不允许插件获取此实例】
- ILongMessageCommunicate:长对话支持模块
- IPluginsDataStorage:插件配置持久化模块
- IPluginsStoragePermanentAble:插件临时配置持久化模块
::: tip
最佳实践：Controller类一般需要获取日志模块进行信息输出和API实例进行消息输出
:::

##### Action

以一段代码为例：

```csharp
[Event(EventType.SoloMessage)]  
[Command(CommandAttribute.Prefix.None,"debug [state]")]  
public PluginFucFlag Debug(MessageContext context, string state)  
{  
    switch (state)  
    {        case "on":  
            bot.SendPrivateMessage(context,"Debug Mode on");  
            return PluginFucFlag.MsgIntercepted;  
        case "off":  
            bot.SendPrivateMessage(context,"Debug Mode off");  
            return PluginFucFlag.MsgIntercepted;  
        default:  
            bot.SendPrivateMessage(context,"Error State,only be on/off but receive:" + state);  
            return PluginFucFlag.MsgIntercepted;  
    }}
```

Action分为特性标记，处理内容和返回三部分。

- 特性标记：见[特性页](attribute)
- 处理内容：实现你的逻辑，可见[API页](api)
- 返回：返回消息的状态，见[返回页](returnType.md)

#### Json

插件需要有一个 Json 文件来向插件提供用户可以根据实际情况自行更改的配置项。

::: warning
这并不是说明插件需要把初始化配置项写在此处，此Json文件旨在对插件进行说明，而最好不要配置插件的实现细节。
:::

Json文件应当被放置在Config下，且与插件名称相同，例如对于A.dll，他需要有A.json的配置文件。

::: warning
由于框架可以跨平台运行，所以请注意文件名的细节。在Linux平台中，操作系统对于文件名的大小写有严格的识别。因此可能出现你插件配置文件在Windows下可以运行，但在Linux下不可以，所以请严格控制大小写。
:::

Json文件中的详细配置请见[Json页](json)

自此便是一个插件的基本构造，此外，你还可以使用[特性页](attribute)完成AOP等高级操作。

## 基于其它语言接入开发

文档完善中...