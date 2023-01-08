# Hello, SoruxBot!
你好，欢迎来到 SoruxBot 框架的文档网站，在这里你会对 SoruxBot 的框架开发和插件开发有一个基本的了解！

SoruxBot 是一个支持多语言接入的、可跨平台运行的且可以实现一次开发多处运行的多聊天平台机器人框架，你可以在任意的聊天平台，甚至是Shell接入SoruxBot.

## 快速了解【点击跳转】
- 体系构成概述：见下方
- 接入SoruxBot插件：
- 接入SoruxBot 框架：
- 接入SoruxBot 协议层适配器：
- 接入SoruxBot 协议层实体：

## 体系构成

在 SoruxBot 的体系内，分为四个部分：
- 框架本体：负责SoruxBot的运行和插件的调度等等环节
- 协议层适配器：负责与SoruxBot进行通信，在通用适配模块和特定聊天平台间实现转换
- 协议层实体：负责与聊天平台进行通信，并且和协议层适配器进行沟通
- 插件：负责对某一特定事件进行处理或进行泛处理

### 框架本体
在 SoruxBot 框架内，分为三个部分：
- Core.Kernel：为框架的主体，其性质属于类库，可以被包装器所构建
- Core.Wrapper：为框架的内置包装器，负责组装Kernel以建立机器人实例
- Core.Interface：为Kernel所支持的接口
#### Core.Kernel
SoruxBot 采用管道通信的模型，与外界的通信采用“入栈”，“出栈”和“特连”的概念。
- 入栈：接收外界的信息，并且放入Kernel的消息管道
- 出栈：推送内部信息，并且放入Kernel的通信管道，但是这种方式会主动舍弃对于信息的状态跟踪
- 特接：推送内部的信息，但是直接与协议层实体进行沟通，并且通过内置的异步或同步的两种API获取返回值
Kernel 中有多个内部库，并提供给 Wrapper 进行选择性组装，这意味着你可以实现自己的Wrapper 来动态构建一个不需要管道等微型机器人。
此外，Kernel支持动态组件的替换，这意味着你可以根据Kernel内置的接口开发你自己的Kernel组件。Kernel为了避免对外部程序的依赖，因此内置组件均是基于C#内部库或者是部分NuGet开发，例如你可以根据自己的需求将管道换成Redis版本等。
#### Core.Wrapper
Kernel 的包装库，在SoruxBot内提供了一个内置的Wrapper，其采用了Kernel的全部库并构建出了一个大型的框架模型。
在Wrapper中，负责对Kernel和外界的沟通，同时，你可以在此处无侵入性的增加你的中间件服务，以实现对消息的过滤，对数据的统计，对路由的追踪等等。
#### Core.Interface
Kernel的开发协议库，负责插件作者使用。
Interface是为插件作者服务的，其中提供了基本的数据模型，交互模型和API，同时提供组装自定义API请求的帮助类库，实现插件脱离Kernel开发。
注：若要开发Kernel类库，请使用Kernel中的Interface接口，而不是此处的Interface
### 协议层适配器
协议层适配器负责：
- 接受协议层实体的上报，并组装成通用模型给Wrapper
- 接受框架的请求推送，并组装成特定模型给协议层实体
在部分语境下，若协议层实体支持SoruxBot协议或OneBot协议，那么可以忽略此层。
### 插件
插件负责对于消息进行处理，并向框架发送请求信息。
插件由一下构成：
- Controller：插件内自定义的控制器类
- Register：插件注册类
其中Controller只需要继承Interface的BotController即可，实际上，在框架内部看来插件的基本构成是Controller的Action而不是Controller，因此你最大程度的自定义你的Controller