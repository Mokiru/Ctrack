# PriorityQueue<TElement,TPriority> 类

## 定义

命名空间：`System.Collections.Generic`。
程序集：`System.Collections.dll`。

表示具有值和优先级的项的集合。出队时，将删除优先级最低的项。
```c#
public class PriorityQueue<TElement, TPriority>
```

## 参数类型

`TElement`：元素在队列中的类型。

`TPriority`：指定与排队元素关联的优先级类型。

## 注解

实现数组支持的四元最小堆。每个元素进队都使用关联优先级排队来确定出队顺序。优先级最低的元素首先出队。请注意，类型不保证优先级相等的元素FIFO。

## 构造函数

- `PriorityQueue<TElement,TPriority>()`：初始化。
- `PriorityQueue<TElement,TPriority>(IComparer<TPriority>)`：使用指定的自定义优先级比较器初始化。
- `PriorityQueue<TElement,TPriority>(IEnumerable<ValueTuple<TElement,TPriority>>)`：初始化用指定元素和优先级填充的类。
- `PriorityQueue<TElement,TPriority>(IEnumerable<ValueTuple<TElement,TPriority>>,IComparer<TPriority>)`：初始化类的新实例，使用指定的元素和优先级填充，并使用指定的自定义优先级比较进行填充。
- `PriorityQueue<TElement,TPriority>(Int32)`：使用指定的初始容量初始化。
- `PriorityQueue<TElement,TPriority>(Int32,IComparer<TPriority>)`：使用指定初始容量，并自定义优先级。

## 属性

- `Comparer`：获取所使用的优先级比较器。
- `Count`：获取优先队列中包含的元素个数。
- `UnorderedItems`：获取一个集合，该集合以无序方式枚举队列的元素。

## 方法

- `Clear()`：从优先队列中移除所有项。
- `Dequeue()`：从优先队列中移除并返回优先级最小的元素。
- `DequeueEnqueue(TElement,TPriority)`：删除最小元素，然后立即将具有关联优先级的指定元素添加到优先队列中。
```c#
var pq = new PriorityQueue<int, int>(Comparer<int>.Create((x,y)=>y.CompareTo(x)));
pq.Enqueue(1, 2);
pq.Enqueue(2, 1);
var a = pq.DequeueEnqueue(4, 3);//返回当前优先级最低元素，并出队，然后将(4,3)入队
Console.WriteLine(a);
while (pq.TryDequeue(out var e, out var p))
{
	Console.WriteLine(e);
}
```
- `Enqueue(TElement,TPriority)`：将具有关联优先级的指定元素添加到优先队列中。
- `EnqueueDequeue(TElement,TPriority)`：先将元素添加到优先队列中，然后再出队。
- `EnqueueRange(IEnumerable<TElement>,TPriority)`：将元素加入队列，并设置优先级为TPriority。
- `EnqueueRange(IEnumerable<ValueType<TElement,TPriority>>)`：就是批量入队。
- `Peek()`：返回优先级最低的元素，不删除。
- `TryDequeue(out TElement element, out TPriority priority)`：出队，并将值赋给element和priority。
- `TryPeek(out TElement element, out TPriority priority)`：同上。