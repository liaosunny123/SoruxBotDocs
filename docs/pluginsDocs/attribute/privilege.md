# Privilege特性

## 基本介绍

关于 Privilege 特性的基本介绍可参见 [Json页](../json) 。

## 特性使用

- 插件的`Register`继承`ICommandPrefix`接口
- 插件使用`PermissionAttribute`作为特性标记
- 在Json中注册你的特性

## API使用

按照`SoruxBot`的设计逻辑，API分为泛型API和基础API。

### 泛型API

**增删某物体权限**

```csharp
public bool AddGenericPermission(MessageContext context, PermissionType permissionType, string permissionChar, string node);
```

```csharp
public bool RemoveGenericPermission(MessageContext context, PermissionType permissionType, string permissionChar, string node);
```

以上分别是增删某个权限的泛型API，可作为通用API使用。

- MessageContext ： 消息的上下文，需要传入以供分析节点
- PermissionType ： 需要被解析的对象
- PermissionChar ： 表示关键词的字面值
- node ： 权限节点

::: tip
关于 PermissionChar 和 node 的使用可以见下文对一个特殊 API 的分析
:::

**查询某物体权限**

```csharp
public string GetGenericPermission(MessageContext context, string obj);
```


**得到某物体权限**

```csharp
public bool GenericHasPermission(MessageContext context, PermissionType permissionType, string permissionChar, string node);
```


### 特殊API

例如，我们以增加某物体权限为例：

```csharp
public bool AddTriggerIdPermission(MessageContext context, string node)  
{  
    return AddGenericPermission(context, PermissionType.BasicPermission, "TriggerId", node);  
}
```

以下为设计的逻辑：

1. 需要设计一个判断个人是否有触发条件的权限节点，并在MessageContext中找到了这个属性，其为`TriggerId`
2. 由于`TriggerId`与平台无关，所以其直接暴露在`MessageContext`中，而非`UnderLayModel`，因此选择`BasicPermission`
3. `TriggerId`的字面值为`TriggerId`，如果是`UnderLayModel`中的字段，那么需要填写相应的字段
4. `node`为任意字符，但是我们推荐采用插件域名反写+个性化字段的命名逻辑，如`cn.epicmo.plugins.command.say`其中`say`为个性化字段，多个个性化字段之间使用`.`相连接
