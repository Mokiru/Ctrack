# LINQ(Language Integrated Query)

LINQ能够将查询功能直接引入到C#、VB.NET等编程语言中。查询操作可以通过编程语言自身来表示，而不是嵌入字符串SQL语句。

LINQ主要包含以下三部分：
- LINQ to Objects。
- LINQ to XML
- LINQ to ADO.NET 主要负责对数据库的查询：
    - LINQ to SQL。
    - LINQ to DataSet。
    - LINQ to Entities。

LINQ所在`System.Linq`命名空间。
## 注意

使用LINQ查询过后`ToArray()`等方法想要直接赋值给原数组例如：
```c#
int[] nums = new int[3]{1,2,3};
nums = nums.Take(1).ToArray();
```
这样是没有作用的必须使用遍历赋值。

## Linq查询方法

### 1. 扩展方法Select()

```c#
public static IEnumerable<TResult> Select<TSource, TResult>(this IEnumerable<TSource> source, Func<TSource, TResult> selector);
```

IEnumerable详情于[IEnumerable](../class/IEnumerable/IEnumerable.md)。

委托(delegate)详情于[delegate](../delegate/delegate.md)。

> Select()是一个泛型方法，Select()方法使用的时候要求传递一个委托实例（方法）

```c#
int[] nums = { 1, 2, 3, 4, 7, 8, 9 };
var list = nums.Select(x => x * x);
foreach (var i in list)
{
	Console.Write($"{i} ");
}
```

最终输出结果为`1 4 9 16 49 64 81`。

### 2. 筛选数据Where()

```c#
public static IEnumerable<TSource> Where<TSource>(this IEnumerable<TSource> source, Func<TSource, bool> predicate);
```

> Where()是一个泛型方法，使用时要求传递一个委托实例，但是该实例是一个判断条件，因此应该返回bool类型。

```c#
int[] nums = { 1, 2, 3, 4, 7, 8, 9 };
var list = nums.Where(x => (x & 1) == 1);
foreach (var i in list)
{
	Console.Write($"{i} ");
}
```

最终输出结果为`1 3 7 9`。

### 3. 排序数据OrderBy()

```c#
public static IOrderedEnumerable<TSource> OrderBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, IComparer<TKey>? comparer);
//参数
// source: A sequence of values to order 注意 第一个参数为 this 所以是隐式传递自动传入对象的值
// keySelector: A function to extract a key from an element.
// comparer: An System.Collection.Generic.IComparer`1 to compare keys
//类型参数
//TSource: The type of the elements of source
//TKey: The type of the key returned by keySelector
```

> OrderBy()是一个扩展方法，OrderBy()默认按照升序排序，若想使用降序排序可以使用OrderByDescending()方法。

`OrderBy()`:
```c#
int[] nums = { 3, 1, 6, 21, 54 };
var list = nums.OrderBy(x => x, null);
foreach (var i in list)
{
	Console.Write($"{i} ");
}
```

最终输出结果为`1 3 6 21 54`

`OrderByDescending()`
```c#
int[] nums = { 3, 1, 6, 21, 54 };
var list = nums.OrderByDescending(x => x, null);
foreach (var i in list)
{
	Console.Write($"{i} ");
}
```

最终输出结果为`54 21 6 3 1`。

当然可以看见OrderBy()方法的参数中还有[IComparer](../IComparer/IComparer.md)，自然肯定可以自定义排序，同样做到降序也是肯定的。如下：

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

最终输出结果与`OrderByDescending()`一致。

### 4. 分组数据GroupBy()

```c#
public static IEnumerable<IGrouping<TKey, TElement>> GroupBy<TSource, TKey, TElement>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector, Func<TSource, TElement> elementSelector, IEqualityComparer<TKey>? comparer);
```

```c#
public static IEnumerable<IGrouping<TKey, TSource>> GroupBy<TSource, TKey>(this IEnumerable<TSource> source, Func<TSource, TKey> keySelector);
```

可以将数据按照自定义的条件分类，可以对元素进行分组使用键选择器（keySelector）也可以使用元素选择器（elementSelector）。

只是用keySelector如下：
```c#
List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
// 使用 GroupBy 方法将数字列表按照是否为偶数进行分组
var groupedNumbers = numbers.GroupBy(n => n % 2 == 0 ? "偶数" : "奇数");

// 遍历每个分组并输出分组结果
foreach (var group in groupedNumbers)
{
	Console.WriteLine("Key: " + group.Key);
	foreach (var number in group)
	{
		Console.WriteLine(number);
	}
	Console.WriteLine("------");
}
```

最终输出结果为:
```
Key: 奇数
1
3
5
7
9
------
Key: 偶数
2
4
6
8
10
------
```

使用keySelector和elementSelector如下：

```c#
List<string> fruits = new List<string> { "apple", "banana", "cherry", "blueberry", "orange" };

// 使用 GroupBy 方法将水果列表按照首字母进行分组，并且元素选择器选择长度大于5的水果名
var groupedFruits = fruits.GroupBy(fruit => fruit[0], fruit => fruit.Length > 5 ? fruit : null);

// 遍历每个分组并输出分组结果
foreach (var group in groupedFruits)
{
	Console.WriteLine("Key: " + group.Key);
	foreach (var fruit in group)
	{
		if (fruit != null)
		{
			Console.WriteLine(fruit);
		}
	}
	Console.WriteLine("------");
}
```

最终输出结果为：

```
Key: a
------
Key: b
banana
blueberry
------
Key: c
cherry
------
Key: o
orange
------
```

### LINQ查询时机与查询形式

当定义一个查询时，查询并未立即执行，而是直到需要枚举结果（遍历）时才真正执行，这种方式称之为“延迟执行”。而当使用“聚合扩展方法”返回单一结果，则会强制查询立即执行。例如`var list = nums.Where(x=>x==0).Count();`。

#### LINQ查询的两种形式

1. Method Syntax查询方法方式，主要利用`System.Linq.Enumerable`类中定义的扩展方法和`Lambda`表达式方式进行查询；
2. Query Syntax查询语句方式，一种更接近于SQL语法的查询方式，但是查询语句最终会被翻译为查询方法。两者效果完全一致。例如`var list = from num in nums where x == 0` 相当于 `var list = nums.Where(x=>x==0)`。

### LINQ查询子句

#### 概述

查询表达式：是一种用查询语法表示的表达式，有一组用于类似SQL的语法编写的句子组成，每个句子可以包含一个或多个C#表达式。

#### LINQ查询表达式包含的子句：

1. from子句：指定查询操作的数据源和范围变量；
2. where子句：筛选元素的逻辑条件，返回值是一个bool类型；
3. select子句：指定查询结果的类型和表现形式；
4. orderby子句：对查询结果进行排序；
5. group子句：对查询结果进行分组；
6. into子句：提供一个临时标识符，该表示可以充当对join/group/select子句结果的引用；
7. join子句：连接多个查询操作的数据源；
8. let子句：引入用于存储查询表达式中的子表达式结果的范围变量。

例子：
```c#
List<int> list = new List<int> { 1,3,3,3,3,2 };
var m1 = from i in list where i % 2 == 1 select i;
foreach(var i in m1) {
	Console.Write($"{i} ");
}
/// 最终输出结果为 1 3 3 3 3
```

### LINQ高级查询

- `Count()`：返回集合项的数目。
- `Max/Min/Average/Sum`：即最大/最小/平均/总和。**注意**：使用Sum求和时，可能数组本身是int，但是最终可能结果应该用long，最好自己遍历求和而不使用LINQ。
- `ThenBy()`：提供复合排序条件。 `OrderBy`对一个条件排序，`ThenBy`可以在此结果上又对另一个条件排序。
- 分区类查询：
	- `Take`：提取指定数量的项。
	```c#
	List<int> list = new List<int> { 1, 3, 8, 0, 10 };
	var m = list.Take(2);
	foreach(var i in m) {
		Console.Write($"{i} ");
	}
	// 输出 1 3
	```
	- `Skip`：跳过指定数量的项并获取剩余的项
	```c#
	List<int> list = new List<int> { 1, 3, 8, 0, 10 };
	var m = list.Skip(2);
	foreach(var i in m) {
		Console.Write($"{i} ");
	}
	// 输出 8 0 10
	```
	- `TakeWhile`：只要满足指定条件，就会返回序列的元素，然后跳过剩余元素。（即遇到第一个不满足条件的就停止，然后返回之前的）。
	- `SkipWhile`：只要满足指定条件，就会跳过序列中的元素，然后返回剩余元素。（即遇到第一个不满足条件的就停止，然后返回剩下的）。
	```c#
	List<int> list = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
	var m1 = list.Where(x => x < 3);
	var m2 = list.SkipWhile(x => x < 3);
	var m3 = list.TakeWhile(x => x < 3);
	Console.WriteLine("Where:");
	foreach (var i in m1)
	{
		Console.Write($"{i} ");
	}
	Console.WriteLine("\nSkipWhile:");
	foreach (var i in m2)
	{
		Console.Write($"{i} ");
	}
	Console.WriteLine("\nTakeWhile:");
	foreach (var i in m3)
	{
		Console.Write($"{i} ");
	}
	//输出结果为
	//Where:
	//1 2
	//SkipWhile:
	//3 4 5 6 7 8 9 10
	//TakeWhile:
	//1 2
	```
- `Distinct`：去掉集合中的重复项。
```c#
List<int> list = new List<int> { 1,3,3,3,3,2 };
var m1 = list.Distinct();
foreach(var i in m1) {
	Console.Write($"{i} ");
}
// 最终输出结果 为 1 3 2。
```
- `Range(int start, int count)`：生成一个**整数**序列。是Enumerable静态方法。
```c#
var list = Enumerable.Range(2, 10);
foreach(var i in list) {
	Console.Write($"{i} ");
}
// 输出结果为 2 3 4 5 6 7 8 9 10 11
```
- `Repeat(T element, int count)`：生成一个重复项（可以是泛型）的序列。是Enumerable静态方法。
```c#
var list = Enumerable.Repeat(2, 10);
foreach(var i in list) {
	Console.Write($"{i} ");
}
// 输出 2 2 2 2 2 2 2 2 2 2 
```



