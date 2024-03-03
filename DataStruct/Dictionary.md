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

## 源码

```c#
private struct Entry {
    public int hashCode;    // Lower 31 bits of hash code, -1 if unused
    public int next;        // Index of next entry, -1 if last
    public TKey key;           // Key of entry
    public TValue value;         // Value of entry
}
 
private int[] buckets; // 内部维护的数据地址
private Entry[] entries; // 元素数组，用于维护哈希表中的数据
private int count; // 元素数量
private int version; 
private int freeList; // 空闲的列表
private int freeCount; // 空闲列表的元素数量
private IEqualityComparer<TKey> comparer; // 哈希表中的比较函数
private KeyCollection keys; // 键集合
private ValueCollection values; // 值集合
private Object _syncRoot;
```
`buckets`想在哈希函数与`entries`之间解耦的一层关系，哈希函数变化不再直接影响`entries`，`freeList`类似一个单链表，用于存储被释放出来的空间，即空链表，一般被优先存入数据。`freeCount`空链表的空位数量。

```C#
private void Initialize(int capacity) { 
    // 根据构造函数设定的初始容量，获取一个近似的质数
    int size = HashHelpers.GetPrime(capacity);
    buckets = new int[size];
    for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;
    entries = new Entry[size];
    freeList = -1;
}

// HashHelpers中
// Table of prime numbers to use as hash table sizes. 
// A typical resize algorithm would pick the smallest prime number in this array
// that is larger than twice the previous capacity. 
// Suppose our Hashtable currently has capacity x and enough elements are added 
// such that a resize needs to occur. Resizing first computes 2x then finds the 
// first prime in the table greater than 2x, i.e. if primes are ordered 
// p_1, p_2, ..., p_i, ..., it finds p_n such that p_n-1 < 2x < p_n. 
// Doubling is important for preserving the asymptotic complexity of the 
// hashtable operations such as add.  Having a prime guarantees that double 
// hashing does not lead to infinite loops.  IE, your hash function will be 
// h1(key) + i*h2(key), 0 <= i < size.  h2 and the size must be relatively prime.
public static readonly int[] primes = {
    3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369};

public static int GetPrime(int min) 
{
    if (min < 0)
        throw new ArgumentException(Environment.GetResourceString("Arg_HTCapacityOverflow"));
    Contract.EndContractBlock();
 
    for (int i = 0; i < primes.Length; i++) 
    {
        int prime = primes[i];
        if (prime >= min) return prime;
    }
 
    //outside of our predefined table. 
    //compute the hard way. 
    for (int i = (min | 1); i < Int32.MaxValue;i+=2) 
    {
        if (IsPrime(i) && ((i - 1) % Hashtable.HashPrime != 0))
            return i;
    }
    return min;
}
```
初始化Dictionary内部数组容器，`buckets`和`entries`，分别分配长度3。`size`哈希表的长度是质数，可以使元素更均匀地分布在每个节点上。`buckets`中的节点值-1表示空值。`freeList`为-1表示没有空链表。`buckets`和`freeList`所指向的数据其实全是存储于一块连续的内存空间`entries`之中。

```c#
public void Add(TKey key, TValue value) {
    Insert(key, value, true);
}

private void Insert(TKey key, TValue value, bool add) {
        
            if( key == null ) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
            }
 
            if (buckets == null) Initialize(0);
            int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
            int targetBucket = hashCode % buckets.Length;
 
#if FEATURE_RANDOMIZED_STRING_HASHING
            int collisionCount = 0; // 碰撞计数
#endif
 
            for (int i = buckets[targetBucket]; i >= 0; i = entries[i].next) {
                if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) { // 哈希值相同并且键相同
                    if (add) { 
                        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_AddingDuplicate); // 键重复
                    }
                    entries[i].value = value;
                    version++;
                    return;
                } 
 
#if FEATURE_RANDOMIZED_STRING_HASHING
                collisionCount++;
#endif
            }

            int index;
            if (freeCount > 0) {
                index = freeList;
                freeList = entries[index].next;
                freeCount--;
            }
            else {
                if (count == entries.Length)
                {
                    Resize();
                    targetBucket = hashCode % buckets.Length;
                }
                index = count;
                count++;
            }
 
            entries[index].hashCode = hashCode;
            entries[index].next = buckets[targetBucket];
            entries[index].key = key;
            entries[index].value = value;
            buckets[targetBucket] = index;
            version++;
 
#if FEATURE_RANDOMIZED_STRING_HASHING
 
#if FEATURE_CORECLR
            // In case we hit the collision threshold we'll need to switch to the comparer which is using randomized string hashing
            // in this case will be EqualityComparer<string>.Default.
            // Note, randomized string hashing is turned on by default on coreclr so EqualityComparer<string>.Default will 
            // be using randomized string hashing
 
            if (collisionCount > HashHelpers.HashCollisionThreshold && comparer == NonRandomizedStringEqualityComparer.Default) 
            {
                comparer = (IEqualityComparer<TKey>) EqualityComparer<string>.Default;
                Resize(entries.Length, true);
            }
#else
            if(collisionCount > HashHelpers.HashCollisionThreshold && HashHelpers.IsWellKnownEqualityComparer(comparer)) 
            {
                comparer = (IEqualityComparer<TKey>) HashHelpers.GetRandomizedEqualityComparer(comparer);
                Resize(entries.Length, true);
            }
#endif // FEATURE_CORECLR
 
#endif
 
}
```
1. 通过哈希函数寻址，计算出哈希地址（`buckets`的索引值）。
2. 判断`buckets`中映射到的值是否为-1（即为空位）。若不为-1，表示有冲突，遍历冲突链，不允许重复的键。
3. 判断是否有空链表，有则插入空链表的当前位置，将`freeList`指针后移，`freeCount`减一，否则将元素插入当前空位。在这一步，容量不足将自动扩容，若当前位置已经存在元素则将该元素的地址存在插入元素的next中，形成一个单链表的形式。

```c#
public bool Remove(TKey key) {
    if(key == null) {
        ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
    }
 
    if (buckets != null) {
        int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
        int bucket = hashCode % buckets.Length;
        int last = -1;
        for (int i = buckets[bucket]; i >= 0; last = i, i = entries[i].next) {
            if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) {
                if (last < 0) {
                    buckets[bucket] = entries[i].next;
                }
                else {
                    entries[last].next = entries[i].next;
                }
                entries[i].hashCode = -1;
                entries[i].next = freeList;
                entries[i].key = default(TKey);
                entries[i].value = default(TValue);
                freeList = i;
                freeCount++;
                version++;
                return true;
            }
        }
    }
    return false;
}
```


```c#
// ==++==
// 
//   Copyright (c) Microsoft Corporation.  All rights reserved.
// 
// ==--==
/*============================================================
**
** Class:  Dictionary
** 
** <OWNER>Microsoft</OWNER>
**
** Purpose: Generic hash table implementation
**
** #DictionaryVersusHashtableThreadSafety
** Hashtable has multiple reader/single writer (MR/SW) thread safety built into 
** certain methods and properties, whereas Dictionary doesn't. If you're 
** converting framework code that formerly used Hashtable to Dictionary, it's
** important to consider whether callers may have taken a dependence on MR/SW
** thread safety. If a reader writer lock is available, then that may be used
** with a Dictionary to get the same thread safety guarantee. 
** 
** Reader writer locks don't exist in silverlight, so we do the following as a
** result of removing non-generic collections from silverlight: 
** 1. If the Hashtable was fully synchronized, then we replace it with a 
**    Dictionary with full locks around reads/writes (same thread safety
**    guarantee).
** 2. Otherwise, the Hashtable has the default MR/SW thread safety behavior, 
**    so we do one of the following on a case-by-case basis:
**    a. If the ---- can be addressed by rearranging the code and using a temp
**       variable (for example, it's only populated immediately after created)
**       then we address the ---- this way and use Dictionary.
**    b. If there's concern about degrading performance with the increased 
**       locking, we ifdef with FEATURE_NONGENERIC_COLLECTIONS so we can at 
**       least use Hashtable in the desktop build, but Dictionary with full 
**       locks in silverlight builds. Note that this is heavier locking than 
**       MR/SW, but this is the only option without rewriting (or adding back)
**       the reader writer lock. 
**    c. If there's no performance concern (e.g. debug-only code) we 
**       consistently replace Hashtable with Dictionary plus full locks to 
**       reduce complexity.
**    d. Most of serialization is dead code in silverlight. Instead of updating
**       those Hashtable occurences in serialization, we carved out references 
**       to serialization such that this code doesn't need to build in 
**       silverlight. 
===========================================================*/
namespace System.Collections.Generic {
 
    using System;
    using System.Collections;
    using System.Diagnostics;
    using System.Diagnostics.Contracts;
    using System.Runtime.Serialization;
    using System.Security.Permissions;
 
    [DebuggerTypeProxy(typeof(Mscorlib_DictionaryDebugView<,>))]
    [DebuggerDisplay("Count = {Count}")]
    [Serializable]
    [System.Runtime.InteropServices.ComVisible(false)]
    public class Dictionary<TKey,TValue>: IDictionary<TKey,TValue>, IDictionary, IReadOnlyDictionary<TKey, TValue>, ISerializable, IDeserializationCallback  {
    
        private struct Entry {
            public int hashCode;    // Lower 31 bits of hash code, -1 if unused
            public int next;        // Index of next entry, -1 if last
            public TKey key;           // Key of entry
            public TValue value;         // Value of entry
        }
 
        private int[] buckets; // 内部维护的数据地址
        private Entry[] entries; // 元素数组，用于维护哈希表中的数据
        private int count; // 元素数量
        private int version; 
        private int freeList; // 空闲的列表
        private int freeCount; // 空闲列表的元素数量
        private IEqualityComparer<TKey> comparer; // 哈希表中的比较函数
        private KeyCollection keys; // 键集合
        private ValueCollection values; // 值集合
        private Object _syncRoot;
        
        // constants for serialization
        private const String VersionName = "Version";
        private const String HashSizeName = "HashSize";  // Must save buckets.Length
        private const String KeyValuePairsName = "KeyValuePairs";
        private const String ComparerName = "Comparer";
 
        public Dictionary(): this(0, null) {}
 
        public Dictionary(int capacity): this(capacity, null) {}
 
        public Dictionary(IEqualityComparer<TKey> comparer): this(0, comparer) {}
 
        public Dictionary(int capacity, IEqualityComparer<TKey> comparer) {
            if (capacity < 0) ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.capacity);
            if (capacity > 0) Initialize(capacity);
            this.comparer = comparer ?? EqualityComparer<TKey>.Default;
 
#if FEATURE_CORECLR
            if (HashHelpers.s_UseRandomizedStringHashing && comparer == EqualityComparer<string>.Default)
            {
                this.comparer = (IEqualityComparer<TKey>) NonRandomizedStringEqualityComparer.Default;
            }
#endif // FEATURE_CORECLR
        }
 
        public Dictionary(IDictionary<TKey,TValue> dictionary): this(dictionary, null) {}
 
        public Dictionary(IDictionary<TKey,TValue> dictionary, IEqualityComparer<TKey> comparer):
            this(dictionary != null? dictionary.Count: 0, comparer) {
 
            if( dictionary == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.dictionary);
            }
 
            foreach (KeyValuePair<TKey,TValue> pair in dictionary) {
                Add(pair.Key, pair.Value);
            }
        }
 
        protected Dictionary(SerializationInfo info, StreamingContext context) {
            //We can't do anything with the keys and values until the entire graph has been deserialized
            //and we have a resonable estimate that GetHashCode is not going to fail.  For the time being,
            //we'll just cache this.  The graph is not valid until OnDeserialization has been called.
            HashHelpers.SerializationInfoTable.Add(this, info);
        }
            
        public IEqualityComparer<TKey> Comparer {
            get {
                return comparer;                
            }               
        }
        
        public int Count {
            get { return count - freeCount; }
        }
 
        public KeyCollection Keys {
            get {
                Contract.Ensures(Contract.Result<KeyCollection>() != null);
                if (keys == null) keys = new KeyCollection(this);
                return keys;
            }
        }
 
        ICollection<TKey> IDictionary<TKey, TValue>.Keys {
            get {                
                if (keys == null) keys = new KeyCollection(this);                
                return keys;
            }
        }
 
        IEnumerable<TKey> IReadOnlyDictionary<TKey, TValue>.Keys {
            get {                
                if (keys == null) keys = new KeyCollection(this);                
                return keys;
            }
        }
 
        public ValueCollection Values {
            get {
                Contract.Ensures(Contract.Result<ValueCollection>() != null);
                if (values == null) values = new ValueCollection(this);
                return values;
            }
        }
 
        ICollection<TValue> IDictionary<TKey, TValue>.Values {
            get {                
                if (values == null) values = new ValueCollection(this);
                return values;
            }
        }
 
        IEnumerable<TValue> IReadOnlyDictionary<TKey, TValue>.Values {
            get {                
                if (values == null) values = new ValueCollection(this);
                return values;
            }
        }
 
        public TValue this[TKey key] {
            get {
                int i = FindEntry(key);
                if (i >= 0) return entries[i].value;
                ThrowHelper.ThrowKeyNotFoundException();
                return default(TValue);
            }
            set {
                Insert(key, value, false);
            }
        }
 
        public void Add(TKey key, TValue value) {
            Insert(key, value, true);
        }
 
        void ICollection<KeyValuePair<TKey, TValue>>.Add(KeyValuePair<TKey, TValue> keyValuePair) {
            Add(keyValuePair.Key, keyValuePair.Value);
        }
 
        bool ICollection<KeyValuePair<TKey, TValue>>.Contains(KeyValuePair<TKey, TValue> keyValuePair) {
            int i = FindEntry(keyValuePair.Key);
            if( i >= 0 && EqualityComparer<TValue>.Default.Equals(entries[i].value, keyValuePair.Value)) {
                return true;
            }
            return false;
        }
 
        bool ICollection<KeyValuePair<TKey, TValue>>.Remove(KeyValuePair<TKey, TValue> keyValuePair) {
            int i = FindEntry(keyValuePair.Key);
            if( i >= 0 && EqualityComparer<TValue>.Default.Equals(entries[i].value, keyValuePair.Value)) {
                Remove(keyValuePair.Key);
                return true;
            }
            return false;
        }
 
        public void Clear() {
            if (count > 0) {
                for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;
                Array.Clear(entries, 0, count);
                freeList = -1;
                count = 0;
                freeCount = 0;
                version++;
            }
        }
 
        public bool ContainsKey(TKey key) {
            return FindEntry(key) >= 0;
        }
 
        public bool ContainsValue(TValue value) {
            if (value == null) {
                for (int i = 0; i < count; i++) {
                    if (entries[i].hashCode >= 0 && entries[i].value == null) return true;
                }
            }
            else {
                EqualityComparer<TValue> c = EqualityComparer<TValue>.Default;
                for (int i = 0; i < count; i++) {
                    if (entries[i].hashCode >= 0 && c.Equals(entries[i].value, value)) return true;
                }
            }
            return false;
        }
 
        private void CopyTo(KeyValuePair<TKey,TValue>[] array, int index) {
            if (array == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.array);
            }
            
            if (index < 0 || index > array.Length ) {
                ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
            }
 
            if (array.Length - index < Count) {
                ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_ArrayPlusOffTooSmall);
            }
 
            int count = this.count;
            Entry[] entries = this.entries;
            for (int i = 0; i < count; i++) {
                if (entries[i].hashCode >= 0) {
                    array[index++] = new KeyValuePair<TKey,TValue>(entries[i].key, entries[i].value);
                }
            }
        }
 
        public Enumerator GetEnumerator() {
            return new Enumerator(this, Enumerator.KeyValuePair);
        }
 
        IEnumerator<KeyValuePair<TKey, TValue>> IEnumerable<KeyValuePair<TKey, TValue>>.GetEnumerator() {
            return new Enumerator(this, Enumerator.KeyValuePair);
        }        
 
        [System.Security.SecurityCritical]  // auto-generated_required
        public virtual void GetObjectData(SerializationInfo info, StreamingContext context) {
            if (info==null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.info);
            }
            info.AddValue(VersionName, version);
 
#if FEATURE_RANDOMIZED_STRING_HASHING
            info.AddValue(ComparerName, HashHelpers.GetEqualityComparerForSerialization(comparer), typeof(IEqualityComparer<TKey>));
#else
            info.AddValue(ComparerName, comparer, typeof(IEqualityComparer<TKey>));
#endif
 
            info.AddValue(HashSizeName, buckets == null ? 0 : buckets.Length); //This is the length of the bucket array.
            if( buckets != null) {
                KeyValuePair<TKey, TValue>[] array = new KeyValuePair<TKey, TValue>[Count];
                CopyTo(array, 0);
                info.AddValue(KeyValuePairsName, array, typeof(KeyValuePair<TKey, TValue>[]));
            }
        }
 
        private int FindEntry(TKey key) {
            if( key == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
            }
 
            if (buckets != null) {
                int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
                for (int i = buckets[hashCode % buckets.Length]; i >= 0; i = entries[i].next) {
                    if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) return i;
                }
            }
            return -1;
        }
 
        private void Initialize(int capacity) { 
            // 根据构造函数设定的初始容量，获取一个近似的质数
            int size = HashHelpers.GetPrime(capacity);
            buckets = new int[size];
            for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;
            entries = new Entry[size];
            freeList = -1;
        }
 
        private void Insert(TKey key, TValue value, bool add) {
        
            if( key == null ) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
            }
 
            if (buckets == null) Initialize(0);
            int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
            int targetBucket = hashCode % buckets.Length;
 
#if FEATURE_RANDOMIZED_STRING_HASHING
            int collisionCount = 0;
#endif
 
            for (int i = buckets[targetBucket]; i >= 0; i = entries[i].next) {
                if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) {
                    if (add) { 
                        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_AddingDuplicate);
                    }
                    entries[i].value = value;
                    version++;
                    return;
                } 
 
#if FEATURE_RANDOMIZED_STRING_HASHING
                collisionCount++;
#endif
            }
            int index;
            if (freeCount > 0) {
                index = freeList;
                freeList = entries[index].next;
                freeCount--;
            }
            else {
                if (count == entries.Length)
                {
                    Resize();
                    targetBucket = hashCode % buckets.Length;
                }
                index = count;
                count++;
            }
 
            entries[index].hashCode = hashCode;
            entries[index].next = buckets[targetBucket];
            entries[index].key = key;
            entries[index].value = value;
            buckets[targetBucket] = index;
            version++;
 
#if FEATURE_RANDOMIZED_STRING_HASHING
 
#if FEATURE_CORECLR
            // In case we hit the collision threshold we'll need to switch to the comparer which is using randomized string hashing
            // in this case will be EqualityComparer<string>.Default.
            // Note, randomized string hashing is turned on by default on coreclr so EqualityComparer<string>.Default will 
            // be using randomized string hashing
 
            if (collisionCount > HashHelpers.HashCollisionThreshold && comparer == NonRandomizedStringEqualityComparer.Default) 
            {
                comparer = (IEqualityComparer<TKey>) EqualityComparer<string>.Default;
                Resize(entries.Length, true);
            }
#else
            if(collisionCount > HashHelpers.HashCollisionThreshold && HashHelpers.IsWellKnownEqualityComparer(comparer)) 
            {
                comparer = (IEqualityComparer<TKey>) HashHelpers.GetRandomizedEqualityComparer(comparer);
                Resize(entries.Length, true);
            }
#endif // FEATURE_CORECLR
 
#endif
 
        }
 
        public virtual void OnDeserialization(Object sender) {
            SerializationInfo siInfo;
            HashHelpers.SerializationInfoTable.TryGetValue(this, out siInfo);
            
            if (siInfo==null) {
                // It might be necessary to call OnDeserialization from a container if the container object also implements
                // OnDeserialization. However, remoting will call OnDeserialization again.
                // We can return immediately if this function is called twice. 
                // Note we set remove the serialization info from the table at the end of this method.
                return;
            }            
            
            int realVersion = siInfo.GetInt32(VersionName);
            int hashsize = siInfo.GetInt32(HashSizeName);
            comparer   = (IEqualityComparer<TKey>)siInfo.GetValue(ComparerName, typeof(IEqualityComparer<TKey>));
            
            if( hashsize != 0) {
                buckets = new int[hashsize];
                for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;
                entries = new Entry[hashsize];
                freeList = -1;
 
                KeyValuePair<TKey, TValue>[] array = (KeyValuePair<TKey, TValue>[]) 
                    siInfo.GetValue(KeyValuePairsName, typeof(KeyValuePair<TKey, TValue>[]));
 
                if (array==null) {
                    ThrowHelper.ThrowSerializationException(ExceptionResource.Serialization_MissingKeys);
                }
 
                for (int i=0; i<array.Length; i++) {
                    if ( array[i].Key == null) {
                        ThrowHelper.ThrowSerializationException(ExceptionResource.Serialization_NullKey);
                    }
                    Insert(array[i].Key, array[i].Value, true);
                }
            }
            else {
                buckets = null;
            }
 
            version = realVersion;
            HashHelpers.SerializationInfoTable.Remove(this);
        }
 
        private void Resize() {
            Resize(HashHelpers.ExpandPrime(count), false);
        }
 
        private void Resize(int newSize, bool forceNewHashCodes) {
            Contract.Assert(newSize >= entries.Length);
            int[] newBuckets = new int[newSize];
            for (int i = 0; i < newBuckets.Length; i++) newBuckets[i] = -1;
            Entry[] newEntries = new Entry[newSize];
            Array.Copy(entries, 0, newEntries, 0, count);
            if(forceNewHashCodes) {
                for (int i = 0; i < count; i++) {
                    if(newEntries[i].hashCode != -1) {
                        newEntries[i].hashCode = (comparer.GetHashCode(newEntries[i].key) & 0x7FFFFFFF);
                    }
                }
            }
            for (int i = 0; i < count; i++) {
                if (newEntries[i].hashCode >= 0) {
                    int bucket = newEntries[i].hashCode % newSize;
                    newEntries[i].next = newBuckets[bucket];
                    newBuckets[bucket] = i;
                }
            }
            buckets = newBuckets;
            entries = newEntries;
        }
 
        public bool Remove(TKey key) {
            if(key == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);
            }
 
            if (buckets != null) {
                int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
                int bucket = hashCode % buckets.Length;
                int last = -1;
                for (int i = buckets[bucket]; i >= 0; last = i, i = entries[i].next) {
                    if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) {
                        if (last < 0) {
                            buckets[bucket] = entries[i].next;
                        }
                        else {
                            entries[last].next = entries[i].next;
                        }
                        entries[i].hashCode = -1;
                        entries[i].next = freeList;
                        entries[i].key = default(TKey);
                        entries[i].value = default(TValue);
                        freeList = i;
                        freeCount++;
                        version++;
                        return true;
                    }
                }
            }
            return false;
        }
 
        public bool TryGetValue(TKey key, out TValue value) {
            int i = FindEntry(key);
            if (i >= 0) {
                value = entries[i].value;
                return true;
            }
            value = default(TValue);
            return false;
        }
 
        // This is a convenience method for the internal callers that were converted from using Hashtable.
        // Many were combining key doesn't exist and key exists but null value (for non-value types) checks.
        // This allows them to continue getting that behavior with minimal code delta. This is basically
        // TryGetValue without the out param
        internal TValue GetValueOrDefault(TKey key) {
            int i = FindEntry(key);
            if (i >= 0) {
                return entries[i].value;
            }
            return default(TValue);
        }
 
        bool ICollection<KeyValuePair<TKey,TValue>>.IsReadOnly {
            get { return false; }
        }
 
        void ICollection<KeyValuePair<TKey,TValue>>.CopyTo(KeyValuePair<TKey,TValue>[] array, int index) {
            CopyTo(array, index);
        }
 
        void ICollection.CopyTo(Array array, int index) {
            if (array == null) {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.array);
            }
            
            if (array.Rank != 1) {
                ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_RankMultiDimNotSupported);
            }
 
            if( array.GetLowerBound(0) != 0 ) {
                ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_NonZeroLowerBound);
            }
            
            if (index < 0 || index > array.Length) {
                ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
            }
 
            if (array.Length - index < Count) {
                ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_ArrayPlusOffTooSmall);
            }
            
            KeyValuePair<TKey,TValue>[] pairs = array as KeyValuePair<TKey,TValue>[];
            if (pairs != null) {
                CopyTo(pairs, index);
            }
            else if( array is DictionaryEntry[]) {
                DictionaryEntry[] dictEntryArray = array as DictionaryEntry[];
                Entry[] entries = this.entries;
                for (int i = 0; i < count; i++) {
                    if (entries[i].hashCode >= 0) {
                        dictEntryArray[index++] = new DictionaryEntry(entries[i].key, entries[i].value);
                    }
                }                
            }
            else {
                object[] objects = array as object[];
                if (objects == null) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidArrayType);
                }
 
                try {
                    int count = this.count;
                    Entry[] entries = this.entries;
                    for (int i = 0; i < count; i++) {
                        if (entries[i].hashCode >= 0) {
                            objects[index++] = new KeyValuePair<TKey,TValue>(entries[i].key, entries[i].value);
                        }
                    }
                }
                catch(ArrayTypeMismatchException) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidArrayType);
                }
            }
        }
 
        IEnumerator IEnumerable.GetEnumerator() {
            return new Enumerator(this, Enumerator.KeyValuePair);
        }
    
        bool ICollection.IsSynchronized {
            get { return false; }
        }
 
        object ICollection.SyncRoot { 
            get { 
                if( _syncRoot == null) {
                    System.Threading.Interlocked.CompareExchange<Object>(ref _syncRoot, new Object(), null);    
                }
                return _syncRoot; 
            }
        }
 
        bool IDictionary.IsFixedSize {
            get { return false; }
        }
 
        bool IDictionary.IsReadOnly {
            get { return false; }
        }
 
        ICollection IDictionary.Keys {
            get { return (ICollection)Keys; }
        }
    
        ICollection IDictionary.Values {
            get { return (ICollection)Values; }
        }
    
        object IDictionary.this[object key] {
            get { 
                if( IsCompatibleKey(key)) {                
                    int i = FindEntry((TKey)key);
                    if (i >= 0) { 
                        return entries[i].value;                
                    }
                }
                return null;
            }
            set {                 
                if (key == null)
                {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);                          
                }
                ThrowHelper.IfNullAndNullsAreIllegalThenThrow<TValue>(value, ExceptionArgument.value);
 
                try {
                    TKey tempKey = (TKey)key;
                    try {
                        this[tempKey] = (TValue)value; 
                    }
                    catch (InvalidCastException) { 
                        ThrowHelper.ThrowWrongValueTypeArgumentException(value, typeof(TValue));   
                    }
                }
                catch (InvalidCastException) { 
                    ThrowHelper.ThrowWrongKeyTypeArgumentException(key, typeof(TKey));
                }
            }
        }
 
        private static bool IsCompatibleKey(object key) {
            if( key == null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);                          
                }
            return (key is TKey); 
        }
    
        void IDictionary.Add(object key, object value) {            
            if (key == null)
            {
                ThrowHelper.ThrowArgumentNullException(ExceptionArgument.key);                          
            }
            ThrowHelper.IfNullAndNullsAreIllegalThenThrow<TValue>(value, ExceptionArgument.value);
 
            try {
                TKey tempKey = (TKey)key;
 
                try {
                    Add(tempKey, (TValue)value);
                }
                catch (InvalidCastException) { 
                    ThrowHelper.ThrowWrongValueTypeArgumentException(value, typeof(TValue));   
                }
            }
            catch (InvalidCastException) { 
                ThrowHelper.ThrowWrongKeyTypeArgumentException(key, typeof(TKey));
            }
        }
    
        bool IDictionary.Contains(object key) {    
            if(IsCompatibleKey(key)) {
                return ContainsKey((TKey)key);
            }
       
            return false;
        }
    
        IDictionaryEnumerator IDictionary.GetEnumerator() {
            return new Enumerator(this, Enumerator.DictEntry);
        }
    
        void IDictionary.Remove(object key) {            
            if(IsCompatibleKey(key)) {
                Remove((TKey)key);
            }
        }
 
        [Serializable]
        public struct Enumerator: IEnumerator<KeyValuePair<TKey,TValue>>,
            IDictionaryEnumerator
        {
            private Dictionary<TKey,TValue> dictionary;
            private int version;
            private int index;
            private KeyValuePair<TKey,TValue> current;
            private int getEnumeratorRetType;  // What should Enumerator.Current return?
            
            internal const int DictEntry = 1;
            internal const int KeyValuePair = 2;
 
            internal Enumerator(Dictionary<TKey,TValue> dictionary, int getEnumeratorRetType) {
                this.dictionary = dictionary;
                version = dictionary.version;
                index = 0;
                this.getEnumeratorRetType = getEnumeratorRetType;
                current = new KeyValuePair<TKey, TValue>();
            }
 
            public bool MoveNext() {
                if (version != dictionary.version) {
                    ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
                }
 
                // Use unsigned comparison since we set index to dictionary.count+1 when the enumeration ends.
                // dictionary.count+1 could be negative if dictionary.count is Int32.MaxValue
                while ((uint)index < (uint)dictionary.count) {
                    if (dictionary.entries[index].hashCode >= 0) {
                        current = new KeyValuePair<TKey, TValue>(dictionary.entries[index].key, dictionary.entries[index].value);
                        index++;
                        return true;
                    }
                    index++;
                }
 
                index = dictionary.count + 1;
                current = new KeyValuePair<TKey, TValue>();
                return false;
            }
 
            public KeyValuePair<TKey,TValue> Current {
                get { return current; }
            }
 
            public void Dispose() {
            }
 
            object IEnumerator.Current {
                get { 
                    if( index == 0 || (index == dictionary.count + 1)) {
                        ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);                        
                    }      
 
                    if (getEnumeratorRetType == DictEntry) {
                        return new System.Collections.DictionaryEntry(current.Key, current.Value);
                    } else {
                        return new KeyValuePair<TKey, TValue>(current.Key, current.Value);
                    }
                }
            }
 
            void IEnumerator.Reset() {
                if (version != dictionary.version) {
                    ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
                }
 
                index = 0;
                current = new KeyValuePair<TKey, TValue>();    
            }
 
            DictionaryEntry IDictionaryEnumerator.Entry {
                get { 
                    if( index == 0 || (index == dictionary.count + 1)) {
                         ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);                        
                    }                        
                    
                    return new DictionaryEntry(current.Key, current.Value); 
                }
            }
 
            object IDictionaryEnumerator.Key {
                get { 
                    if( index == 0 || (index == dictionary.count + 1)) {
                         ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);                        
                    }                        
                    
                    return current.Key; 
                }
            }
 
            object IDictionaryEnumerator.Value {
                get { 
                    if( index == 0 || (index == dictionary.count + 1)) {
                         ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);                        
                    }                        
                    
                    return current.Value; 
                }
            }
        }
 
        [DebuggerTypeProxy(typeof(Mscorlib_DictionaryKeyCollectionDebugView<,>))]
        [DebuggerDisplay("Count = {Count}")]        
        [Serializable]
        public sealed class KeyCollection: ICollection<TKey>, ICollection, IReadOnlyCollection<TKey>
        {
            private Dictionary<TKey,TValue> dictionary;
 
            public KeyCollection(Dictionary<TKey,TValue> dictionary) {
                if (dictionary == null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.dictionary);
                }
                this.dictionary = dictionary;
            }
 
            public Enumerator GetEnumerator() {
                return new Enumerator(dictionary);
            }
 
            public void CopyTo(TKey[] array, int index) {
                if (array == null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.array);
                }
 
                if (index < 0 || index > array.Length) {
                    ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
                }
 
                if (array.Length - index < dictionary.Count) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_ArrayPlusOffTooSmall);
                }
                
                int count = dictionary.count;
                Entry[] entries = dictionary.entries;
                for (int i = 0; i < count; i++) {
                    if (entries[i].hashCode >= 0) array[index++] = entries[i].key;
                }
            }
 
            public int Count {
                get { return dictionary.Count; }
            }
 
            bool ICollection<TKey>.IsReadOnly {
                get { return true; }
            }
 
            void ICollection<TKey>.Add(TKey item){
                ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_KeyCollectionSet);
            }
            
            void ICollection<TKey>.Clear(){
                ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_KeyCollectionSet);
            }
 
            bool ICollection<TKey>.Contains(TKey item){
                return dictionary.ContainsKey(item);
            }
 
            bool ICollection<TKey>.Remove(TKey item){
                ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_KeyCollectionSet);
                return false;
            }
            
            IEnumerator<TKey> IEnumerable<TKey>.GetEnumerator() {
                return new Enumerator(dictionary);
            }
 
            IEnumerator IEnumerable.GetEnumerator() {
                return new Enumerator(dictionary);                
            }
 
            void ICollection.CopyTo(Array array, int index) {
                if (array==null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.array);
                }
 
                if (array.Rank != 1) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_RankMultiDimNotSupported);
                }
 
                if( array.GetLowerBound(0) != 0 ) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_NonZeroLowerBound);
                }
 
                if (index < 0 || index > array.Length) {
                    ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
                }
 
                if (array.Length - index < dictionary.Count) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_ArrayPlusOffTooSmall);
                }
                
                TKey[] keys = array as TKey[];
                if (keys != null) {
                    CopyTo(keys, index);
                }
                else {
                    object[] objects = array as object[];
                    if (objects == null) {
                        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidArrayType);
                    }
                                         
                    int count = dictionary.count;
                    Entry[] entries = dictionary.entries;
                    try {
                        for (int i = 0; i < count; i++) {
                            if (entries[i].hashCode >= 0) objects[index++] = entries[i].key;
                        }
                    }                    
                    catch(ArrayTypeMismatchException) {
                        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidArrayType);
                    }
                }
            }
 
            bool ICollection.IsSynchronized {
                get { return false; }
            }
 
            Object ICollection.SyncRoot { 
                get { return ((ICollection)dictionary).SyncRoot; }
            }
 
            [Serializable]
            public struct Enumerator : IEnumerator<TKey>, System.Collections.IEnumerator
            {
                private Dictionary<TKey, TValue> dictionary;
                private int index;
                private int version;
                private TKey currentKey;
            
                internal Enumerator(Dictionary<TKey, TValue> dictionary) {
                    this.dictionary = dictionary;
                    version = dictionary.version;
                    index = 0;
                    currentKey = default(TKey);                    
                }
 
                public void Dispose() {
                }
 
                public bool MoveNext() {
                    if (version != dictionary.version) {
                        ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
                    }
 
                    while ((uint)index < (uint)dictionary.count) {
                        if (dictionary.entries[index].hashCode >= 0) {
                            currentKey = dictionary.entries[index].key;
                            index++;
                            return true;
                        }
                        index++;
                    }
 
                    index = dictionary.count + 1;
                    currentKey = default(TKey);
                    return false;
                }
                
                public TKey Current {
                    get {                        
                        return currentKey;
                    }
                }
 
                Object System.Collections.IEnumerator.Current {
                    get {                      
                        if( index == 0 || (index == dictionary.count + 1)) {
                             ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);                        
                        }                        
                        
                        return currentKey;
                    }
                }
                
                void System.Collections.IEnumerator.Reset() {
                    if (version != dictionary.version) {
                        ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);                        
                    }
 
                    index = 0;                    
                    currentKey = default(TKey);
                }
            }                        
        }
 
        [DebuggerTypeProxy(typeof(Mscorlib_DictionaryValueCollectionDebugView<,>))]
        [DebuggerDisplay("Count = {Count}")]
        [Serializable]
        public sealed class ValueCollection: ICollection<TValue>, ICollection, IReadOnlyCollection<TValue>
        {
            private Dictionary<TKey,TValue> dictionary;
 
            public ValueCollection(Dictionary<TKey,TValue> dictionary) {
                if (dictionary == null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.dictionary);
                }
                this.dictionary = dictionary;
            }
 
            public Enumerator GetEnumerator() {
                return new Enumerator(dictionary);                
            }
 
            public void CopyTo(TValue[] array, int index) {
                if (array == null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.array);
                }
 
                if (index < 0 || index > array.Length) {
                    ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
                }
 
                if (array.Length - index < dictionary.Count) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_ArrayPlusOffTooSmall);
                }
                
                int count = dictionary.count;
                Entry[] entries = dictionary.entries;
                for (int i = 0; i < count; i++) {
                    if (entries[i].hashCode >= 0) array[index++] = entries[i].value;
                }
            }
 
            public int Count {
                get { return dictionary.Count; }
            }
 
            bool ICollection<TValue>.IsReadOnly {
                get { return true; }
            }
 
            void ICollection<TValue>.Add(TValue item){
                ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_ValueCollectionSet);
            }
 
            bool ICollection<TValue>.Remove(TValue item){
                ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_ValueCollectionSet);
                return false;
            }
 
            void ICollection<TValue>.Clear(){
                ThrowHelper.ThrowNotSupportedException(ExceptionResource.NotSupported_ValueCollectionSet);
            }
 
            bool ICollection<TValue>.Contains(TValue item){
                return dictionary.ContainsValue(item);
            }
 
            IEnumerator<TValue> IEnumerable<TValue>.GetEnumerator() {
                return new Enumerator(dictionary);
            }
 
            IEnumerator IEnumerable.GetEnumerator() {
                return new Enumerator(dictionary);                
            }
 
            void ICollection.CopyTo(Array array, int index) {
                if (array == null) {
                    ThrowHelper.ThrowArgumentNullException(ExceptionArgument.array);
                }
 
                if (array.Rank != 1) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_RankMultiDimNotSupported);
                }
 
                if( array.GetLowerBound(0) != 0 ) {
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_NonZeroLowerBound);
                }
 
                if (index < 0 || index > array.Length) { 
                    ThrowHelper.ThrowArgumentOutOfRangeException(ExceptionArgument.index, ExceptionResource.ArgumentOutOfRange_NeedNonNegNum);
                }
 
                if (array.Length - index < dictionary.Count)
                    ThrowHelper.ThrowArgumentException(ExceptionResource.Arg_ArrayPlusOffTooSmall);
                
                TValue[] values = array as TValue[];
                if (values != null) {
                    CopyTo(values, index);
                }
                else {
                    object[] objects = array as object[];
                    if (objects == null) {
                        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidArrayType);
                    }
 
                    int count = dictionary.count;
                    Entry[] entries = dictionary.entries;
                    try {
                        for (int i = 0; i < count; i++) {
                            if (entries[i].hashCode >= 0) objects[index++] = entries[i].value;
                        }
                    }
                    catch(ArrayTypeMismatchException) {
                        ThrowHelper.ThrowArgumentException(ExceptionResource.Argument_InvalidArrayType);
                    }
                }
            }
 
            bool ICollection.IsSynchronized {
                get { return false; }
            }
 
            Object ICollection.SyncRoot { 
                get { return ((ICollection)dictionary).SyncRoot; }
            }
 
            [Serializable]
            public struct Enumerator : IEnumerator<TValue>, System.Collections.IEnumerator
            {
                private Dictionary<TKey, TValue> dictionary;
                private int index;
                private int version;
                private TValue currentValue;
            
                internal Enumerator(Dictionary<TKey, TValue> dictionary) {
                    this.dictionary = dictionary;
                    version = dictionary.version;
                    index = 0;
                    currentValue = default(TValue);
                }
 
                public void Dispose() {
                }
 
                public bool MoveNext() {                    
                    if (version != dictionary.version) {
                        ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
                    }
                    
                    while ((uint)index < (uint)dictionary.count) {
                        if (dictionary.entries[index].hashCode >= 0) {
                            currentValue = dictionary.entries[index].value;
                            index++;
                            return true;
                        }
                        index++;
                    }
                    index = dictionary.count + 1;
                    currentValue = default(TValue);
                    return false;
                }
                
                public TValue Current {
                    get {                        
                        return currentValue;
                    }
                }
 
                Object System.Collections.IEnumerator.Current {
                    get {                      
                        if( index == 0 || (index == dictionary.count + 1)) {
                             ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumOpCantHappen);                        
                        }                        
                        
                        return currentValue;
                    }
                }
                
                void System.Collections.IEnumerator.Reset() {
                    if (version != dictionary.version) {
                        ThrowHelper.ThrowInvalidOperationException(ExceptionResource.InvalidOperation_EnumFailedVersion);
                    }
                    index = 0;                    
                    currentValue = default(TValue);
                }
            }
        }
    }
}
```