# 存储特性

存储分为文本存储和二进制存储。文本存储在框架提供的SQLite数据库中，二进制存储被保存在Plugins的Data目录中，且文件名为SHA1算法取Hash后的值。

## 使用

- `Controller`初始化的时候，在构造函数中填写`IPluginsDataStorage`类型的参数，作为构造参数，并自己保存为本地的类变量。

## 参数介绍

所有的API均为三个或两个参数。

- pluginMark：插件名称
- key：键名
- value：内容

其中，在二进制存储时，value为`byte[]`类型

