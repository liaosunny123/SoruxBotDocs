# Json

Json用于说明插件的信息，一般需要配置的有Name和Privilege选项：

例如：
```json
{
    "Name": "Sorux示例插件",
    "Privilege": 2,
    "PermissionNode": [],
    "PermissionDefaultConfig": []
}
```

- Name:插件的名称
- Privilege:插件的优先级
- PermissionNode:插件的权限配置
- PermissionDefaultConfig:默认权限配置

::: tip
插件优先级不可冲突，当发送冲突时框架会主动处理冲突并提示用户，此外用户也可以更改框架处理冲突时候的行为和插件的优先级。因此不用担心插件优先级冲突。
在插件优先级冲突时，默认采用的顺序时文件读取的顺序，且后一个插件的优先级向后推1，以此类推
:::

## Name

仅仅表示插件的名称，我们推荐的做法是将此处的名称，和日志输出的名称统一，以方便用户识别。

## Privilege

表示插件的优先级，优先级表示同一个触发指令下插件的执行顺序和流程，优先级数值越低，表示优先级顺序越高。高优先级的插件可以控制消息是否传递和消息的具体状态，但不可更改根消息，即除消息状态外的任何这状态。关于消息状态可见[返回值概述](/pluginsDocs/returnType.md).

## PermissionNode

是一个数组，表示插件的权限配置节点，用于配置插件Permission特性注释的Action执行条件。

例如，对于这样的一个权限节点：

```csharp
[Event(EventType.SoloMessage)]  
[Command(CommandAttribute.Prefix.Single,"echoPrivileged [msg]")]  
[Permission("EpicMo.Example.ChatPlugins.GroupSay")]  
public PluginFucFlag EchoPrivilege(MessageContext context,string msg)  
{  
    _bot.SendPrivateMessage(context,"你好，你发送的消息是" + msg);  
    return PluginFucFlag.MsgFlag;  
}
```

对于这样的一个Action，执行的权限是`EpicMo.Example.ChatPlugins.GroupSay`，在插件执行前，框架会对消息上下文进行解析，判断触发消息的对象是否具有相应的权限。

以Json配置文件的一项配置为例：

```json
"PermissionNode":
    [
        {
            "Node": "EpicMo.Example.ChatPlugins.GroupSay",
            "Description": "在群聊中使用Saying",
            "ConditionType": "BasicModel",
            "ConditionChar": "TriggerId"
        },
        {
            "Node": "EpicMo.Example.ChatPlugins.GroupBan",
            "Description": "在群聊中禁言",
            "ConditionType": "BasicModel",
            "ConditionChar": "TriggerId"
        }
    ]
```

对于PermissionNode下的每一个节点，称为`Node`，每一个`Node`应该具有如上的四个字段。



- Node:表示Node的具体表现，且对大小写敏感。
- Description:表示Node的描述，只对框架使用者有效，相当于对Node的注释。
- ConditionType:表示触发条件的消息状态，可为：BasicModel和UnderLayModel两个值。
- ConditionChar:表示触发消息的字段类型

### ConditionType与ConditionChar的使用
其中，BasicModel表示基本消息字段，其可以触发的ConditionChar为TriggerId或者是TriggerPlatformId两个字段，否则会报错。
UnderLayModel为额外消息字段，其可以触发的ConditionChar可以为任意数据。

具体说明：
在SoruxBot的权限系统中，每一个Action可以绑定一个泛型节点，表示触发Action的过滤条件：
其中，在插件中使用权限节点的时候需要配置相应的泛型节点，框架会自动生成平台的特定节点。

例如：对于特性标注的Node<>，插件会自动生成若干Node，分别对应不同的聊天平台。这意味着账号为A的TriggerId一般只能在一个平台触发这个Action。因为我们没有理由认为不同平台的相同个体具有同样的TriggerId.

在具体进行权限过滤的同时，Action除了会根据平台生成平台特定的权限节点，还会生成特定的条件作为节点后缀，而生成后缀的过程需要显式指明插件的生成过程。

对于BasicModel类型的ConditionType，其可选值TriggerId表示判断Action过滤的条件是否为某个具体的人。对于TriggerPlatformId，表示判断Action过滤的条件是否为某个具体的群聊。

对于UnderLayModel类型的ConditionType，ConditionChar为可以在消息上下文中的UnderLayDictionary获取的字段。

因此，插件过滤的具体执行流程为：

插件开发者提供Node，框架自动根据Node生成平台特定的Node并存储，且根据Condition判断Node如何工作，比如是验证群聊还是个体。

## PermissionDefaultConfig

对于插件开发者而言，可能需要在插件未进行任何操作时已经对某些成员授权，这个操作被称为预授权，也就是权限的默认配置。

例如：

```json
"PermissionDefaultConfig":
    [
        {
            "Node": "EpicMo.Example.ChatPlugins.GroupSay",
            "Platform": "qq",
            "Condition": "1919810114514"
        }
    ]
```

对于每一个节点，需要具有上述三个类型：
- Node:表示授权的节点
- Platform:表示泛型节点的显示平台
- 授权类型，即ConditionChar的预设值

## 再看权限节点

我们以本文的权限配置为例，可以帮助我们理解权限系统的设计。我们以`EpicMo.Example.ChatPlugins.GroupSay`为例子：

首先，我们在PermissionNode显示设置了这个节点，并且将触发条件，也就是Condition设置为TriggerId，表示我们验证的类型是触发者。

然后，我们配置了默认权限。由于Node在框架中会根据平台自动生成非泛型版本，因此我们在为一个用户授权的同时需要显示指定Node的非泛型版本，也就是Platform。此外，由于ConditionChar为TriggerId，我们需要指定一个TriggerId表示我们想要授权的对象，根据语义，这里指的是操作的个人账号，那么我们设置qq平台的qq账号即可。