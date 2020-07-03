# C# Dictionary字典源码分析与自定义Dictionary字典

## 官网源码地址：

[https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/dictionary.cs)

## 关键点

- Dictionary实际容器为Entry[] entries 结构体数组

 ```c#
private struct Entry
{
    public int hashCode;
    public int next;
    public TKey key;
    public TValue value;
}
 ```

- entries数组默认长度为3
- 如果初始化指定长度 entries数组长度为大于指定长度最接近的质数
- Dictionary查找元素的时间复杂度接近`O(1)`是因为
  - 定义了一个int[] buckets Hash桶数组
  - 获取key了hashCode 
  - 用hashCode除buckets长度取余做为索引（这也是为什么长度为质数的原因）
- 防止hashCode冲突Dictionary使用了拉链法
  - 使用Entry.next保存冲突项index
- 当添加值时冲突达到100次会使用新的hashCode方法解决
- 当Entry[] entries或int[] buckets满了的时候会自动扩了当前容量的2倍
- 扩容后会重现计算hashcode赋值索引
- 查找元素是会判断hashcode和key.Equals

## 优化点

- 已知容器大小的情况 直接初始化对应大小
- key最好用简单类型 
  - 如果一定要用枚举或引用类型 
  - 初始化字典的时候赋值对比函数
  - 或者实现IEquatable
- 获取值得时候使用TryGetValue不需要先判断ContainsKey

## 自定义Dictionary字典

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.Runtime.Serialization;

public interface IMyDictionary<TKey, TValue> : IEnumerable, ICollection<KeyValuePair<TKey, TValue>>, IEnumerable<KeyValuePair<TKey, TValue>>
{
    TValue this[TKey key] { get; set; }

    ICollection<TKey> Keys { get; }
    ICollection<TValue> Values { get; }

    void Add(TKey key, TValue value);
    bool ContainsKey(TKey key);
    bool Remove(TKey key);
    bool TryGetValue(TKey key, out TValue value);
}


public class MyDictionary<TKey, TValue>
{
    private struct Entry
    {
        public int hashCode;
        public int next;
        public TKey key;
        public TValue value;
    }
    /// <summary>
    /// Hash桶
    /// </summary>
    private int[] buckets;
    /// <summary>
    /// 所有键值对条目数组
    /// </summary>
    private Entry[] entries;
    /// <summary>
    /// 当前entries的index位置
    /// </summary>
    private int count;
    /// <summary>
    /// 当前版本，防止迭代过程中集合被更改
    /// </summary>
    private int version;
    /// <summary>
    /// entries空闲索引
    /// </summary>
    private int freeList;
    /// <summary>
    /// entries空闲数量
    /// </summary>
    private int freeCount;
    /// <summary>
    /// 对比函数
    /// </summary>
    private IEqualityComparer<TKey> comparer;
    /// <summary>
    /// key集合
    /// </summary>
    private KeyCollection keys;
    /// <summary>
    /// value集合
    /// </summary>
    private ValueCollection values;

    public MyDictionary() : this(0, null) { }

    public MyDictionary(IEqualityComparer<TKey> comparer) : this(0, comparer) { }

    public MyDictionary(IDictionary<TKey, TValue> dictionary) : this(dictionary, null) { }

    public MyDictionary(int capacity) : this(capacity, null) { }

    public MyDictionary(IDictionary<TKey, TValue> dictionary, IEqualityComparer<TKey> comparer) : this(dictionary != null ? dictionary.Count : 0, comparer)
    {
        if (dictionary == null) { return; }//异常

        foreach (KeyValuePair<TKey, TValue> pair in dictionary)
        {
            Add(pair.Key, pair.Value);
        }
    }

    public MyDictionary(int capacity, IEqualityComparer<TKey> comparer)
    {
        if (capacity < 0) { throw new Exception("capacity 异常"); }
        if (capacity > 0) Initialize(capacity);
        this.comparer = comparer ?? EqualityComparer<TKey>.Default;
    }

    public TValue this[TKey key]
    {
        get
        {
            int i = FindEntry(key);
            if (i >= 0) return entries[i].value;

            throw new Exception("TKey 异常");
        }
        set
        {
            Insert(key, value, false);
        }
    }

    public IEqualityComparer<TKey> Comparer
    {
        get { return comparer; }
    }

    public KeyCollection Keys
    {
        get
        {
            if (keys == null) keys = new KeyCollection(this);
            return keys;
        }
    }
    public ValueCollection Values
    {
        get
        {
            if (values == null) values = new ValueCollection(this);
            return values;
        }
    }
    public int Count
    {
        get { return count - freeCount; }
    }

    public void Add(TKey key, TValue value)
    {
        Insert(key, value, true);
    }
    public void Clear()
    {
        if (count > 0)
        {
            for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;
            Array.Clear(entries, 0, count);
            freeList = -1;
            count = 0;
            freeCount = 0;
            version++;
        }
    }
    public bool ContainsKey(TKey key)
    {
        return FindEntry(key) >= 0;
    }
    public bool ContainsValue(TValue value)
    {
        if (value == null)
        {
            for (int i = 0; i < count; i++)
            {
                if (entries[i].hashCode >= 0 && entries[i].value == null) return true;
            }
        }
        else
        {
            EqualityComparer<TValue> c = EqualityComparer<TValue>.Default;
            for (int i = 0; i < count; i++)
            {
                if (entries[i].hashCode >= 0 && c.Equals(entries[i].value, value)) return true;
            }
        }
        return false;
    }
    public Enumerator GetEnumerator()
    {
        return new Enumerator(this, Enumerator.KeyValuePair);
    }
    public virtual void GetObjectData(SerializationInfo info, StreamingContext context)
    {
    }
    public virtual void OnDeserialization(object sender)
    {

    }
    public bool Remove(TKey key)
    {
        if (key == null)
        {
            throw new Exception("异常");
        }

        if (buckets != null)
        {
            int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
            int bucket = hashCode % buckets.Length;
            int last = -1;
            for (int i = buckets[bucket]; i >= 0; last = i, i = entries[i].next)
            {
                if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key))
                {
                    if (last < 0)
                    {
                        buckets[bucket] = entries[i].next;//找到元素后，如果last< 0，代表当前是bucket中最后一个元素，那么直接让bucket内下标赋值为 entries[i].next即可
                    }
                    else
                    {
                        entries[last].next = entries[i].next;//last不小于0，代表当前元素处于bucket单链表中间位置，需要将该元素的头结点和尾节点相连起来,防止链表中断
                    }
                    entries[i].hashCode = -1;
                    entries[i].next = freeList;
                    entries[i].key = default(TKey);
                    entries[i].value = default(TValue);
                    freeList = i;//freeList等于当前的entry位置，下一次Add元素会优先Add到该位置
                    freeCount++;
                    version++;
                    return true;
                }
            }
        }
        return false;
    }
    public bool TryGetValue(TKey key, out TValue value)
    {
        int i = FindEntry(key);
        if (i >= 0)
        {
            value = entries[i].value;
            return true;
        }
        value = default(TValue);
        return false;
    }


    /// <summary>
    /// 初始化
    /// </summary>
    /// <param name="capacity">容量</param>
    private void Initialize(int capacity)
    {
        int size = HashHelpers.GetPrime(capacity);//获取该容量匹配的质数
        buckets = new int[size];//初始化桶数组
        for (int i = 0; i < buckets.Length; i++) buckets[i] = -1;//桶数组全部赋值为-1
        entries = new Entry[size];//初始化容量
        freeList = -1;
    }

    /// <summary>
    /// 插入
    /// </summary>
    /// <param name="key"></param>
    /// <param name="value"></param>
    /// <param name="add"></param>
    private void Insert(TKey key, TValue value, bool add)
    {

        if (key == null)
        {
            throw new Exception("key == null 异常");
        }

        if (buckets == null) Initialize(0);
        int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;//01111111 11111111 11111111 11111111 忽略符号位
        int targetBucket = hashCode % buckets.Length;//获得hashCode在buckets中存放在位置


        int collisionCount = 0;//冲突数量

        //从hash桶中获取索引     i = entries[i].next 继续获取拉链下一个数据
        for (int i = buckets[targetBucket]; i >= 0; i = entries[i].next)
        {
            if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key))//如果键值对条目数组索引位置hashCode等于key的hashCode 且key相等 赋值
            {
                if (add)
                {
                    throw new Exception("key已存在 异常");
                }
                entries[i].value = value;
                version++;
                return;
            }
            //如果不相等 冲突+1
            collisionCount++;
        }

        //如果entries[i].next=-1 下一个拉链数据为空则执行下面的操作
        //创建一个拉链数据

        int index;
        if (freeCount > 0)//有空闲位置
        {
            index = freeList;
            freeList = entries[index].next;
            freeCount--;
        }
        else
        {
            if (count == entries.Length)//如果键值对条目数组已满
            {
                Resize();//调整大小
                targetBucket = hashCode % buckets.Length;//重现获取hashCode在buckets中存放在位置
            }
            index = count;//取键值对条目数组空闲的位置
            count++;
        }

        entries[index].hashCode = hashCode;
        entries[index].next = buckets[targetBucket];//把原hash桶中索引的键值对条目索引赋值到当前键值对条目的下一个位
        entries[index].key = key;
        entries[index].value = value;
        buckets[targetBucket] = index;//替换hash桶位的指向
        version++;

        // 如果碰撞次数大于设置的最大碰撞次数，那么触发Hash碰撞扩容
        if (collisionCount > 100 && (comparer == null || comparer == EqualityComparer<TKey>.Default))
        {
            comparer = EqualityComparer<TKey>.Default;//使用新的对比hash函数
            Resize(entries.Length, true);
        }


    }

    /// <summary>
    /// 查找键值对条目
    /// </summary>
    /// <param name="key"></param>
    /// <returns></returns>
    private int FindEntry(TKey key)
    {
        if (key == null)
        {
            throw new Exception("key 异常");
        }

        if (buckets != null)
        {
            int hashCode = comparer.GetHashCode(key) & 0x7FFFFFFF;
            for (int i = buckets[hashCode % buckets.Length]; i >= 0; i = entries[i].next)
            {
                //判断hashCode相等且key相等 则返回索引   如果不想等 获取next
                if (entries[i].hashCode == hashCode && comparer.Equals(entries[i].key, key)) return i;
            }
        }
        return -1;
    }

    /// <summary>
    /// 调整大小
    /// </summary>
    private void Resize()
    {
        Resize(HashHelpers.ExpandPrime(count), false);
    }

    /// <summary>
    /// 调整大小
    /// </summary>
    /// <param name="newSize">新da'xia</param>
    /// <param name="forceNewHashCodes">强制使用新的hashcode方法</param>
    private void Resize(int newSize, bool forceNewHashCodes)
    {
        if (newSize >= entries.Length == false) throw new Exception("newSize 异常");

        int[] newBuckets = new int[newSize];//新的hash桶
        for (int i = 0; i < newBuckets.Length; i++) newBuckets[i] = -1;

        Entry[] newEntries = new Entry[newSize];//新的键值对条目数组

        Array.Copy(entries, 0, newEntries, 0, count);//拷贝老的键值对条目数组到新的键值对条目数组

        if (forceNewHashCodes)//强制使用新的hashcode
        {
            for (int i = 0; i < count; i++)
            {
                if (newEntries[i].hashCode != -1)//过滤未使用的
                {
                    newEntries[i].hashCode = (comparer.GetHashCode(newEntries[i].key) & 0x7FFFFFFF);//使用新HashCode函数重新计算Hash值
                }
            }
        }
        for (int i = 0; i < count; i++)
        {
            int bucket = newEntries[i].hashCode % newSize;//获取hash桶索引
            newEntries[i].next = newBuckets[bucket];//第一次 newBuckets[bucket]==-1 第n次 newBuckets[bucket]==上次赋值索引
            newBuckets[bucket] = i;//赋值到hash桶
        }
        buckets = newBuckets;
        entries = newEntries;
    }

    private void CopyTo(KeyValuePair<TKey, TValue>[] array, int index)
    {
        if (array == null)
        {
            throw new Exception("异常");
        }

        if (index < 0 || index > array.Length)
        {
            throw new Exception("异常");
        }

        if (array.Length - index < Count)
        {
            throw new Exception("异常");
        }

        int count = this.count;
        Entry[] entries = this.entries;
        for (int i = 0; i < count; i++)
        {
            if (entries[i].hashCode >= 0)
            {
                array[index++] = new KeyValuePair<TKey, TValue>(entries[i].key, entries[i].value);
            }
        }
    }

    public struct Enumerator : IEnumerator<KeyValuePair<TKey, TValue>>,
            IDictionaryEnumerator
    {
        private MyDictionary<TKey, TValue> dictionary;
        private int version;
        private int index;
        private KeyValuePair<TKey, TValue> current;
        private int getEnumeratorRetType;  // What should Enumerator.Current return?

        internal const int DictEntry = 1;
        internal const int KeyValuePair = 2;

        internal Enumerator(MyDictionary<TKey, TValue> dictionary, int getEnumeratorRetType)
        {
            this.dictionary = dictionary;
            version = dictionary.version;
            index = 0;
            this.getEnumeratorRetType = getEnumeratorRetType;
            current = new KeyValuePair<TKey, TValue>();
        }

        public bool MoveNext()
        {
            if (version != dictionary.version)
            {
                throw new Exception("异常");
            }

            // Use unsigned comparison since we set index to dictionary.count+1 when the enumeration ends.
            // dictionary.count+1 could be negative if dictionary.count is Int32.MaxValue
            while ((uint)index < (uint)dictionary.count)
            {
                if (dictionary.entries[index].hashCode >= 0)
                {
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

        public KeyValuePair<TKey, TValue> Current
        {
            get { return current; }
        }

        public void Dispose()
        {
        }

        object IEnumerator.Current
        {
            get
            {
                if (index == 0 || (index == dictionary.count + 1))
                {
                    throw new Exception("异常");
                }

                if (getEnumeratorRetType == DictEntry)
                {
                    return new System.Collections.DictionaryEntry(current.Key, current.Value);
                }
                else
                {
                    return new KeyValuePair<TKey, TValue>(current.Key, current.Value);
                }
            }
        }

        void IEnumerator.Reset()
        {
            if (version != dictionary.version)
            {
                throw new Exception("异常");
            }

            index = 0;
            current = new KeyValuePair<TKey, TValue>();
        }

        DictionaryEntry IDictionaryEnumerator.Entry
        {
            get
            {
                if (index == 0 || (index == dictionary.count + 1))
                {
                    throw new Exception("异常");
                }

                return new DictionaryEntry(current.Key, current.Value);
            }
        }

        object IDictionaryEnumerator.Key
        {
            get
            {
                if (index == 0 || (index == dictionary.count + 1))
                {
                    throw new Exception("异常");
                }

                return current.Key;
            }
        }

        object IDictionaryEnumerator.Value
        {
            get
            {
                if (index == 0 || (index == dictionary.count + 1))
                {
                    throw new Exception("异常");
                }

                return current.Value;
            }
        }
    }

    public sealed class KeyCollection : ICollection<TKey>, ICollection
    {
        private MyDictionary<TKey, TValue> dictionary;

        public KeyCollection(MyDictionary<TKey, TValue> dictionary)
        {
            if (dictionary == null)
            {
                throw new Exception("dictionary == null 异常");
            }
            this.dictionary = dictionary;
        }

        public Enumerator GetEnumerator()
        {
            return new Enumerator(dictionary);
        }

        public void CopyTo(TKey[] array, int index)
        {
            if (array == null)
            {
                throw new Exception("array == null 异常");
            }

            if (index < 0 || index > array.Length)
            {
                throw new Exception("index < 0 || index > array.Length 异常");
            }

            if (array.Length - index < dictionary.Count)
            {
                throw new Exception("异常");
            }

            int count = dictionary.count;
            Entry[] entries = dictionary.entries;
            for (int i = 0; i < count; i++)
            {
                if (entries[i].hashCode >= 0) array[index++] = entries[i].key;
            }
        }

        public int Count
        {
            get { return dictionary.Count; }
        }

        bool ICollection<TKey>.IsReadOnly
        {
            get { return true; }
        }

        void ICollection<TKey>.Add(TKey item)
        {
            throw new Exception("异常");
        }

        void ICollection<TKey>.Clear()
        {
            throw new Exception("异常");
        }

        bool ICollection<TKey>.Contains(TKey item)
        {
            return dictionary.ContainsKey(item);
        }

        bool ICollection<TKey>.Remove(TKey item)
        {
            throw new Exception("异常");
        }

        IEnumerator<TKey> IEnumerable<TKey>.GetEnumerator()
        {
            return new Enumerator(dictionary);
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return new Enumerator(dictionary);
        }

        void ICollection.CopyTo(Array array, int index)
        {
            if (array == null)
            {
                throw new Exception("异常");
            }

            if (array.Rank != 1)
            {
                throw new Exception("异常");
            }

            if (array.GetLowerBound(0) != 0)
            {
                throw new Exception("异常");
            }

            if (index < 0 || index > array.Length)
            {
                throw new Exception("异常");
            }

            if (array.Length - index < dictionary.Count)
            {
                throw new Exception("异常");
            }

            TKey[] keys = array as TKey[];
            if (keys != null)
            {
                CopyTo(keys, index);
            }
            else
            {
                object[] objects = array as object[];
                if (objects == null)
                {
                    throw new Exception("异常");
                }

                int count = dictionary.count;
                Entry[] entries = dictionary.entries;
                try
                {
                    for (int i = 0; i < count; i++)
                    {
                        if (entries[i].hashCode >= 0) objects[index++] = entries[i].key;
                    }
                }
                catch (ArrayTypeMismatchException)
                {
                    throw new Exception("异常");
                }
            }
        }

        bool ICollection.IsSynchronized
        {
            get { return false; }
        }

        Object ICollection.SyncRoot
        {
            get { return ((ICollection)dictionary).SyncRoot; }
        }

        [Serializable]
        public struct Enumerator : IEnumerator<TKey>, System.Collections.IEnumerator
        {
            private MyDictionary<TKey, TValue> dictionary;
            private int index;
            private int version;
            private TKey currentKey;

            internal Enumerator(MyDictionary<TKey, TValue> dictionary)
            {
                this.dictionary = dictionary;
                version = dictionary.version;
                index = 0;
                currentKey = default(TKey);
            }

            public void Dispose()
            {
            }

            public bool MoveNext()
            {
                if (version != dictionary.version)
                {
                    throw new Exception("异常");
                }

                while ((uint)index < (uint)dictionary.count)
                {
                    if (dictionary.entries[index].hashCode >= 0)
                    {
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

            public TKey Current
            {
                get
                {
                    return currentKey;
                }
            }

            Object System.Collections.IEnumerator.Current
            {
                get
                {
                    if (index == 0 || (index == dictionary.count + 1))
                    {
                        throw new Exception("异常");
                    }

                    return currentKey;
                }
            }

            void System.Collections.IEnumerator.Reset()
            {
                if (version != dictionary.version)
                {
                    throw new Exception("异常");
                }

                index = 0;
                currentKey = default(TKey);
            }
        }
    }

    public sealed class ValueCollection : ICollection<TValue>, ICollection
    {
        private MyDictionary<TKey, TValue> dictionary;

        public ValueCollection(MyDictionary<TKey, TValue> dictionary)
        {
            if (dictionary == null)
            {
                throw new Exception("异常");
            }
            this.dictionary = dictionary;
        }

        public Enumerator GetEnumerator()
        {
            return new Enumerator(dictionary);
        }

        public void CopyTo(TValue[] array, int index)
        {
            if (array == null)
            {
                throw new Exception("异常");
            }

            if (index < 0 || index > array.Length)
            {
                throw new Exception("异常");
            }

            if (array.Length - index < dictionary.Count)
            {
                throw new Exception("异常");
            }

            int count = dictionary.count;
            Entry[] entries = dictionary.entries;
            for (int i = 0; i < count; i++)
            {
                if (entries[i].hashCode >= 0) array[index++] = entries[i].value;
            }
        }

        public int Count
        {
            get { return dictionary.Count; }
        }

        bool ICollection<TValue>.IsReadOnly
        {
            get { return true; }
        }

        void ICollection<TValue>.Add(TValue item)
        {
            throw new Exception("异常");
        }

        bool ICollection<TValue>.Remove(TValue item)
        {
            throw new Exception("异常");
        }

        void ICollection<TValue>.Clear()
        {
            throw new Exception("异常");
        }

        bool ICollection<TValue>.Contains(TValue item)
        {
            return dictionary.ContainsValue(item);
        }

        IEnumerator<TValue> IEnumerable<TValue>.GetEnumerator()
        {
            return new Enumerator(dictionary);
        }

        IEnumerator IEnumerable.GetEnumerator()
        {
            return new Enumerator(dictionary);
        }

        void ICollection.CopyTo(Array array, int index)
        {
            if (array == null)
            {
                throw new Exception("异常");
            }

            if (array.Rank != 1)
            {
                throw new Exception("异常");
            }

            if (array.GetLowerBound(0) != 0)
            {
                throw new Exception("异常");
            }

            if (index < 0 || index > array.Length)
            {
                throw new Exception("异常");
            }

            if (array.Length - index < dictionary.Count)
                throw new Exception("异常");

            TValue[] values = array as TValue[];
            if (values != null)
            {
                CopyTo(values, index);
            }
            else
            {
                object[] objects = array as object[];
                if (objects == null)
                {
                    throw new Exception("异常");
                }

                int count = dictionary.count;
                Entry[] entries = dictionary.entries;
                try
                {
                    for (int i = 0; i < count; i++)
                    {
                        if (entries[i].hashCode >= 0) objects[index++] = entries[i].value;
                    }
                }
                catch (ArrayTypeMismatchException)
                {
                    throw new Exception("异常");
                    throw new Exception("异常");
                }
            }
        }

        bool ICollection.IsSynchronized
        {
            get { return false; }
        }

        Object ICollection.SyncRoot
        {
            get { return ((ICollection)dictionary).SyncRoot; }
        }

        [Serializable]
        public struct Enumerator : IEnumerator<TValue>, System.Collections.IEnumerator
        {
            private MyDictionary<TKey, TValue> dictionary;
            private int index;
            private int version;
            private TValue currentValue;

            internal Enumerator(MyDictionary<TKey, TValue> dictionary)
            {
                this.dictionary = dictionary;
                version = dictionary.version;
                index = 0;
                currentValue = default(TValue);
            }

            public void Dispose()
            {
            }

            public bool MoveNext()
            {
                if (version != dictionary.version)
                {
                    throw new Exception("异常");
                }

                while ((uint)index < (uint)dictionary.count)
                {
                    if (dictionary.entries[index].hashCode >= 0)
                    {
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

            public TValue Current
            {
                get
                {
                    return currentValue;
                }
            }

            Object System.Collections.IEnumerator.Current
            {
                get
                {
                    if (index == 0 || (index == dictionary.count + 1))
                    {
                        throw new Exception("异常");
                    }

                    return currentValue;
                }
            }

            void System.Collections.IEnumerator.Reset()
            {
                if (version != dictionary.version)
                {
                    throw new Exception("异常");
                }
                index = 0;
                currentValue = default(TValue);
            }
        }
    }
}
```

```c#
using System;

public class HashHelpers
{
    public static readonly int[] primes = {
            3, 7, 11, 17, 23, 29, 37, 47, 59, 71, 89, 107, 131, 163, 197, 239, 293, 353, 431, 521, 631, 761, 919,
            1103, 1327, 1597, 1931, 2333, 2801, 3371, 4049, 4861, 5839, 7013, 8419, 10103, 12143, 14591,
            17519, 21023, 25229, 30293, 36353, 43627, 52361, 62851, 75431, 90523, 108631, 130363, 156437,
            187751, 225307, 270371, 324449, 389357, 467237, 560689, 672827, 807403, 968897, 1162687, 1395263,
            1674319, 2009191, 2411033, 2893249, 3471899, 4166287, 4999559, 5999471, 7199369};

    /// <summary>
    /// 获取质数
    /// </summary>
    /// <param name="min">最小值</param>
    /// <returns></returns>
    public static int GetPrime(int min)
    {
        if (min < 0) { return -1; }//异常


        for (int i = 0; i < primes.Length; i++)
        {
            int prime = primes[i];
            if (prime >= min) return prime;
        }

        //outside of our predefined table. 
        //compute the hard way. 
        for (int i = (min | 1); i < Int32.MaxValue; i += 2)
        {
            if (IsPrime(i) && ((i - 1) % 101 != 0))
                return i;
        }
        return min;
    }

    /// <summary>
    /// 判断是否为质数（素数）
    /// </summary>
    /// <param name="candidate"></param>
    /// <returns></returns>
    public static bool IsPrime(int candidate)
    {
        //与0000 0001位运算  可以把奇数偶数 刷选出来，因为偶数不是质数（2除外）
        if ((candidate & 1) != 0)
        {
            int limit = (int)Math.Sqrt(candidate);//求根号 相当于乘法中的中位数
            for (int divisor = 3; divisor <= limit; divisor += 2)//每次+2是跳过 偶数
            {
                if ((candidate % divisor) == 0)
                    return false;
            }
            return true;
        }
        return (candidate == 2);//最后这个保证2也是质数
    }

    public static int ExpandPrime(int oldSize)
    {
        int newSize = 2 * oldSize;

        // Allow the hashtables to grow to maximum possible size (~2G elements) before encoutering capacity overflow.
        // Note that this check works even when _items.Length overflowed thanks to the (uint) cast
        if ((uint)newSize > 0x7FEFFFFD && 0x7FEFFFFD > oldSize)
        {
            throw new Exception("oldSize 异常");
        }

        return GetPrime(newSize);
    }
}
```

