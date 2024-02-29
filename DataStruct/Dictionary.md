# Dictionary
## 定义


## 1. 创建字典

```c#
using Sysetm.Colletions.Generic;

Dictionary<T1, T2> dict = new Dictionary<>();
```

在上述代码中，我们创建了一个`Dictionary<T1, T2>`类型的字典，key类型为T1，value类型为T2。

## 2. 添加元素

可以使用`Add()`方法向字典中直接添加键值对。如果key已经存在字典中，则`Add()`方法将会抛出异常，如果我们希望可以重复，则可以使用索引符号`[]`直接给字典赋值，这样如果key已经存在，则会更新对应的值。

```c#
dict.add(1, 1);
dict.add(2, 44);

dict[1] = 3;
```

## 3. 访问元素

可以通过key访问字典中的值，使用索引符号`[]`和key来获取对应的value，如果字典中不存在该key，则会抛出异常。所以为了避免这个现象，可以使用`TryGetValue()`方法。

```c#
int value = dict[1];

if (dict.TryGetValue(4, out int check)) {
    Console.WriteLine($"value is :{check}");
}
else 
{
    Console.WriteLine("not found");
}
```

上述代码中，我们通过`dict[1]`直接获取key=1的value，并使用`TryGetValue()`方法获取key=4的value，如果key=4存在，则将对应的value赋值给变量check，否则输出not found。

## 4. 删除元素

可以使用`Remove()`方法根据key从字典中删除元素。

```c#
dict.Remove(1);
```

## 5. 遍历字典

可以使用`foreach`循环遍历字典中的所有键值对，或者分别遍历key和value，

```c#
foreach(var m in dict) {
    Console.WriteLine($"key:{m.Key},value:{m.Value}");
}

// 遍历key
foreach(var key in dict.Keys) {
    Console.WriteLine(key);
}

// 遍历value
foreach(var value in dict.Values) {
    Console.WriteLine(value);
}
```

## 6. 常用方法

### Count

`Count`属性用于获取字典中键值对的数量。
`int count = dict.Count;`

### ContainsKey & ContainsValue

`ContainsKey()`方法用于判断字典中是否包含指定的key，返回一个bool。`ContainsValue()`同理。

### Clear

`Clear()`方法用于清空整个字典。即删除所有的键值对。


