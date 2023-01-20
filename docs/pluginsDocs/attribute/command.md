# Command特性

Command提供了两个功能：

- 自定义回复前缀
- 自定义消息触发体

命令原型
`[Command(CommandAttribute.Prefix,para string[])]`

## 自定义回复前缀

自定义消息的回复前缀，有三个可选项：

- None:表示插件不需要使用前缀
- Single:表示插件需要使用前缀，且前缀由插件自己提供
- Global:表示插件需要使用前缀，且前缀由框架提供（也就是用户提供）

## 自定义命令

命令表示插件触发命令后Action所做的事情。

注意，若同一个命令头被注册，那么会按照框架用户定义的优先级执行。若优先级较低且优先级较高的插件拦截了消息请求，那么插件Action可能出现不会被触发的现象。

命令分为两个部分，一个是母命令头，一个是参数部分，例如：

`Say [type]`

那么当用户输入Say时会传入Action，且空格后的被解析成为参数。

:::warning
由于语义的歧义性，Sorux仅支持`string`和`int`类型的解析，且不支持其他类型的解析。
:::

:::tip
Sorux的参数类型解析是根据Action的参数类型进行解析转换的。当声明了Action的参数类型时，参数会被对应解析成相应的类型。
:::

框架提供参数注入，有两种参数类型：

- `[Para]`:以中括号包围的被称为必须参数，表示参数一定要存在
- `<Para>`:以尖括号包围的被称为可选参数，表示参数不存在

在Command声明的参数应当与Action函数参数的顺序一致，且可选参数需要使用`?`做Nullable声明。此外，必须设计为必须参数在前，可选参数在后的结构，否则会引发歧义。

## 特殊情况

正常情况下，插件Action的参数注入应当是可选参数或必须参数。但是，在一一对应注入之外，框架也提供了一些特殊注入，适配于参数并非一一注入的情况。

:::tip
在正常注入下，每个参数会一一匹配，若参数数目不匹配则会检查参数是否可选，若均不符合那么会视为消息发生了“匹配丢失”，这说明不会有任何一个Action被触发。
:::

### 全监听

全监听指的是Action不需要向框架请求参数注入，而是以“事件”方式进行监听。简而言之，发送的任何消息都会进入Action。

我们需要这样去声明这样的一个Action:

```csharp
[Event(EventType.SoloMessage)]  
[Command(CommandAttribute.Prefix.None,"[SF-ALL]")]  
public PluginFucFlag Test(MessageContext context, string msg)  
{  
	//此处进行一些逻辑处理
}
```

:::tip
`[SF-ALL]`与`CQ码`的概念类似，指的是以简短的字符串表示特殊含义的语句。
请不要写为`[SF:ALL]`这可能会造成将名称为SF的参数解析为ALL类型的参数注入歧义的现象。
其中，SF表示的是SoruxBotFramework，简称SF。
:::

:::warning
请保持Action的命令注册在逻辑上一致。若消息被注册为全监听，那么不应该有"命令头"，也不应该有触发的"Prefix"，若存在二者中任何一个，或二者均存在，那么会造成命令解析的歧义。
:::

### 正则监听

正则监听指的是Action需要更为复杂的命令路由解析，以简化在Action内部判断的流程。
正则监听需要使用`SF-Regex`标注，如下述例子：

```csharp
[Event(EventType.SoloMessage)]  
[Command(CommandAttribute.Prefix.None,"test [SF-Regex-正则表达式]")]  
public PluginFucFlag Test(MessageContext context, string msg)  
{  
	//此处进行一些逻辑处理
}
```

其中，正则表达式这五个字应该被替换成对于的正则表达式。