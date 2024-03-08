# LinkedList<T>类

## 定义

命名空间：`System.Collections.Generic`
程序集：`System.Collections.dll`

表示双重链接列表。

```C#
public class LinkedList<T> : System.Collections.Generic.ICollection<T>, System.Collections.Generic.IEnumerable<T>, System.Collections.Generic.IReadOnlyCollection<T>, System.Collections.ICollection, System.Runtime.Serialization.IDeserializationCallback, System.Runtime.Serialization.ISerializable
```

## 类型参数
`T`：指定链接列表的元素类型。

## 示例

下面的代码示例演示类的许多`LinkedList<T>`功能。
```c#
using System;
using System.Text;
using System.Collections.Generic;

public class Example
{
    public static void Main()
    {
        // Create the link list.
        string[] words =
            { "the", "fox", "jumps", "over", "the", "dog" };
        LinkedList<string> sentence = new LinkedList<string>(words);
        Display(sentence, "The linked list values:");

        // Add the word 'today' to the beginning of the linked list.
        sentence.AddFirst("today");
        Display(sentence, "Test 1: Add 'today' to beginning of the list:");

        // Move the first node to be the last node.
        LinkedListNode<string> mark1 = sentence.First;
        sentence.RemoveFirst();
        sentence.AddLast(mark1);
        Display(sentence, "Test 2: Move first node to be last node:");

        // Change the last node to 'yesterday'.
        sentence.RemoveLast();
        sentence.AddLast("yesterday");
        Display(sentence, "Test 3: Change the last node to 'yesterday':");

        // Move the last node to be the first node.
        mark1 = sentence.Last;
        sentence.RemoveLast();
        sentence.AddFirst(mark1);
        Display(sentence, "Test 4: Move last node to be first node:");

        // Indicate the last occurence of 'the'.
        sentence.RemoveFirst();
        LinkedListNode<string> current = sentence.FindLast("the");
        IndicateNode(current, "Test 5: Indicate last occurence of 'the':");

        // Add 'lazy' and 'old' after 'the' (the LinkedListNode named current).
        sentence.AddAfter(current, "old");
        sentence.AddAfter(current, "lazy");
        IndicateNode(current, "Test 6: Add 'lazy' and 'old' after 'the':");

        // Indicate 'fox' node.
        current = sentence.Find("fox");
        IndicateNode(current, "Test 7: Indicate the 'fox' node:");

        // Add 'quick' and 'brown' before 'fox':
        sentence.AddBefore(current, "quick");
        sentence.AddBefore(current, "brown");
        IndicateNode(current, "Test 8: Add 'quick' and 'brown' before 'fox':");

        // Keep a reference to the current node, 'fox',
        // and to the previous node in the list. Indicate the 'dog' node.
        mark1 = current;
        LinkedListNode<string> mark2 = current.Previous;
        current = sentence.Find("dog");
        IndicateNode(current, "Test 9: Indicate the 'dog' node:");

        // The AddBefore method throws an InvalidOperationException
        // if you try to add a node that already belongs to a list.
        Console.WriteLine("Test 10: Throw exception by adding node (fox) already in the list:");
        try
        {
            sentence.AddBefore(current, mark1);
        }
        catch (InvalidOperationException ex)
        {
            Console.WriteLine("Exception message: {0}", ex.Message);
        }
        Console.WriteLine();

        // Remove the node referred to by mark1, and then add it
        // before the node referred to by current.
        // Indicate the node referred to by current.
        sentence.Remove(mark1);
        sentence.AddBefore(current, mark1);
        IndicateNode(current, "Test 11: Move a referenced node (fox) before the current node (dog):");

        // Remove the node referred to by current.
        sentence.Remove(current);
        IndicateNode(current, "Test 12: Remove current node (dog) and attempt to indicate it:");

        // Add the node after the node referred to by mark2.
        sentence.AddAfter(mark2, current);
        IndicateNode(current, "Test 13: Add node removed in test 11 after a referenced node (brown):");

        // The Remove method finds and removes the
        // first node that that has the specified value.
        sentence.Remove("old");
        Display(sentence, "Test 14: Remove node that has the value 'old':");

        // When the linked list is cast to ICollection(Of String),
        // the Add method adds a node to the end of the list.
        sentence.RemoveLast();
        ICollection<string> icoll = sentence;
        icoll.Add("rhinoceros");
        Display(sentence, "Test 15: Remove last node, cast to ICollection, and add 'rhinoceros':");

        Console.WriteLine("Test 16: Copy the list to an array:");
        // Create an array with the same number of
        // elements as the linked list.
        string[] sArray = new string[sentence.Count];
        sentence.CopyTo(sArray, 0);

        foreach (string s in sArray)
        {
            Console.WriteLine(s);
        }

        Console.WriteLine("Test 17: linked list Contains 'jumps' = {0}",
            sentence.Contains("jumps"));
        
        // Release all the nodes.
        sentence.Clear();

        Console.WriteLine();
        Console.WriteLine("Test 18: Cleared linked list Contains 'jumps' = {0}",
            sentence.Contains("jumps"));

        Console.ReadLine();
    }

    private static void Display(LinkedList<string> words, string test)
    {
        Console.WriteLine(test);
        foreach (string word in words)
        {
            Console.Write(word + " ");
        }
        Console.WriteLine();
        Console.WriteLine();
    }

    private static void IndicateNode(LinkedListNode<string> node, string test)
    {
        Console.WriteLine(test);
        if (node.List == null)
        {
            Console.WriteLine("Node '{0}' is not in the list.\n",
                node.Value);
            return;
        }

        StringBuilder result = new StringBuilder("(" + node.Value + ")");
        LinkedListNode<string> nodeP = node.Previous;

        while (nodeP != null)
        {
            result.Insert(0, nodeP.Value + " ");
            nodeP = nodeP.Previous;
        }

        node = node.Next;
        while (node != null)
        {
            result.Append(" " + node.Value);
            node = node.Next;
        }

        Console.WriteLine(result);
        Console.WriteLine();
    }
}

//This code example produces the following output:
//
//The linked list values:
//the fox jumps over the dog

//Test 1: Add 'today' to beginning of the list:
//today the fox jumps over the dog

//Test 2: Move first node to be last node:
//the fox jumps over the dog today

//Test 3: Change the last node to 'yesterday':
//the fox jumps over the dog yesterday

//Test 4: Move last node to be first node:
//yesterday the fox jumps over the dog

//Test 5: Indicate last occurence of 'the':
//the fox jumps over (the) dog

//Test 6: Add 'lazy' and 'old' after 'the':
//the fox jumps over (the) lazy old dog

//Test 7: Indicate the 'fox' node:
//the (fox) jumps over the lazy old dog

//Test 8: Add 'quick' and 'brown' before 'fox':
//the quick brown (fox) jumps over the lazy old dog

//Test 9: Indicate the 'dog' node:
//the quick brown fox jumps over the lazy old (dog)

//Test 10: Throw exception by adding node (fox) already in the list:
//Exception message: The LinkedList node belongs a LinkedList.

//Test 11: Move a referenced node (fox) before the current node (dog):
//the quick brown jumps over the lazy old fox (dog)

//Test 12: Remove current node (dog) and attempt to indicate it:
//Node 'dog' is not in the list.

//Test 13: Add node removed in test 11 after a referenced node (brown):
//the quick brown (dog) jumps over the lazy old fox

//Test 14: Remove node that has the value 'old':
//the quick brown dog jumps over the lazy fox

//Test 15: Remove last node, cast to ICollection, and add 'rhinoceros':
//the quick brown dog jumps over the lazy rhinoceros

//Test 16: Copy the list to an array:
//the
//quick
//brown
//dog
//jumps
//over
//the
//lazy
//rhinoceros

//Test 17: linked list Contains 'jumps'= True

//Test 18: Cleared linked list Contains 'jumps'  = False
//
```

## 注解

`LinkedList<T>`是常规用途的链接列表。它支持枚举器并实现`ICollection`接口，与`.NET Framework`中的其他集合类一致。

`LinkedList<T>`提供`LinkedListNode<T>`类型的单独节点，因此插入和删除是`O(1)`操作。

可以在同一列表中或另一个列表中删除节点并重新插入它们，这会导致堆上没有分配其他对象。由于列表还维护内部计数，因此获取`Count`属性是一个`O(1)`操作。

对象中每个`LinkedList<T>`节点的类型为`LinkedListNode<T>`。`LinkedList<T>`是双向链接的。

同时创建节点及其值时，包含引用类型的列表性能更好。`LinkedList<T>`接受`null`作为引用类型的有效`Value`属性，并允许重复值。

`LinkedList<T>`如果为空，则`First`和`Last`属性包含`null`。

`LinkedList<T>`类不支持可能使列表处于不一致状态的功能，列表在单个线程上保持一致。支持的唯一多线程方案时多线程读取操作。

## 构造函数

- `LinkedList<T>()`：初始化空的新实例。
- `LinkedList<T>(IEnumerable<T>)`：使用`IEnumerable`中的元素初始化。

## 属性

- `Count`：获取`LinkedList<T>`中实际包含的节点数。
- `First`：获取`LinkedList<T>`的第一个节点。
- `Last`：获取`LinkedList<T>`的最后一个节点。


