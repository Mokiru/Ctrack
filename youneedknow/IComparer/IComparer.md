# IComparer<T>接口

## 定义

命名空间:System.Colletion.Generic

程序集:System.Runtime.dll

定义类型为比较两个对象而实现的方法

```c#
public interface IComparer<in T>
```

## 类型参数

`T`要比较的对象的类型。这是逆变类型参数。即，可以使用指定的类型，也可以使用派生程度较低的任何类型。

## 示例

以下示例实现 接口 ，`IComparer<T>`以根据对象的尺寸比较类型`Box`对象，此示例是为类提供的更大示例的——`Comparer<T>`部分。

```c#
public class BoxComp : IComparer<Box>
{
    // Compares by Height, Length, and Width.
    public int Comparer(Box x, Box y) 
    {
        if (x.Height.CompareTo(y.Height) != 0) {
            return x.Height.CompareTo(y.Height);
        } 
        else if (x.Length.CompareTo(y.Length) != 0) {
            return x.Length.CompareTo(y.Length);
        } 
        else if (x.Width.CompareTo(y.Width) != 0) {
            return x.Width.CompareTo(y.Width);
        }
        else
        {
            return 0;
        }
    }
}
```

建议派生自`Comparer<T>`类，而不是实现`IComparer<T>`接口，因为`Comparer<T>`类提供方法的`IComparer.Compare`显示接口实现，以及`Default`获取对象的默认比较器的属性。

在自定义排序中使用如下：

```c#
int[] nums = { 3, 1, 6, 21, 54 };

var list = nums.OrderBy(x => x, Comparer<int>.Create((x, y)=>{
	return y.CompareTo(x);
}));

foreach(var i in list) 
{
	Console.Write($"{i} ");
}
```

以上使用OrderBy，默认是升序，最后使用自定义排序，实现降序，最终输出结果为`54 21 6 3 1`。


