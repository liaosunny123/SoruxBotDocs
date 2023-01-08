# 特性

特性是C#中的一种语法，可以相当于Java中的注解，可以标注方法的具体属性。

在SoruxBot框架中，你可以在Attribute文件夹中找到特性的API，并引用到你的项目中。

## 特性使用规范

对于Controller中的特性才会被框架注册，除此之外，框架也同样拒绝不在Register类中注册接口而使用需要被这样注册的特性。

例如，PermissionAttribute需要开发者在Register中继承IPermissionRegister接口，如果没有继承接口就使用可能会导致无法预知的后果。

## 基本特性

对于需要接受聊天平台的事件通知的Action，需要注册EventAttribute和CommandAttribute(可选)两个特性。

例如，以一段代码为例：

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

在这个地方，Action注册了SoloMessage特性，表示其接受私聊消息，此外他注册了Command特性，表示他被触发的指令是`debug [state]`

那么当用于以私聊方式发送消息`debug [state]`到机器人时，机器人会自动识别指令并且调用Action

对于此类管理插件是否被触发的基本特性，成为基本特性，且以下是详细的介绍：

- [Event](/pluginsDocs/attribute/event.md)
- [Command](/pluginsDocs/attribute/command.md)

