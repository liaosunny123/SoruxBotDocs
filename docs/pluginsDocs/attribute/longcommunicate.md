# LongCommunicate 特性

长对话模块的用处在于讲多个对话之间封装为简答的逻辑暴露给开发者。

在我们实际开发中可能会有一句话无法表达的场景，可能会有引导式收集用户输入的场景，这个时候你可以使用长对话模块。

## 基本概念

每一个MessageContext包含了一个状态，长对话模块会人为捕获某一个状态输入给指定的插件。

## 使用

- `Controller`初始化的时候，在构造函数中填写`ILongMessageCommunicate`类型的参数，作为构造参数，并自己保存为本地的类变量。

## API

根据`SoruxBot`的开发逻辑，API分为泛型API和通用API。

### 泛型API

```csharp
public Task<MessageContext?> CreateGenericListenerAsync(EventType eventType, string? targetPlatform,string? targetAction, Func<MessageContext, bool> action, bool isIntercept, PluginFucFlag flag, int? timeOut);
```

- EventType：表示事件类型，即触发的五大事件中的一种
- targerPlatform：表示触发的平台类型，可为空
- targetAction：表示触发的动作类型，可为空

:::tip
如果对这三个参数不了解，请前往Command特性页查看相关逻辑。
:::

:::warning
最佳实践：建议三个参数都填写，或填写至少前两个参数。但是根据实际开发要求，插件往往只需要对某一个特定的状态（三个参数都确定的情况）下进行唯一监听
:::

- action：一个委托，表示该消息是否为需要监听的对象
- isIntercept：是否拦截，表示的是监听器是否消耗消息。
- flag：表示处理后返回结果。
:::tip
在框架中可能有多个插件，暂且即为A，B，C。此时A开始对用户下一条信息进行监听，B插件会捕获用户的每一个语句。
在框架中的调用顺序为：权限>长对话>普通插件调用
因此，在长对话被解析后，用户可以自行选择是否允许其他监听器/插件对该消息做出反应。
如果为拦截，那么消息立即被框架消耗，不被使用。
如果不拦截，那么消息继续被所有监听器处理（或在下一个监听器选择拦截后被消耗）。此时，监听器可以给消息一个Flag，表示消息的状态。因为每一个消息进入了插件被处理，必须更新相应的Flag。(如果将Flag设置为拦截，那么消息只会被监听器所读取，而不会被任何插件读取。)
:::

:::warning
在`不拦截+flag拦截`的情况下中会出现某个监听器意外的对flag已经设置为`拦截`的消息进行覆写为其他flag类型，这被称为`脏设置`。
这是由于监听器不受`flag`的影响。
因此我们在将flag标记为拦截之前，需要检查一下是否为消息是否已经为拦截状态，从而确定flag是否能随意设置
:::

- timeOut：最大等待时间。超过这个时间后监听器将被自动释放，单位为秒。