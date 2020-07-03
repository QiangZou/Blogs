# C# HashSet源码分享 自定义HashSet

## 官网源码地址：

[https://referencesource.microsoft.com/#System.Core/System/Collections/Generic/HashSet.cs](https://referencesource.microsoft.com/#System.Core/System/Collections/Generic/HashSet.cs)

## 关键点

- 实现原理和Dictionary差不多 

  - Dictionary 有Key Value
  - HashSet只有Value

- 实际容器为Slot[] m_slots;

  - ```c#
    internal struct Slot 
    {
      internal int hashCode;      // Lower 31 bits of hash code, -1 if unused
      internal int next;          // Index of next entry, -1 if last
      internal T value;
    }
    ```

- HashSet操作元素的时间复杂度接近`O(1)`

  - 定义int[] m_buckets 数组来保存元素在实际容器Slot[] m_slots 位置
  - 即 Value的保存在 m_slots[m_buckets[value.GetHashCode()%m_buckets.Length]].value

- 容器长度为质数

  - 质数只能被1和自身整除
  - 减少位置冲突

- 当位置冲突时使用Slot.next保存数据

- 数据已满时添加数据扩容会自动扩充当前容量的2倍 

  - 新建一个2倍大小的容器
  - 数据拷贝过去 重新计算位置

## 优化点

- 已知容器大小的情况 直接初始化对应大小
- 自定义元素可以实现IEqualityComparer可以更高效判断相等和获取HashCode

## 自定义HashSet

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.Diagnostics;
using System.Runtime.Serialization;

public class MyHashSet<T> : IEnumerable<T>
{
    internal struct Slot
    {
        /// <summary>
        /// 如果未使用则为-1
        /// </summary>
        internal int hashCode;
        internal T value;
        /// <summary>
        /// hashCode冲突是指向下一索引 未使用为-1
        /// </summary>
        internal int next;
    }

    internal struct ElementCount
    {
        internal int uniqueCount;
        internal int unfoundCount;
    }

    private const int Lower31BitMask = 0x7FFFFFFF;

    /// <summary>
    /// 堆栈阈值
    /// </summary>
    private const int StackAllocThreshold = 100;


    /// <summary>
    /// Hash桶
    /// </summary>
    private int[] m_buckets;
    /// <summary>
    /// 数据
    /// </summary>
    private Slot[] m_slots;
    /// <summary>
    /// 实际使用大小
    /// </summary>
    private int m_count;
    /// <summary>
    /// 上次使用索引位置
    /// </summary>
    private int m_lastIndex;
    /// <summary>
    /// 空闲索引
    /// </summary>
    private int m_freeList;
    /// <summary>
    /// 相等对比函数
    /// </summary>
    private IEqualityComparer<T> m_comparer;
    /// <summary>
    /// 版本
    /// </summary>
    private int m_version;


    /// <summary>
    /// 初始化 使用默认相等对比函数
    /// </summary>
    public MyHashSet() : this(EqualityComparer<T>.Default) { }

    /// <summary>
    /// 初始化
    /// </summary>
    /// <param name="comparer">相等对比函数</param>
    public MyHashSet(IEqualityComparer<T> comparer)
    {
        if (comparer == null)
        {
            comparer = EqualityComparer<T>.Default;
        }

        this.m_comparer = comparer;
        m_lastIndex = 0;
        m_count = 0;
        m_freeList = -1;
        m_version = 0;
    }

    /// <summary>
    /// 初始化 
    /// </summary>
    /// <param name="collection">指定容器</param>
    public MyHashSet(IEnumerable<T> collection) : this(collection, EqualityComparer<T>.Default) { }

    /// <summary>
    /// 初始化
    /// </summary>
    /// <param name="collection">指定容器</param>
    /// <param name="comparer">指定相等对比函数</param>
    public MyHashSet(IEnumerable<T> collection, IEqualityComparer<T> comparer) : this(comparer)
    {
        if (collection == null)
        {
            throw new ArgumentNullException("collection");
        }

        int suggestedCapacity = 0;
        ICollection<T> coll = collection as ICollection<T>;
        if (coll != null)
        {
            suggestedCapacity = coll.Count;
        }
        Initialize(suggestedCapacity);

        this.UnionWith(collection);//合并数据
        if ((m_count == 0 && m_slots.Length > 3) || (m_count > 0 && m_slots.Length / m_count > 3))
        {
            TrimExcess();//设置容量为实际元素数
        }
    }

    /// <summary>
    /// 数量
    /// </summary>
    public int Count
    {
        get { return m_count; }
    }



    /// <summary>
    /// 添加
    /// </summary>
    /// <param name="item"></param>
    /// <returns></returns>
    public bool Add(T item)
    {
        return AddIfNotPresent(item);
    }

    /// <summary>
    /// 清理
    /// </summary>
    public void Clear()
    {
        if (m_lastIndex > 0)
        {
            Debug.Assert(m_buckets != null, "m_buckets was null but m_lastIndex > 0");

            Array.Clear(m_slots, 0, m_lastIndex);
            Array.Clear(m_buckets, 0, m_buckets.Length);
            m_lastIndex = 0;
            m_count = 0;
            m_freeList = -1;
        }
        m_version++;
    }

    /// <summary>
    /// 是否包含
    /// </summary>
    /// <param name="item"></param>
    /// <returns></returns>
    public bool Contains(T item)
    {
        if (m_buckets != null)
        {
            int hashCode = InternalGetHashCode(item);
            // see note at "HashSet" level describing why "- 1" appears in for loop
            for (int i = m_buckets[hashCode % m_buckets.Length] - 1; i >= 0; i = m_slots[i].next)
            {
                if (m_slots[i].hashCode == hashCode && m_comparer.Equals(m_slots[i].value, item))
                {
                    return true;
                }
            }
        }
        // either m_buckets is null or wasn't found
        return false;
    }

    /// <summary>
    /// 拷贝到数组
    /// </summary>
    /// <param name="array">拷贝到数组容器</param>
    public void CopyTo(T[] array)
    {
        CopyTo(array, 0, m_count);
    }

    /// <summary>
    /// 拷贝到数组
    /// </summary>
    /// <param name="array">拷贝到数组容器</param>
    /// <param name="arrayIndex">数组容器开始项</param>
    public void CopyTo(T[] array, int arrayIndex)
    {
        CopyTo(array, arrayIndex, m_count);
    }

    /// <summary>
    /// 拷贝到数组
    /// </summary>
    /// <param name="array">拷贝到数组容器</param>
    /// <param name="arrayIndex">数组容器开始项</param>
    /// <param name="count">拷贝数量</param>
    public void CopyTo(T[] array, int arrayIndex, int count)
    {
        if (array == null)
        {
            throw new ArgumentNullException("array");
        }

        if (arrayIndex < 0)
        {
            throw new ArgumentOutOfRangeException("arrayIndex", "arrayIndex < 0");
        }

        if (count < 0)
        {
            throw new ArgumentOutOfRangeException("count", "count < 0");
        }

        if (arrayIndex > array.Length || count > array.Length - arrayIndex)
        {
            throw new ArgumentException("arrayIndex > array.Length || count > array.Length - arrayIndex");
        }

        int numCopied = 0;
        for (int i = 0; i < m_lastIndex && numCopied < count; i++)
        {
            if (m_slots[i].hashCode >= 0)
            {
                array[arrayIndex + numCopied] = m_slots[i].value;
                numCopied++;
            }
        }
    }

    /// <summary>
    /// 删除和other相等的项
    /// </summary>
    /// <param name="other"></param>
    public void ExceptWith(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        // this is already the enpty set; return
        if (m_count == 0)
        {
            return;
        }

        // special case if other is this; a set minus itself is the empty set
        if (other == this)
        {
            Clear();
            return;
        }

        // remove every element in other from this
        foreach (T element in other)
        {
            Remove(element);
        }
    }

    /// <summary>
    /// 获取迭代枚举数
    /// </summary>
    /// <returns></returns>
    public Enumerator GetEnumerator()
    {
        return new Enumerator(this);
    }

    IEnumerator<T> IEnumerable<T>.GetEnumerator()
    {
        return new Enumerator(this);
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return new Enumerator(this);
    }

    public IEqualityComparer<T> Comparer
    {
        get
        {
            return m_comparer;
        }
    }

    public virtual void GetObjectData(SerializationInfo info, StreamingContext context)
    {

    }

    /// <summary>
    /// 求和other交集
    /// </summary>
    /// <param name="other"></param>
    public void IntersectWith(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        if (m_count == 0)
        {
            return;
        }

        ICollection<T> otherAsCollection = other as ICollection<T>;
        if (otherAsCollection != null)
        {
            if (otherAsCollection.Count == 0)
            {
                Clear();
                return;
            }

            MyHashSet<T> otherAsSet = other as MyHashSet<T>;
            // faster if other is a hashset using same equality comparer; so check 
            // that other is a hashset using the same equality comparer.
            if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
            {
                IntersectWithHashSetWithSameEC(otherAsSet);
                return;
            }
        }

        IntersectWithEnumerable(other);
    }

    /// <summary>
    /// 是否为other的真子集
    /// </summary>
    /// <param name="other"></param>
    /// <returns></returns>
    public bool IsProperSubsetOf(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        ICollection<T> otherAsCollection = other as ICollection<T>;
        if (otherAsCollection != null)
        {
            // the empty set is a proper subset of anything but the empty set
            if (m_count == 0)
            {
                return otherAsCollection.Count > 0;
            }
            MyHashSet<T> otherAsSet = other as MyHashSet<T>;
            // faster if other is a hashset (and we're using same equality comparer)
            if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
            {
                if (m_count >= otherAsSet.Count)
                {
                    return false;
                }
                // this has strictly less than number of items in other, so the following
                // check suffices for proper subset.
                return IsSubsetOfHashSetWithSameEC(otherAsSet);
            }
        }

        ElementCount result = CheckUniqueAndUnfoundElements(other, false);
        return (result.uniqueCount == m_count && result.unfoundCount > 0);

    }

    /// <summary>
    /// 是否为other的真超集
    /// </summary>
    /// <param name="other"></param>
    /// <returns></returns>
    public bool IsProperSupersetOf(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        // the empty set isn't a proper superset of any set.
        if (m_count == 0)
        {
            return false;
        }

        ICollection<T> otherAsCollection = other as ICollection<T>;
        if (otherAsCollection != null)
        {
            // if other is the empty set then this is a superset
            if (otherAsCollection.Count == 0)
            {
                // note that this has at least one element, based on above check
                return true;
            }
            MyHashSet<T> otherAsSet = other as MyHashSet<T>;
            // faster if other is a hashset with the same equality comparer
            if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
            {
                if (otherAsSet.Count >= m_count)
                {
                    return false;
                }
                // now perform element check
                return ContainsAllElements(otherAsSet);
            }
        }
        // couldn't fall out in the above cases; do it the long way
        ElementCount result = CheckUniqueAndUnfoundElements(other, true);
        return (result.uniqueCount < m_count && result.unfoundCount == 0);

    }

    /// <summary>
    /// 是否为other的子集
    /// </summary>
    /// <param name="other"></param>
    /// <returns></returns>
    public bool IsSubsetOf(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        // The empty set is a subset of any set
        if (m_count == 0)
        {
            return true;
        }

        MyHashSet<T> otherAsSet = other as MyHashSet<T>;
        // faster if other has unique elements according to this equality comparer; so check 
        // that other is a hashset using the same equality comparer.
        if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
        {
            // if this has more elements then it can't be a subset
            if (m_count > otherAsSet.Count)
            {
                return false;
            }

            // already checked that we're using same equality comparer. simply check that 
            // each element in this is contained in other.
            return IsSubsetOfHashSetWithSameEC(otherAsSet);
        }
        else
        {
            ElementCount result = CheckUniqueAndUnfoundElements(other, false);
            return (result.uniqueCount == m_count && result.unfoundCount >= 0);
        }
    }

    /// <summary>
    /// 是否为other的超集
    /// </summary>
    /// <param name="other"></param>
    /// <returns></returns>
    public bool IsSupersetOf(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        // try to fall out early based on counts
        ICollection<T> otherAsCollection = other as ICollection<T>;
        if (otherAsCollection != null)
        {
            // if other is the empty set then this is a superset
            if (otherAsCollection.Count == 0)
            {
                return true;
            }
            MyHashSet<T> otherAsSet = other as MyHashSet<T>;
            // try to compare based on counts alone if other is a hashset with
            // same equality comparer
            if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
            {
                if (otherAsSet.Count > m_count)
                {
                    return false;
                }
            }
        }

        return ContainsAllElements(other);
    }

    public virtual void OnDeserialization(Object sender)
    {

    }

    /// <summary>
    /// 是否包含other中至少一个元素
    /// </summary>
    /// <param name="other"></param>
    /// <returns></returns>
    public bool Overlaps(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        if (m_count == 0)
        {
            return false;
        }

        foreach (T element in other)
        {
            if (Contains(element))
            {
                return true;
            }
        }
        return false;
    }

    /// <summary>
    /// 删除元素
    /// </summary>
    /// <param name="item"></param>
    /// <returns></returns>
    public bool Remove(T item)
    {
        if (m_buckets != null)
        {
            int hashCode = InternalGetHashCode(item);
            int bucket = hashCode % m_buckets.Length;
            int last = -1;
            for (int i = m_buckets[bucket] - 1; i >= 0; last = i, i = m_slots[i].next)
            {
                if (m_slots[i].hashCode == hashCode && m_comparer.Equals(m_slots[i].value, item))
                {
                    if (last < 0)
                    {
                        // first iteration; update buckets
                        //如果第一次遍历就找到 m_buckets Hash桶索引指向当前的next next为-1则为0
                        m_buckets[bucket] = m_slots[i].next + 1;
                    }
                    else
                    {
                        // subsequent iterations; update 'next' pointers
                        //第N次遍历到  则上一个数据容器 next执行当前要删除的next
                        m_slots[last].next = m_slots[i].next;
                    }
                    m_slots[i].hashCode = -1;
                    m_slots[i].value = default(T);
                    m_slots[i].next = m_freeList;

                    m_count--;
                    m_version++;
                    if (m_count == 0)
                    {
                        m_lastIndex = 0;
                        m_freeList = -1;
                    }
                    else
                    {
                        m_freeList = i;
                    }
                    return true;
                }
            }
        }
        // either m_buckets is null or wasn't found
        return false;
    }

    /// <summary>
    /// 删除匹配项 返回匹配数量
    /// </summary>
    /// <param name="match"></param>
    /// <returns></returns>
    public int RemoveWhere(Predicate<T> match)
    {
        if (match == null)
        {
            throw new ArgumentNullException("match");
        }

        int numRemoved = 0;
        for (int i = 0; i < m_lastIndex; i++)
        {
            if (m_slots[i].hashCode >= 0)
            {
                // cache value in case delegate removes it
                T value = m_slots[i].value;
                if (match(value))
                {
                    // check again that remove actually removed it
                    if (Remove(value))
                    {
                        numRemoved++;
                    }
                }
            }
        }
        return numRemoved;
    }

    /// <summary>
    /// 判断other所有元素与自身相等
    /// </summary>
    /// <param name="other"></param>
    /// <returns></returns>
    public bool SetEquals(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        MyHashSet<T> otherAsSet = other as MyHashSet<T>;
        // faster if other is a hashset and we're using same equality comparer
        if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
        {
            // attempt to return early: since both contain unique elements, if they have 
            // different counts, then they can't be equal
            if (m_count != otherAsSet.Count)
            {
                return false;
            }

            // already confirmed that the sets have the same number of distinct elements, so if
            // one is a superset of the other then they must be equal
            return ContainsAllElements(otherAsSet);
        }
        else
        {
            ICollection<T> otherAsCollection = other as ICollection<T>;
            if (otherAsCollection != null)
            {
                // if this count is 0 but other contains at least one element, they can't be equal
                if (m_count == 0 && otherAsCollection.Count > 0)
                {
                    return false;
                }
            }
            ElementCount result = CheckUniqueAndUnfoundElements(other, true);
            return (result.uniqueCount == m_count && result.unfoundCount == 0);
        }
    }

    /// <summary>
    /// 修改自身 删除存在自身和other的元素
    /// </summary>
    /// <param name="other"></param>
    public void SymmetricExceptWith(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        // if set is empty, then symmetric difference is other
        if (m_count == 0)
        {
            UnionWith(other);
            return;
        }

        // special case this; the symmetric difference of a set with itself is the empty set
        if (other == this)
        {
            Clear();
            return;
        }

        MyHashSet<T> otherAsSet = other as MyHashSet<T>;
        // If other is a HashSet, it has unique elements according to its equality comparer,
        // but if they're using different equality comparers, then assumption of uniqueness
        // will fail. So first check if other is a hashset using the same equality comparer;
        // symmetric except is a lot faster and avoids bit array allocations if we can assume
        // uniqueness
        if (otherAsSet != null && AreEqualityComparersEqual(this, otherAsSet))
        {
            SymmetricExceptWithUniqueHashSet(otherAsSet);
        }
        else
        {
            SymmetricExceptWithEnumerable(other);
        }
    }

    /// <summary>
    /// 除去多余的容器
    /// </summary>
    public void TrimExcess()
    {
        Debug.Assert(m_count >= 0, "m_count is negative");

        if (m_count == 0)//如果计数为零，清除引用
        {
            m_buckets = null;
            m_slots = null;
            m_version++;
        }
        else
        {
            Debug.Assert(m_buckets != null, "m_buckets was null but m_count > 0");

            int newSize = HashHelpers.GetPrime(m_count);//获取最小适用素数
            Slot[] newSlots = new Slot[newSize];
            int[] newBuckets = new int[newSize];

            int newIndex = 0;
            for (int i = 0; i < m_lastIndex; i++)//遍历容器已使用项
            {
                if (m_slots[i].hashCode >= 0)
                {
                    newSlots[newIndex] = m_slots[i];//拷贝到新容器

                    int bucket = newSlots[newIndex].hashCode % newSize;//重现获取hash桶索引
                    newSlots[newIndex].next = newBuckets[bucket] - 1;//设置新容器的next
                    newBuckets[bucket] = newIndex + 1;//设置新hash桶索引

                    newIndex++;
                }
            }

            Debug.Assert(newSlots.Length <= m_slots.Length, "capacity increased after TrimExcess");

            m_lastIndex = newIndex;
            m_slots = newSlots;
            m_buckets = newBuckets;
            m_freeList = -1;
        }
    }

    /// <summary>
    /// 合并
    /// </summary>
    /// <param name="other"></param>
    public void UnionWith(IEnumerable<T> other)
    {
        if (other == null)
        {
            throw new ArgumentNullException("other");
        }

        foreach (T item in other)
        {
            AddIfNotPresent(item);//添加项
        }
    }




    /// <summary>
    /// 初始化容器
    /// </summary>
    /// <param name="capacity"></param>
    private void Initialize(int capacity)
    {
        Debug.Assert(m_buckets == null, "Initialize was called but m_buckets was non-null");

        int size = HashHelpers.GetPrime(capacity);//去最小质数容器

        m_buckets = new int[size];//初始化指定大小Hash桶
        m_slots = new Slot[size];
    }

    /// <summary>
    /// 添加如果值不存在
    /// </summary>
    /// <param name="value"></param>
    /// <returns></returns>
    private bool AddIfNotPresent(T value)
    {
        if (m_buckets == null)
        {
            Initialize(0);
        }

        int hashCode = InternalGetHashCode(value);//获取整数Hash code 例如：978656418
        int bucket = hashCode % m_buckets.Length;//求与 获取存放位置 例如长度为8 求余为2 

        int collisionCount = 0;//冲突数量

        for (int i = m_buckets[hashCode % m_buckets.Length] - 1; i >= 0; i = m_slots[i].next)
        {
            //遍历第一次  获取数据容器m_slots[2]值 
            //判断当前位置已存有相同的值
            //遍历第二次  取m_slots[i].next 是否大于-1
            //判断当前位置是否冲突  大于等于0 即代表冲突
            if (m_slots[i].hashCode == hashCode && m_comparer.Equals(m_slots[i].value, value))
            {
                return false;//容器中已经有相等的值  添加失败
            }

            collisionCount++;//冲突增加
        }

        int index;
        if (m_freeList >= 0)//有空闲位置
        {
            index = m_freeList;
            m_freeList = m_slots[index].next;
        }
        else
        {
            if (m_lastIndex == m_slots.Length)
            {
                IncreaseCapacity();//调整大小

                bucket = hashCode % m_buckets.Length;////重现获取hashCode在buckets中存放在位置
            }
            index = m_lastIndex;
            m_lastIndex++;
        }
        //存数据
        m_slots[index].hashCode = hashCode;
        m_slots[index].value = value;
        m_slots[index].next = m_buckets[bucket] - 1;

        //存索引 存的时候+1  取得时候要-1
        m_buckets[bucket] = index + 1;


        m_count++;
        m_version++;


        if (collisionCount > 100 && (m_comparer == null || m_comparer == EqualityComparer<T>.Default))
        {
            m_comparer = EqualityComparer<T>.Default;//使用新的对比hash函数
            SetCapacity(m_buckets.Length, true);
        }

        return true;
    }

    /// <summary>
    /// 判断是相等对比函数是否相等
    /// </summary>
    /// <param name="set1"></param>
    /// <param name="set2"></param>
    /// <returns></returns>
    private static bool AreEqualityComparersEqual(MyHashSet<T> set1, MyHashSet<T> set2)
    {
        return set1.Comparer.Equals(set2.Comparer);
    }

    /// <summary>
    /// 内部获取Hash Code
    /// </summary>
    /// <param name="item"></param>
    /// <returns></returns>
    private int InternalGetHashCode(T item)
    {
        if (item == null)
        {
            return 0;
        }
        return m_comparer.GetHashCode(item) & Lower31BitMask;//忽略符号位
    }

    /// <summary>
    /// 夸大容量
    /// </summary>
    private void IncreaseCapacity()
    {
        Debug.Assert(m_buckets != null, "IncreaseCapacity called on a set with no elements");

        int newSize = HashHelpers.ExpandPrime(m_count);//扩大两倍
        if (newSize <= m_count)
        {
            throw new ArgumentException("newSize <= m_count");
        }

        // Able to increase capacity; copy elements to larger array and rehash
        SetCapacity(newSize, false);
    }

    /// <summary>
    /// 设置容量大小
    /// </summary>
    /// <param name="newSize"></param>
    /// <param name="forceNewHashCodes"></param>
    private void SetCapacity(int newSize, bool forceNewHashCodes)
    {
        if (!HashHelpers.IsPrime(newSize))
        {
            throw new ArgumentException("New size is not prime!");
        }
        if (m_buckets == null)
        {
            throw new ArgumentException("SetCapacity called on a set with no elements");
        }

        Slot[] newSlots = new Slot[newSize];
        if (m_slots != null)
        {
            Array.Copy(m_slots, 0, newSlots, 0, m_lastIndex);
        }

        if (forceNewHashCodes)
        {
            for (int i = 0; i < m_lastIndex; i++)
            {
                if (newSlots[i].hashCode != -1)
                {
                    newSlots[i].hashCode = InternalGetHashCode(newSlots[i].value);
                }
            }
        }

        int[] newBuckets = new int[newSize];
        for (int i = 0; i < m_lastIndex; i++)
        {
            int bucket = newSlots[i].hashCode % newSize;
            newSlots[i].next = newBuckets[bucket] - 1;
            newBuckets[bucket] = i + 1;
        }
        m_slots = newSlots;
        m_buckets = newBuckets;
    }

    /// <summary>
    /// 删除不相交
    /// </summary>
    /// <param name="other"></param>
    private void IntersectWithHashSetWithSameEC(MyHashSet<T> other)
    {
        for (int i = 0; i < m_lastIndex; i++)
        {
            if (m_slots[i].hashCode >= 0)
            {
                T item = m_slots[i].value;
                if (!other.Contains(item))
                {
                    Remove(item);
                }
            }
        }
    }

    private unsafe void IntersectWithEnumerable(IEnumerable<T> other)
    {
        Debug.Assert(m_buckets != null, "m_buckets shouldn't be null; callers should check first");

        // keep track of current last index; don't want to move past the end of our bit array
        // (could happen if another thread is modifying the collection)
        int originalLastIndex = m_lastIndex;
        int intArrayLength = BitHelper.ToIntArrayLength(originalLastIndex);

        BitHelper bitHelper;
        if (intArrayLength <= StackAllocThreshold)
        {
            int* bitArrayPtr = stackalloc int[intArrayLength];
            bitHelper = new BitHelper(bitArrayPtr, intArrayLength);
        }
        else
        {
            int[] bitArray = new int[intArrayLength];
            bitHelper = new BitHelper(bitArray, intArrayLength);
        }

        // mark if contains: find index of in slots array and mark corresponding element in bit array
        foreach (T item in other)
        {
            int index = InternalIndexOf(item);
            if (index >= 0)
            {
                bitHelper.MarkBit(index);
            }
        }

        // if anything unmarked, remove it. Perf can be optimized here if BitHelper had a 
        // FindFirstUnmarked method.
        for (int i = 0; i < originalLastIndex; i++)
        {
            if (m_slots[i].hashCode >= 0 && !bitHelper.IsMarked(i))
            {
                Remove(m_slots[i].value);
            }
        }
    }

    /// <summary>
    /// 内部查找索引
    /// </summary>
    /// <param name="item"></param>
    /// <returns></returns>
    private int InternalIndexOf(T item)
    {
        Debug.Assert(m_buckets != null, "m_buckets was null; callers should check first");

        int hashCode = InternalGetHashCode(item);
        for (int i = m_buckets[hashCode % m_buckets.Length] - 1; i >= 0; i = m_slots[i].next)
        {
            if ((m_slots[i].hashCode) == hashCode && m_comparer.Equals(m_slots[i].value, item))
            {
                return i;
            }
        }
        // wasn't found
        return -1;
    }

    private bool IsSubsetOfHashSetWithSameEC(MyHashSet<T> other)
    {

        foreach (T item in this)
        {
            if (!other.Contains(item))
            {
                return false;
            }
        }
        return true;
    }

    private bool ContainsAllElements(IEnumerable<T> other)
    {
        foreach (T element in other)
        {
            if (!Contains(element))
            {
                return false;
            }
        }
        return true;
    }

    private void SymmetricExceptWithUniqueHashSet(MyHashSet<T> other)
    {
        foreach (T item in other)
        {
            if (!Remove(item))
            {
                AddIfNotPresent(item);
            }
        }
    }

    private unsafe void SymmetricExceptWithEnumerable(IEnumerable<T> other)
    {
        int originalLastIndex = m_lastIndex;
        int intArrayLength = BitHelper.ToIntArrayLength(originalLastIndex);

        BitHelper itemsToRemove;
        BitHelper itemsAddedFromOther;
        if (intArrayLength <= StackAllocThreshold / 2)
        {
            int* itemsToRemovePtr = stackalloc int[intArrayLength];
            itemsToRemove = new BitHelper(itemsToRemovePtr, intArrayLength);

            int* itemsAddedFromOtherPtr = stackalloc int[intArrayLength];
            itemsAddedFromOther = new BitHelper(itemsAddedFromOtherPtr, intArrayLength);
        }
        else
        {
            int[] itemsToRemoveArray = new int[intArrayLength];
            itemsToRemove = new BitHelper(itemsToRemoveArray, intArrayLength);

            int[] itemsAddedFromOtherArray = new int[intArrayLength];
            itemsAddedFromOther = new BitHelper(itemsAddedFromOtherArray, intArrayLength);
        }

        foreach (T item in other)
        {
            int location = 0;
            bool added = AddOrGetLocation(item, out location);
            if (added)
            {
                // wasn't already present in collection; flag it as something not to remove
                // *NOTE* if location is out of range, we should ignore. BitHelper will
                // detect that it's out of bounds and not try to mark it. But it's 
                // expected that location could be out of bounds because adding the item
                // will increase m_lastIndex as soon as all the free spots are filled.
                itemsAddedFromOther.MarkBit(location);
            }
            else
            {
                // already there...if not added from other, mark for remove. 
                // *NOTE* Even though BitHelper will check that location is in range, we want 
                // to check here. There's no point in checking items beyond originalLastIndex
                // because they could not have been in the original collection
                if (location < originalLastIndex && !itemsAddedFromOther.IsMarked(location))
                {
                    itemsToRemove.MarkBit(location);
                }
            }
        }

        // if anything marked, remove it
        for (int i = 0; i < originalLastIndex; i++)
        {
            if (itemsToRemove.IsMarked(i))
            {
                Remove(m_slots[i].value);
            }
        }
    }

    private bool AddOrGetLocation(T value, out int location)
    {
        Debug.Assert(m_buckets != null, "m_buckets is null, callers should have checked");

        int hashCode = InternalGetHashCode(value);
        int bucket = hashCode % m_buckets.Length;
        for (int i = m_buckets[hashCode % m_buckets.Length] - 1; i >= 0; i = m_slots[i].next)
        {
            if (m_slots[i].hashCode == hashCode && m_comparer.Equals(m_slots[i].value, value))
            {
                location = i;
                return false; //already present
            }
        }
        int index;
        if (m_freeList >= 0)
        {
            index = m_freeList;
            m_freeList = m_slots[index].next;
        }
        else
        {
            if (m_lastIndex == m_slots.Length)
            {
                IncreaseCapacity();
                // this will change during resize
                bucket = hashCode % m_buckets.Length;
            }
            index = m_lastIndex;
            m_lastIndex++;
        }
        m_slots[index].hashCode = hashCode;
        m_slots[index].value = value;
        m_slots[index].next = m_buckets[bucket] - 1;
        m_buckets[bucket] = index + 1;
        m_count++;
        m_version++;
        location = index;
        return true;
    }

    private unsafe ElementCount CheckUniqueAndUnfoundElements(IEnumerable<T> other, bool returnIfUnfound)
    {
        ElementCount result;

        // need special case in case this has no elements. 
        if (m_count == 0)
        {
            int numElementsInOther = 0;
            foreach (T item in other)
            {
                numElementsInOther++;
                // break right away, all we want to know is whether other has 0 or 1 elements
                break;
            }
            result.uniqueCount = 0;
            result.unfoundCount = numElementsInOther;
            return result;
        }


        Debug.Assert((m_buckets != null) && (m_count > 0), "m_buckets was null but count greater than 0");

        int originalLastIndex = m_lastIndex;
        int intArrayLength = BitHelper.ToIntArrayLength(originalLastIndex);

        BitHelper bitHelper;
        if (intArrayLength <= StackAllocThreshold)
        {
            int* bitArrayPtr = stackalloc int[intArrayLength];
            bitHelper = new BitHelper(bitArrayPtr, intArrayLength);
        }
        else
        {
            int[] bitArray = new int[intArrayLength];
            bitHelper = new BitHelper(bitArray, intArrayLength);
        }

        // count of items in other not found in this
        int unfoundCount = 0;
        // count of unique items in other found in this
        int uniqueFoundCount = 0;

        foreach (T item in other)
        {
            int index = InternalIndexOf(item);
            if (index >= 0)
            {
                if (!bitHelper.IsMarked(index))
                {
                    // item hasn't been seen yet
                    bitHelper.MarkBit(index);
                    uniqueFoundCount++;
                }
            }
            else
            {
                unfoundCount++;
                if (returnIfUnfound)
                {
                    break;
                }
            }
        }

        result.uniqueCount = uniqueFoundCount;
        result.unfoundCount = unfoundCount;
        return result;
    }

    public struct Enumerator : IEnumerator<T>, System.Collections.IEnumerator
    {
        private MyHashSet<T> set;
        private int index;
        private int version;
        private T current;

        internal Enumerator(MyHashSet<T> set)
        {
            this.set = set;
            index = 0;
            version = set.m_version;
            current = default(T);
        }

        public void Dispose()
        {
        }

        public bool MoveNext()
        {
            if (version != set.m_version)
            {
                throw new InvalidOperationException("version != set.m_version");
            }

            while (index < set.m_lastIndex)
            {
                if (set.m_slots[index].hashCode >= 0)
                {
                    current = set.m_slots[index].value;
                    index++;
                    return true;
                }
                index++;
            }
            index = set.m_lastIndex + 1;
            current = default(T);
            return false;
        }

        public T Current
        {
            get
            {
                return current;
            }
        }

        Object System.Collections.IEnumerator.Current
        {
            get
            {
                if (index == 0 || index == set.m_lastIndex + 1)
                {
                    throw new InvalidOperationException("index == 0 || index == set.m_lastIndex + 1");
                }
                return Current;
            }
        }

        void System.Collections.IEnumerator.Reset()
        {
            if (version != set.m_version)
            {
                throw new InvalidOperationException("version != set.m_version");
            }

            index = 0;
            current = default(T);
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

