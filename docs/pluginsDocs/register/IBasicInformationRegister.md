# IBasicInformationRegister

## 定义

```csharp
public interface IBasicInformationRegister  
{  
    string GetAuthor();  
    string GetDescription();  
    string GetName();  
    string GetVersion();  
    string GetDLL();  
}
```

## 规范

### GetAuthor

填写作者的相关信息，本信息会在插件启动时自动输出

### GetDescription

填写描述信息，本信息会在插件启动时自动输出

### GetName

填写插件的名称，且本名称应该成为插件日志输出的`Source`参数的值

### GetVersion

填写版本信息

### GetDLL

填写DLL名称

::: warning
注意，请确保大小写完全一致，并包含后缀名
:::