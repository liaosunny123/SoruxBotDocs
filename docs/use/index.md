# 使用SoruxBot

## 下载SoruxBot

首先，我们需要在项目地址下载[SoruxBot](https://github.com/liaosunny123/SoruxBot)。

SoruxBot分为若干组件，本文仅以内置组件为例，我们需要下载一个Wrapper用于运行SoruxBot的核心程序，此外，我们需要下载任意我们喜欢的Provider作为我们需要给插件提供沟通的聊天平台。

如果Provider不具有协议功能，那么需要下载对应的协议层实体对象。

例如，我们需要组建一个QQ平台的聊天机器人在Windows上面使用，那么我们需要下载：

- [Wrapper-v1.0.1-Windows.zip](https://github.com/liaosunny123/SoruxBot/releases/download/v1.0.1/Wrapper-v1.0.1-Windows.zip)
- [Provider.Cqhttp.v1.0.1-Windows.zip](https://github.com/liaosunny123/SoruxBot/releases/download/v1.0.1/Provider.Cqhttp.v1.0.1-Windows.zip)
- [go-cqhttp_windows_386.exe](https://github.com/Mrs4s/go-cqhttp/releases/download/v1.0.0-rc4/go-cqhttp_windows_386.exe)

:::tip
下载go-cqhttp是因为Provider.Cqhttp需要使用go-cqhttp作为协议层实体，你可以根据Provider的描述判断是否需要只下载一个Provider
:::

## 配置SoruxBot

### 启动Wrapper

将Wrapper放在任意一个文件夹中，并且运行EXE文件，等待Wrapper输出完基本信息后关闭Wrapper，此时你会发现你的文件夹中多了一些内容，自此完成了Wrapper的初始化。

下面应该是你多出的文件夹：

```bash
├─Config
├─Lib
├─Logs
├─Plugins
│  ├─Bin
│  ├─Config
│  └─Data
```

然后，我们再次运行EXE文件，打开Wrapper，自此Wrapper完成了启动。

### 启动协议层

我们将Provider放在另一个文件夹中，并运行EXE文件，注意，你需要对Provider进行配置。
在Provider的运行目录下新建`appsettings.json`的文件，并且配置一下内容：
```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "AllowedHosts": "*",
  "gRPCHost": "http://localhost:7151",
  "GoCqHost": "http://localhost:5700/",
  "urls": "https://localhost:7290"
}
```

其中最后一个表示的是Provider的监听地址，你需要保证这个地址与Wrapper的BotConfiguration.xml中的ProviderItem.HttpPostJsonPath一致。

其中gRPCHost需要被保证与xml文件中的WebLister中给定的地址和端口一致。

然后运行go-cqhttp.exe文件，根据提示输入账号密码并登录。登录后更改config.yml配置，使得appsettings.json中的GoCqHost与go-cqhttp的config.yml中的http.address相同，并和xml中的ProviderItem.NetWorkHttpPostPath相同。

此外，你需要取消config.yml中反向HTTP POST的注释，并保证url与appsettings.json中的urls相同。

最后，你需要将go-cqhttp中的消息模式更改为string，而不是array，至此完成了配置。

### 后续启动

你只需要把插件放入Wrapper的Plugins中Bin目录下，并将对应的Json放在Config文件夹下即可。

后续启动时，你可以按照Wrapper-Provider-gocqhttp的方式，依次启动三个程序即可。

