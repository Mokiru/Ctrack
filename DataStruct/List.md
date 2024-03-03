# List<T>类

## 定义

命名空间：`System.Collections.Generic`。
程序集：`System.Collections.dll`。

表示可通过索引访问的对象的强类型列表。提供用于对列表进行搜索、排序和操作的方法。

```c#
public class List<T> : System.Collections.Generic.ICollection<T>, System.Collections.Generic.IEnumerable<T>, System.Collections.Generic.IList<T>, System.Collections.Generic.IReadOnlyCollection<T>, System.Collections.Generic.IReadOnlyList<T>, System.Collections.IList
```

## 类型参数

`T`：列表中元素的类型。

继承`Object`->List<T>

## 构造函数

- `List<T>()`: 初始化`List<T>`，为空并且具有默认的初始容量0。
- `List<T>(IEnumerable<T>)`：初始化，且包含从指定集合赋值的元素并且具有足够的容量容纳所复制的元素。如：
```c#
List<int> list = new List<int>(Enumerable.Repeat(3,4));
foreach (var i in list)
{
    Console.Write($"{i} ") // 3 3 3 3
}
```
- `List<T>(Int32)`：初始化，该实例为空并且具有指定的`Capacity`。

## 属性

- `Capacity`：获取或设置该内部数据结构在不调整大小的情况下能够容纳的元素总数。
- `Count`：获取List中包含的元素数。
- `Item(Int32)`：获取或设置指定索引处的元素。

## 方法

- `Add(T)`：将对象添加到List结尾处。
- `Sort()`：使用默认比较器对整个List排序。
- `Sort(IComparer<T>)`：使用指定比较器排序。
- `Sort(Int32 index, Int32 count, IComparer<T>)`：使用指定比较器，对指定范围排序。



## 问题总结

1. C#的List在使用过程中有什么需要注意的点？防止效率低下需要注意的点呢？
    1. 避免频繁的插入和删除操作。（如果在`Capacity`范围内插入也不影响，主要是删除，可能会让元素移动，影响效率）。
    2. 如果预先知道大小尽量初始化时就指定`Capacity`。
    3. 避免在循环中使用`List.Count`，尽量提前用变量保存，而不是每次都使用`Count`。
    4. 避免频繁的集合操作。如LINQ。
    5. 注意对获取元素时可能为空的情况检查。
    6. 避免频繁的排序。

2. 与ArrayList区别：
    1. 类型安全性
        - List是泛型集合，其中T是指定存储的元素类型，可以确保集合中只能存储指定类型的元素，提高代码的类型安全性。
        - ArrayList是非泛型集合，可以存储任意类型的元素，添加元素时不会进行类型检查，可能导致类型转换错误。
    2. 自动装箱和拆箱：
        - List泛型集合不涉及装箱和拆箱操作，提高了性能。
        - ArrayList非泛型集合在存储值类型时会发生装箱操作，获取值类型时会发生拆箱操作。
    3. 强类型访问：
        - List可以通过下标直接访问元素，并且不需要进行类型转换。
        - ArrayList需要进行类型转换后才能访问元素，可能引发类型转换异常。
    4. 扩展性：
        - List提供了更多的API和LINQ查询（实现IEnumerable<T>接口）语法，更易于对集合进行操作和处理。
        - ArrayList较为简单，处理元素繁琐。