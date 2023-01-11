# 插件示例开发

## 概述

本文将以开发者的视角开发一个通用平台的一言插件。

## 开发之前

首先，我们需要在项目的Release下载我们需要的Wrapper和Provider版本，在本文中我们仅以QQ平台作为我们的示例开发平台。

然后我们需要新建一个`.NET 6.0 Libraries`项目，并导入`Interface`的DLL引用。

当你可以引用Sorux的命名空间时，说明我们可以开始我们的开发了。

## 注册插件Register

创建一个Register类，并实现IBasicInformation接口。由于此插件有命令触发前缀的需求，因此我们同时也实现ICommandPrefix接口。

填写基本信息：

```csharp
public class Register: IBasicInformationRegister , ICommandPrefix  
{  
    public string GetAuthor() => "EpicMo";  
  
    public string GetDescription() => "用于信息的手机、整理";  
  
    public string GetName() => "InformationCollector";  
  
    public string GetVersion() => "1.0.0-release";  
  
    public string GetDLL() => "EpicMo.Plugins.InformationCollector.dll";  
  
    public string GetCommandPrefix() => "#";  
}
```

其中，GetDLL()的返回值便是我们最后编译出来的DLL名称。

## 完成一个最基本的Controller

因为此插件只需要对接一言接口，因此我们只需要一个基本的Controller类即可。

首先，建立我们的类，并继承BotController接口：

```csharp
public class YiYanController: BotController  
{

}
```

接着便是向框架请求相应的服务，因为我们需要向用户发送信息，因此我们需要请求BasicAPI服务，此外，我们可能会有日志记录的要求，因此我们也请求ILoggerService服务：

```csharp
public class YiYanController: BotController  
{  
    private ILoggerService _loggerService;  
    private IBasicAPI _bot;  
    private RestClient _client;  
    public YiYanController(ILoggerService loggerService,IBasicAPI bot)  
    {        
	    this._loggerService = loggerService;  
        this._bot = bot;  
        _client = new RestClient("https://v1.hitokoto.cn/");  
    }
}
```

在这个地方，我们除了请求两个服务外，还在Controller中执行了初始化任务。

:::tip
相较于以插件启用/禁用时调用插件的OnEnable和OnDisable方法，SoruxBot认为一个插件的最基本单位应该是功能集，而不是插件整体。因此，当你有初始化某个功能集的一些服务的需求时，请在Controller执行初始化服务。
:::

:::tip
特别的，插件的每一个Controller只会初始化一次，且初始化时间在所有消息和Action执行之前，由框架统一初始化
:::

然后，分析我们的需求，我们在插件中需要提供一个命令，让在群聊中的用户能够触发得到一言句子的Action，同时，在一言的API文档中，我们发现我们可以手动指定得到句子的类型。

也就是我们既可以直接向一言请求句子，也可以指定句子类型再请求，那么我们的命令参数就应该是包含一个可选参数的命令。因此我们可以得到一个Action雏形:

```csharp
[Event(EventType.GroupMessage)]  
[Command(CommandAttribute.Prefix.Single,"saying <type>")]  
public PluginFucFlag YiYanGet(MessageContext context,string? type)  
{
}
```

在这个Action里面，我们指定了命令的前缀是Single，也就是插件本身向框架注册的前缀，这个前缀就是你在Register里面指定的`GetCommandPrefix()`，此外，由于type是一个可选参数，因此由`<>`包围，而不是`[]`，并且由于type为可选参数，因此他可空。关于插件的参数说明，可见插件的[命令特性](/pluginsDocs/attribute/command.md)，关于插件的触发条件，可见插件的[事件特性](/pluginsDocs/attribute/event.md)。

:::tip
虽然string本身可以为null，但是我们建议你声明可空类型string?而不是string。
:::

由于我们只想本插件在saying被触发后执行，因此我们返回一个拦截消息请求，防止多个插件被触发。关于插件其他的返回请求请见[插件的返回说明](/pluginsDocs/returnType.md)

```csharp
[Event(EventType.GroupMessage)]  
[Command(CommandAttribute.Prefix.Single,"saying <type>")]  
public PluginFucFlag YiYanGet(MessageContext context,string? type)  
{
	return PluginFucFlag.MsgIntercepted;
}
```

:::tip
这一种做法也可能被其他使用saying指令的Action操作。而遇到这个情况是本插件执行还是另一个插件执行，由插件配置中的优先级决定。关于优先级的说明可以见插件的[Json配置说明](/pluginsDocs/json.md)
:::

然后，我们需要具体编写Action的内容，这不是本文重点，因此我们直接给出代码：
```csharp
[Event(EventType.GroupMessage)]  
[Command(CommandAttribute.Prefix.Single,"saying <type>")]  
public PluginFucFlag YiYanGet(MessageContext context,string? type)  
{  
    var request = new RestRequest();  
    request.Method = Method.Get;  
    if (!string.IsNullOrEmpty(type))  
    {        
	    request.AddQueryParameter("c", type);  
    }    
    var result = _client.Execute(request);  
    YiYan model = JsonConvert.DeserializeObject<YiYan>(result.Content!)!;  
    if (!string.IsNullOrEmpty(model.from_who))  
    {        
	    _bot.SendGroupMessage(context,model.hitokoto + "   ---" + model.from_who);  
    }    
    else  
    {  
        _bot.SendGroupMessage(context,model.hitokoto);  
    }    
    return PluginFucFlag.MsgIntercepted;  
}
```
在Action中，我们可以使用插件的BasicAPI来帮助我们快速的发送消息。在SoruxBot的体系中，API分为通用API和特定API，由于我们编写的插件是多平台的，因此使用BasicAPI的基本服务即可，可见我们的`SendGroupMessage`中并未包含任何与QQ聊天平台有关的信息，这也说明了我们的插件可以服务多个聊天平台。

在Action中，我们定义了一个YiYan，这个作为Json反序列化的模板。

## 制作Json
```json
{
    "Name": "YiYan",
    "Privilege": 1,
    "PermissionNode":[],
    "PermissionDefaultConfig":[]
}
```

我们制作了一个最基本的Json，并让我们的默认优先级为1。关于Json的详细说明可以见[插件Json说明](/pluginsDocs/json.md)
:::tip
优先级是可以被用户更改的，因此我们无法保证运行时也是1。关于优先级冲突的叙述请见[插件Json说明](/pluginsDocs/json.md)
:::

## 运行插件

将插件放置在`Wrapper/Plugins/Bin`中，将Json放置在`Wrapper/Plugins/Config`中，然后重启Wrapper，并打开Provider.Cqhttp和Go-cqhttp，我们便完成了插件的运行。