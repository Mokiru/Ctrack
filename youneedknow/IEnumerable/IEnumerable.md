# IEnumerable<T>接口

## 定义

命名空间：`System.Collections.Generic`。
程序集：`System.Runtime.dll`。

公开枚举数， 该枚举数支持在指定类型的集合上进行简单迭代。
```c#
public interface IEnumerable<out T> : System.Collections.IEnumerable
```

## 类型参数

`T`:要枚举的对象的类型，这是协变类型参数，即，可以使用指定的类型，也可以使用派生程度较高的任何类型。有关协变和逆变的详细信息，于[泛型中的协变和逆变](../youneedknow/泛型中的协变和逆变/泛型中的协变和逆变.md)。


