# IEnumerator<T>接口

## 定义

命名空间：`System.Collections.Generic`。
程序集：`System.Runtime.dll`。

支持在泛型集合上进行简单迭代。
```c#
public interface IEnumerator<out T> : IDisposable, System.Collections.IEnumerator
```

## 类型参数

`T`：要枚举的对象的类型。

## 示例

下面的示例演示自定义对象的集合类 接口实现`IEnumerator<T>`。自定义对象是类型的`Box`实例，集合类为`BoxCollection`。
```c#
// Defines the enumerator for the Boxes collection.
// (Some prefer this class nested in the collection class.)
public class BoxEnumerator : IEnumerator<Box>
{
    private BoxCollection _collection;
    private int curIndex;
    private Box curBox;

    public BoxEnumerator(BoxCollection collection)
    {
        _collection = collection;
        curIndex = -1;
        curBox = default(Box);
    }

    public bool MoveNext()
    {
        //Avoids going beyond the end of the collection.
        if (++curIndex >= _collection.Count)
        {
            return false;
        }
        else
        {
            // Set current box to next item in collection.
            curBox = _collection[curIndex];
        }
        return true;
    }

    public void Reset() { curIndex = -1; }

    void IDisposable.Dispose() { }

    public Box Current
    {
        get { return curBox; }
    }

    object IEnumerator.Current
    {
        get { return Current; }
    }
}
```

## 注解

`IEnumerator<T>`是所有泛型枚举器的基本接口。

枚举器可以用于读取集合中的数据，但不能用于修改基础集合。

最初，**枚举数定位在集合中第一个元素的前面**。在此位置上，未定义`Current`，为对应类型的默认值。因此，在读取`MoveNext`的值之前，必须调用`Current`将枚举器向前移动到集合的第一个元素。例如
```c#
List<int> list = new List<int>(5) { 0, 1, 2, 3, 4 };
IEnumerator<int> de = list.GetEnumerator();
Console.WriteLine(de.Current);
de.MoveNext();
Console.WriteLine(de.Current);
// out : 
//0 
//0
```

在调用`Current`之前，`MoveNext`返回相同的对象。`MoveNext`将`Current`设置为下一个元素。

如果`MoveNext`传递集合的末尾，则枚举器位于集合中最后一个元素之后，并`MoveNext`返回`false`。当枚举器位于此位置时，对`MoveNext`的后续调用也会是返回`false`。如果最后一次`MoveNext`调用返回`false`，`Current`则为未定义。无法再次将`Current`设置为集合的第一个元素；必须改为创建新的枚举器实例。

提供`Reset`方法是为了实现COM互操作性。它不一定需要实施：相反，实现者只需引发。`NotSupportedException`但是，如果选择执行此操作，则应确保没有调用方法依赖该功能`Reset`。

如果对集合进行了更改，例如添加、修改或删除元素，则枚举器的行为是不确定的。

枚举数没有对集合的独占访问权；因此，从头到尾对一个集合进行枚举在本质上不是一个线程安全的过程。若要确保枚举过程中的线程安全性，可以在整个枚举过程中锁定集合，若要允许多个线程访问集合以进行读写操作，则必须实现自己的同步。

`System.Collections.Generic`命名空间中集合的默认实现是不同步的。

## 属性

`Current`：获取集合中位于枚举数当前位置的元素。

## 方法

`Dispose()`：执行与释放或重置非托管资源关联的应用程序定义的任务。

`MoveNext()`：将枚举数推进到集合的下一个元素。

`Reset()`：将枚举数设置为其初始位置，该位置位于集合中第一个元素之前。









