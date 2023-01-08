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


框架提供参数注入，有两种参数类型：

- `[Para]`:以中括号包围的被称为必须参数，表示参数一定要存在
- `<Para>`:以尖括号包围的被称为可选参数，表示参数不存在

在Command声明的参数应当与Action函数参数的顺序一致，且可选参数需要使用`?`做Nullable声明。此外，必须设计为必须参数在前，可选参数在后的结构，否则会引发歧义。