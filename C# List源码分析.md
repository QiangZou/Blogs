# C# List源码分析

## 官网源码地址

[https://referencesource.microsoft.com/#mscorlib/system/collections/generic/list.cs](https://referencesource.microsoft.com/#mscorlib/system/collections/generic/list.cs)

## 关键点

- List实际容器为泛型数组
- Count表示数组的已使用长度
- Capacity表示数组长度
- Capacity>=Count
- List数组容量自动扩充实现方式
  - 当容量大小为0时 初始化一个大小为4的数组
  - 当容量大小非0时 初始化一个大小为原大小2倍的数组 
  - 把原数组数据浅拷贝到新数组中
-  设置Capacity容量大小实现方式（正常不使用）
  - 把原数组拷贝到新Capacity容量大小的数组中

## 优化点

- 已知容器大小的情况 直接初始化对应大小的容器
- 查找项索引的时候可以用BinarySearch查找

## 方法分析

```c#
public List();//初始化了一个大小为零的泛型数组
public List(IEnumerable<T> collection);//初始化了一个大小为collection.Count的泛型数组
public List(int capacity);//初始化了一个大小为capacity的泛型数组

public T this[int index] { get; set; }//获取或设置指定索引的值 异常判断索引是否正确

public int Count { get; }//获取数组实际使用的数量
public int Capacity { get; set; }//获取或设置原数组拷贝到新Capacity容量大小的数组中 异常判断设置大小小于数组使用数量

public void Add(T item);//判断是否自动扩容 赋值到数组中
public void AddRange(IEnumerable<T> collection);//调用InsertRange
public ReadOnlyCollection<T> AsReadOnly();//返回只读数据
public int BinarySearch(T item);
public int BinarySearch(T item, IComparer<T> comparer);
public int BinarySearch(int index, int count, T item, IComparer<T> comparer);//二分查找查找匹配项
public void Clear();//清除数组Array.Clear
public bool Contains(T item);//判断是否包含 算法复杂度O(n) 
public List<TOutput> ConvertAll<TOutput>(Converter<T, TOutput> converter);//转换为其他类型
public void CopyTo(int index, T[] array, int arrayIndex, int count);//拷贝到数组
public void CopyTo(T[] array, int arrayIndex);
public void CopyTo(T[] array);
public bool Exists(Predicate<T> match);//判断是否存匹配函数 调用FindIndex
public T Find(Predicate<T> match);//查找匹配函数项 调用FindIndex
public List<T> FindAll(Predicate<T> match);//查找匹配函数项列表 算法复杂度O(n) 
public int FindIndex(Predicate<T> match);//
public int FindIndex(int startIndex, Predicate<T> match);
public int FindIndex(int startIndex, int count, Predicate<T> match);//从前查找匹配函数项索引
public T FindLast(Predicate<T> match);
public int FindLastIndex(Predicate<T> match);
public int FindLastIndex(int startIndex, Predicate<T> match);
public int FindLastIndex(int startIndex, int count, Predicate<T> match);//从后查找匹配函数项索引
public void ForEach(Action<T> action);//遍历对每项执行action
public Enumerator GetEnumerator();//获取遍历枚举数
public List<T> GetRange(int index, int count);//获取范围中的项
public int IndexOf(T item, int index, int count);//查找项索引 Array.IndexOf
public int IndexOf(T item, int index);
public int IndexOf(T item);
public void Insert(int index, T item);//插入指定索引项 如果自动扩容原数组拷贝到新容量大小的数组中
public void InsertRange(int index, IEnumerable<T> collection);//插入多个项到指定索引
public int LastIndexOf(T item);
public int LastIndexOf(T item, int index);
public int LastIndexOf(T item, int index, int count);//从后查找索引
public bool Remove(T item);//删除项 实际调用IndexOf RemoveAt
public int RemoveAll(Predicate<T> match);//删除所有匹配项
public void RemoveAt(int index);//删除索引项 异常判断索引是否正确
public void RemoveRange(int index, int count);//删除范围内项目 异常判断参数是否正确
public void Reverse(int index, int count);//反转顺序 异常判断参数是否正常
public void Reverse();//Reverse(0, Count);
public void Sort(Comparison<T> comparison);
public void Sort(int index, int count, IComparer<T> comparer);//指定范围、函数排序
public void Sort();//Sort(0, Count, null);
public void Sort(IComparer<T> comparer);//Sort(0, Count, comparer);
public T[] ToArray();//转换为数组 返回一个新数组
public void TrimExcess();//删除多余容量 实际调用Capacity = Count;
public bool TrueForAll(Predicate<T> match);//判断每项是否与函数都匹配
```

## 自定义List

```c#
using System;
using System.Collections;
using System.Collections.Generic;
using System.Collections.ObjectModel;

public interface IMyList<T> : ICollection<T>
{
    T this[int index] { get; set; }
    int IndexOf(T item);
    void Insert(int index, T item);
    void RemoveAt(int index);
}

public class MyList<T> : IEnumerable, ICollection, IList, ICollection<T>, IEnumerable<T>, IList<T>, IMyList<T>
{
    /// <summary>
    ///  空数组
    /// </summary>
    static readonly T[] _emptyArray = new T[0];
    /// <summary>
    ///  默认容量数量
    /// </summary>
    private const int _defaultCapacity = 4;


    /// <summary>
    /// 所有项
    /// </summary>
    private T[] _items;
    /// <summary>
    /// 实际使用容量大小
    /// </summary>
    private int _size;
    /// <summary>
    /// 版本
    /// </summary>
    private int _version;


    #region 公开方法

    /// <summary>
    /// 初始化 其实就是长度位0的数组
    /// </summary>
    public MyList()
    {
        _items = _emptyArray;
    }

    /// <summary>
    /// 初始化
    /// </summary>
    /// <param name="collection">收集对象</param>
    public MyList(IEnumerable<T> collection)
    {
        if (collection == null)
        {
            return;
        }

        ICollection<T> c = collection as ICollection<T>;
        if (c != null)
        {
            int count = c.Count;
            if (count == 0)
            {
                _items = _emptyArray;
            }
            else
            {
                _items = new T[count];
                c.CopyTo(_items, 0);
                _size = count;
            }
        }
        else
        {
            _size = 0;
            _items = _emptyArray;

            using (IEnumerator<T> en = collection.GetEnumerator())
            {
                while (en.MoveNext())
                {
                    Add(en.Current);
                }
            }
        }
    }

    /// <summary>
    /// 初始化
    /// </summary>
    /// <param name="capacity">容量大小</param>
    public MyList(int capacity)
    {
        if (capacity < 0)
        {
            //异常
            return;
        }
        else if (capacity == 0)
        {
            _items = _emptyArray;
        }
        else
        {
            _items = new T[capacity];
        }
    }

    /// <summary>
    /// 获取或设置值
    /// </summary>
    /// <param name="index">索引</param>
    /// <returns></returns>
    public T this[int index]
    {
        get
        {
            if ((uint)index >= (uint)_size) { return default(T); }//异常

            return _items[index];
        }

        set
        {
            if ((uint)index >= (uint)_size) { return; }//异常

            _items[index] = value;
            _version++;
        }
    }

    /// <summary>
    /// 获得或设置实际容量大小
    /// </summary>
    public int Capacity
    {
        get
        {
            return _items.Length;
        }
        set
        {
            if (value < _size)
            {
                //新容量小于实际容量 异常
                return;
            }

            if (value == _items.Length)
            {
                //新容量等于实际容量  不处理
                return;
            }


            if (value > 0)
            {
                //创建一个新大小的容量
                T[] newItems = new T[value];
                if (_size > 0)
                {
                    //原本数据浅拷贝到新容量去
                    Array.Copy(_items, 0, newItems, 0, _size);
                }
                _items = newItems;
            }
            else
            {
                _items = _emptyArray;
            }
        }
    }

    /// <summary>
    /// 获得容量大小
    /// </summary>
    public int Count
    {
        get
        {
            return _size;
        }
    }

    /// <summary>
    /// 添加
    /// </summary>
    /// <param name="item">项</param>
    public void Add(T item)
    {
        if (_size == _items.Length)
        {
            EnsureCapacity(_size + 1);
        }
        _items[_size++] = item;
        _version++;
    }

    /// <summary>
    /// 添加一列
    /// </summary>
    /// <param name="collection">收集对象</param>
    public void AddRange(IEnumerable<T> collection)
    {
        InsertRange(_size, collection);
    }

    /// <summary>
    /// 返回当前集合的只读
    /// </summary>
    /// <returns></returns>
    public ReadOnlyCollection<T> AsreadOnly()
    {
        return new ReadOnlyCollection<T>(this);
    }

    /// <summary>
    /// 二分查找查找匹配项
    /// </summary>
    /// <param name="index">索引</param>
    /// <param name="count">匹配数量</param>
    /// <param name="item">匹配项</param>
    /// <param name="comparer">匹配方法</param>
    /// <returns></returns>
    public int BinarySearch(int index, int count, T item, IComparer<T> comparer)
    {
        if (index < 0) { return -1; }//异常
        if (count < 0) { return -1; }//异常
        if (_size - index < count) { return -1; }//异常

        return Array.BinarySearch<T>(_items, index, count, item, comparer);
    }

    /// <summary>
    /// 二分查找查找匹配项
    /// </summary>
    /// <param name="item">匹配项</param>
    /// <returns></returns>
    public int BinarySearch(T item)
    {
        return BinarySearch(0, Count, item, null);
    }

    /// <summary>
    /// 二分查找查找匹配项
    /// </summary>
    /// <param name="item">匹配项</param>
    /// <param name="comparer">匹配方法</param>
    /// <returns></returns>
    public int BinarySearch(T item, IComparer<T> comparer)
    {
        return BinarySearch(0, Count, item, comparer);
    }

    /// <summary>
    /// 清除数据
    /// </summary>
    public void Clear()
    {
        if (_size > 0)
        {
            Array.Clear(_items, 0, _size); // Don't need to doc this but we clear the elements so that the gc can reclaim the references.
            _size = 0;
        }
        _version++;
    }

    /// <summary>
    /// 判断是否包含(算法复杂度O(n))
    /// </summary>
    /// <param name="item">项</param>
    /// <returns></returns>
    public bool Contains(T item)
    {
        if ((Object)item == null)
        {
            for (int i = 0; i < _size; i++)
                if ((Object)_items[i] == null)
                    return true;
            return false;
        }
        else
        {
            EqualityComparer<T> c = EqualityComparer<T>.Default;
            for (int i = 0; i < _size; i++)
            {
                if (c.Equals(_items[i], item)) return true;
            }
            return false;
        }
    }

    /// <summary>
    /// 转换列表为其他类型
    /// </summary>
    /// <typeparam name="TOutput"></typeparam>
    /// <param name="converter"></param>
    /// <returns></returns>
    public MyList<TOutput> ConvertAll<TOutput>(Converter<T, TOutput> converter)
    {
        if (converter == null) { return null; }//异常

        MyList<TOutput> list = new MyList<TOutput>(_size);
        for (int i = 0; i < _size; i++)
        {
            list._items[i] = converter(_items[i]);
        }
        list._size = _size;
        return list;
    }

    /// <summary>
    /// 拷贝到
    /// </summary>
    /// <param name="index">索引</param>
    /// <param name="array">拷贝数组容器</param>
    /// <param name="arrayIndex">拷贝数组开始容器</param>
    /// <param name="count">数量</param>
    public void CopyTo(int index, T[] array, int arrayIndex, int count)
    {
        if (_size - index < count) { return; }//异常

        Array.Copy(_items, index, array, arrayIndex, count);
    }

    /// <summary>
    /// 拷贝到
    /// </summary>
    /// <param name="array">拷贝数组容器</param>
    /// <param name="arrayIndex">拷贝数组开始容器</param>
    public void CopyTo(T[] array, int arrayIndex)
    {
        Array.Copy(_items, 0, array, arrayIndex, _size);
    }

    /// <summary>
    /// 拷贝到
    /// </summary>
    /// <param name="array">拷贝数组容器</param>
    public void CopyTo(T[] array)
    {
        CopyTo(array, 0);
    }

    /// <summary>
    /// 判断是否存在
    /// </summary>
    /// <param name="match"></param>
    /// <returns></returns>
    public bool Exists(Predicate<T> match)
    {
        return FindIndex(match) != -1;
    }

    /// <summary>
    /// 从前查找匹配项索引
    /// </summary>
    /// <param name="match">匹配函数委托</param>
    /// <returns></returns>
    public int FindIndex(Predicate<T> match)
    {
        return FindIndex(0, _size, match);
    }

    /// <summary>
    /// 从前查找匹配项索引
    /// </summary>
    /// <param name="startIndex">查找开始索引</param>
    /// <param name="match">匹配函数委托</param>
    /// <returns></returns>
    public int FindIndex(int startIndex, Predicate<T> match)
    {
        return FindIndex(startIndex, _size - startIndex, match);
    }

    /// <summary>
    /// 从前查找匹配项索引
    /// </summary>
    /// <param name="startIndex">查找开始索引</param>
    /// <param name="count">查找数量</param>
    /// <param name="match">匹配函数委托</param>
    /// <returns></returns>
    public int FindIndex(int startIndex, int count, Predicate<T> match)
    {
        if ((uint)startIndex > (uint)_size) { return -1; }//异常

        if (count < 0 || startIndex > _size - count) { return -1; }//异常

        if (match == null) { return -1; }//异常

        int endIndex = startIndex + count;
        for (int i = startIndex; i < endIndex; i++)
        {
            if (match(_items[i])) return i;
        }
        return -1;
    }

    /// <summary>
    /// 从后查找匹配项索引
    /// </summary>
    /// <param name="match">匹配函数委托</param>
    /// <returns></returns>
    public T FindLast(Predicate<T> match)
    {
        if (match == null) { return default(T); }//异常

        for (int i = _size - 1; i >= 0; i--)
        {
            if (match(_items[i]))
            {
                return _items[i];
            }
        }
        return default(T);
    }

    /// <summary>
    /// 从后查找匹配项索引
    /// </summary>
    /// <param name="match">匹配函数委托</param>
    /// <returns></returns>
    public int FindLastIndex(Predicate<T> match)
    {
        return FindLastIndex(_size - 1, _size, match);
    }

    /// <summary>
    /// 从后查找匹配项索引
    /// </summary>
    /// <param name="startIndex">查找开始索引</param>
    /// <param name="match">匹配函数委托</param>
    /// <returns></returns>
    public int FindLastIndex(int startIndex, Predicate<T> match)
    {
        return FindLastIndex(startIndex, startIndex + 1, match);
    }

    /// <summary>
    /// 从后查找匹配项索引
    /// </summary>
    /// <param name="startIndex">查找开始位置</param>
    /// <param name="count">查找数量</param>
    /// <param name="match"></param>
    /// <returns></returns>
    public int FindLastIndex(int startIndex, int count, Predicate<T> match)
    {
        if (match == null) { return -1; }//异常

        if (_size == 0)
        {
            // 0长度列表的特殊情况
            if (startIndex != -1) { return -1; }//异常
        }
        else
        {
            // Make sure we're not out of range            
            if ((uint)startIndex >= (uint)_size) { return -1; }//异常 不在范围之内
        }

        if (count < 0 || startIndex - count + 1 < 0) { return -1; }//异常 参数不对

        int endIndex = startIndex - count;
        for (int i = startIndex; i > endIndex; i--)
        {
            if (match(_items[i]))
            {
                return i;
            }
        }
        return -1;
    }

    /// <summary>
    /// 对每项执行函数
    /// </summary>
    /// <param name="action">执行函数</param>
    public void ForEach(Action<T> action)
    {
        if (action == null) { return; }//异常

        int version = _version;

        for (int i = 0; i < _size; i++)
        {
            //防止多线程操作
            if (version != _version)
            {
                break;
            }
            action(_items[i]);
        }
    }

    /// <summary>
    /// 获得枚举对象
    /// </summary>
    /// <returns></returns>
    public Enumerator GetEnumerator()
    {
        return new Enumerator(this);
    }

    /// <summary>
    /// 获取范围对象
    /// </summary>
    /// <param name="index"></param>
    /// <param name="count"></param>
    /// <returns></returns>
    public MyList<T> GetRange(int index, int count)
    {
        if (index < 0) { return null; }//异常

        if (count < 0) { return null; }//异常

        if (_size - index < count) { return null; }//异常

        MyList<T> list = new MyList<T>(count);
        Array.Copy(_items, index, list._items, 0, count);
        list._size = count;
        return list;
    }

    /// <summary>
    /// 查找索引
    /// </summary>
    /// <param name="item">项</param>
    /// <returns></returns>
    public int IndexOf(T item)
    {
        return Array.IndexOf(_items, item, 0, _size);
    }

    /// <summary>
    /// 查找索引
    /// </summary>
    /// <param name="item">项</param>
    /// <param name="index">查找开始索引</param>
    /// <returns></returns>
    public int IndexOf(T item, int index)
    {
        if (index > _size) { return -1; }//异常

        return Array.IndexOf(_items, item, index, _size - index);
    }

    /// <summary>
    /// 查找索引
    /// </summary>
    /// <param name="item">项</param>
    /// <param name="index">查找开始索引</param>
    /// <param name="count">查找数量</param>
    /// <returns></returns>
    public int IndexOf(T item, int index, int count)
    {
        if (index > _size) { return -1; }//异常

        if (count < 0 || index > _size - count) { return -1; }//异常

        return Array.IndexOf(_items, item, index, count);
    }

    /// <summary>
    /// 插入一个数据
    /// </summary>
    /// <param name="index">索引</param>
    /// <param name="item">项</param>
    public void Insert(int index, T item)
    {
        // Note that insertions at the end are legal.
        if ((uint)index > (uint)_size)
        {
            return;//索引超出 异常
        }
        if (_size == _items.Length)
        {
            EnsureCapacity(_size + 1);//插入到最后需要扩容一下
        }
        if (index < _size)
        {
            Array.Copy(_items, index, _items, index + 1, _size - index);//把索引后面的数据向后挪一个位置
        }
        _items[index] = item;//替换指定索引位置的项
        _size++;
        _version++;
    }

    /// <summary>
    /// 插入一列数据
    /// </summary>
    /// <param name="index">索引</param>
    /// <param name="collection">收集对象</param>
    public void InsertRange(int index, IEnumerable<T> collection)
    {
        if (collection == null)
        {
            return;//异常
        }

        if ((uint)index > (uint)_size)
        {
            return;//插入位置错误 异常
        }

        ICollection<T> c = collection as ICollection<T>;
        if (c != null)
        {    // if collection is ICollection<T>
            int count = c.Count;
            if (count > 0)
            {
                //扩容
                EnsureCapacity(_size + count);
                if (index < _size)
                {
                    //容量够 直接拷贝进来
                    Array.Copy(_items, index, _items, index + count, _size - index);
                }

                //如果插入的列表是自己
                if (this == c)
                {
                    //分2步插入
                    Array.Copy(_items, 0, _items, index, index);
                    Array.Copy(_items, index + count, _items, index * 2, _size - index);
                }
                else
                {
                    T[] itemsToInsert = new T[count];
                    c.CopyTo(itemsToInsert, 0);
                    itemsToInsert.CopyTo(_items, index);
                }
                _size += count;
            }
        }
        else
        {
            using (IEnumerator<T> en = collection.GetEnumerator())
            {
                while (en.MoveNext())
                {
                    Insert(index++, en.Current);
                }
            }
        }
        _version++;
    }

    /// <summary>
    /// 从后查找索引
    /// </summary>
    /// <param name="item">项</param>
    /// <returns></returns>
    public int LastIndexOf(T item)
    {
        if (_size == 0)
        {  // Special case for empty list
            return -1;
        }
        else
        {
            return LastIndexOf(item, _size - 1, _size);
        }
    }

    /// <summary>
    /// 从后查找索引
    /// </summary>
    /// <param name="item">项</param>
    /// <param name="index">开始索引</param>
    /// <returns></returns>
    public int LastIndexOf(T item, int index)
    {
        if (index >= _size) { return -1; }//异常

        return LastIndexOf(item, index, index + 1);
    }

    /// <summary>
    /// 从后查找索引
    /// </summary>
    /// <param name="item">项</param>
    /// <param name="index">开始索引</param>
    /// <param name="count">数量</param>
    /// <returns></returns>
    public int LastIndexOf(T item, int index, int count)
    {
        if ((Count != 0) && (index < 0)) { return -1; }//异常

        if ((Count != 0) && (count < 0)) { return -1; }//异常

        if (_size == 0) { return -1; }//空列表的特殊情况

        if (index >= _size) { return -1; }//异常

        if (count > index + 1) { return -1; }//异常


        return Array.LastIndexOf(_items, item, index, count);
    }

    /// <summary>
    /// 删除项
    /// </summary>
    /// <param name="item"></param>
    /// <returns></returns>
    public bool Remove(T item)
    {
        int index = IndexOf(item);
        if (index >= 0)
        {
            RemoveAt(index);
            return true;
        }

        return false;
    }

    /// <summary>
    /// 删除索引项
    /// </summary>
    /// <param name="index"></param>
    public void RemoveAt(int index)
    {
        if ((uint)index >= (uint)_size) { return; }//异常

        _size--;

        if (index < _size)
        {
            Array.Copy(_items, index + 1, _items, index, _size - index);
        }
        _items[_size] = default(T);
        _version++;
    }

    /// <summary>
    /// 删除所有匹配项
    /// </summary>
    /// <param name="match"></param>
    /// <returns></returns>
    public int RemoveAll(Predicate<T> match)
    {
        if (match == null) { return -1; }//异常

        int freeIndex = 0;   // the first free slot in items array

        // 找到第一个需要删除的项目。
        while (freeIndex < _size && !match(_items[freeIndex])) freeIndex++;

        //没有匹配项
        if (freeIndex >= _size) return 0;

        int current = freeIndex + 1;
        while (current < _size)
        {
            // Find the first item which needs to be kept.
            while (current < _size && match(_items[current])) current++;

            if (current < _size)
            {
                // 将项复制到空闲槽。
                _items[freeIndex++] = _items[current++];
            }
        }

        Array.Clear(_items, freeIndex, _size - freeIndex);
        int result = _size - freeIndex;
        _size = freeIndex;
        _version++;
        return result;
    }

    /// <summary>
    /// 删除范围项
    /// </summary>
    /// <param name="index"></param>
    /// <param name="count"></param>
    public void RemoveRange(int index, int count)
    {
        if (index < 0) { return; }//异常

        if (count < 0) { return; }//异常

        if (_size - index < count) { return; }//异常

        if (count > 0)
        {
            int i = _size;
            _size -= count;
            if (index < _size)
            {
                Array.Copy(_items, index + count, _items, index, _size - index);
            }
            Array.Clear(_items, _size, count);
            _version++;
        }
    }

    /// <summary>
    /// 反转排序
    /// </summary>
    public void Reverse()
    {
        Reverse(0, Count);
    }

    /// <summary>
    /// 反转范围排序
    /// </summary>
    /// <param name="index"></param>
    /// <param name="count"></param>
    public void Reverse(int index, int count)
    {
        if (index < 0) { return; }//异常

        if (count < 0) { return; }//异常

        if (_size - index < count) { return; }//异常

        Array.Reverse(_items, index, count);
        _version++;
    }

    /// <summary>
    /// 排序
    /// </summary>
    public void Sort()
    {
        Sort(0, Count, null);
    }

    /// <summary>
    /// 排序
    /// </summary>
    /// <param name="comparer">指定方法</param>
    public void Sort(IComparer<T> comparer)
    {
        Sort(0, Count, comparer);
    }

    /// <summary>
    /// 排序
    /// </summary>
    /// <param name="index">排序开始索引</param>
    /// <param name="count">排序数量</param>
    /// <param name="comparer">指定方法</param>
    public void Sort(int index, int count, IComparer<T> comparer)
    {
        if (index < 0) { return; }//异常

        if (count < 0) { return; }//异常

        if (_size - index < count) { return; }//异常

        Array.Sort<T>(_items, index, count, comparer);
        _version++;
    }

    /// <summary>
    /// 排序
    /// </summary>
    /// <param name="comparison">指定方法</param>
    public void Sort(Comparison<T> comparison)
    {
        if (comparison == null) { return; }//异常

        if (_size > 0)
        {
            IComparer<T> comparer = new FunctorComparer<T>(comparison);
            Array.Sort(_items, 0, _size, comparer);
        }
    }

    /// <summary>
    /// 转换为数组
    /// </summary>
    /// <returns></returns>
    public T[] ToArray()
    {
        T[] array = new T[_size];
        Array.Copy(_items, 0, array, 0, _size);
        return array;
    }

    /// <summary>
    /// //将容器设置位实际数目
    /// </summary>
    public void TrimExcess()
    {
        int threshold = (int)(((double)_items.Length) * 0.9);
        if (_size < threshold)
        {
            Capacity = _size;
        }
    }

    /// <summary>
    /// 判断每个项目都匹配
    /// </summary>
    /// <param name="match"></param>
    /// <returns></returns>
    public bool TrueForAll(Predicate<T> match)
    {
        if (match == null) { return false; }//异常

        for (int i = 0; i < _size; i++)
        {
            if (!match(_items[i]))
            {
                return false;
            }
        }
        return true;
    }

    #endregion

    #region 私有方法

    /// <summary>
    /// 保护容量
    /// </summary>
    /// <param name="min">最小容量</param>
    private void EnsureCapacity(int min)
    {
        if (_items.Length < min)
        {
            int newCapacity;

            if (_items.Length == 0)
            {
                //如果本身长度为0则 新容量为4
                newCapacity = _defaultCapacity;
            }
            else
            {
                //新容量为原长度2被
                newCapacity = _items.Length * 2;
            }

            //不能超过int最大值
            if ((uint)newCapacity > int.MaxValue)
            {
                newCapacity = int.MaxValue;
            }
            //新容量不能小于参数
            if (newCapacity < min)
            {
                newCapacity = min;
            }
            Capacity = newCapacity;
        }
    }

    #endregion

    #region 接口实现

    IEnumerator IEnumerable.GetEnumerator()
    {
        return new Enumerator(this);
    }

    bool ICollection.IsSynchronized
    {
        get { return false; }
    }

    private Object _syncRoot;
    object ICollection.SyncRoot
    {
        get
        {
            if (_syncRoot == null)
            {
                System.Threading.Interlocked.CompareExchange<Object>(ref _syncRoot, new Object(), null);
            }
            return _syncRoot;
        }
    }

    void ICollection.CopyTo(Array array, int arrayIndex)
    {
        if ((array != null) && (array.Rank != 1)) { return; }//异常

        try
        {
            Array.Copy(_items, 0, array, arrayIndex, _size);
        }
        catch (ArrayTypeMismatchException) { return; }//异常
    }

    object IList.this[int index]
    {
        get { return this[index]; }
        set
        {
            try
            {
                this[index] = (T)value;
            }
            catch (InvalidCastException) { return; }//异常
        }
    }

    bool IList.IsFixedSize
    {
        get { return false; }
    }

    bool IList.IsReadOnly
    {
        get { return false; }
    }

    int IList.Add(Object item)
    {
        try
        {
            Add((T)item);
        }
        catch (InvalidCastException) { return -1; }//异常

        return Count - 1;
    }

    bool IList.Contains(Object item)
    {
        if (IsCompatibleObject(item))
        {
            return Contains((T)item);
        }
        return false;
    }

    int IList.IndexOf(Object item)
    {
        if (IsCompatibleObject(item))
        {
            return IndexOf((T)item);
        }
        return -1;
    }

    void IList.Insert(int index, object item)
    {
        Insert(index, (T)item);
    }

    void IList.Remove(object value)
    {
        if (IsCompatibleObject(value))
        {
            Remove((T)value);
        }
    }

    bool ICollection<T>.IsReadOnly
    {
        get { return false; }
    }

    IEnumerator<T> IEnumerable<T>.GetEnumerator()
    {
        return GetEnumerator();
    }

    private static bool IsCompatibleObject(object value)
    {
        return ((value is T) || (value == null && default(T) == null));
    }

    #endregion


    /// <summary>
    /// Comparison实现IComparer接口
    /// </summary>
    /// <typeparam name="T"></typeparam>
    internal sealed class FunctorComparer<T> : IComparer<T>
    {
        Comparison<T> comparison;

        public FunctorComparer(Comparison<T> comparison)
        {
            this.comparison = comparison;
        }

        public int Compare(T x, T y)
        {
            return comparison(x, y);
        }
    }

    [Serializable]
    public struct Enumerator : IEnumerator<T>, System.Collections.IEnumerator
    {
        private MyList<T> list;
        private int index;
        private int version;
        private T current;

        internal Enumerator(MyList<T> list)
        {
            this.list = list;
            index = 0;
            version = list._version;
            current = default(T);
        }

        public void Dispose()
        {
        }

        public bool MoveNext()
        {

            MyList<T> localList = list;

            if (version == localList._version && ((uint)index < (uint)localList._size))
            {
                current = localList._items[index];
                index++;
                return true;
            }
            return MoveNextRare();
        }

        private bool MoveNextRare()
        {
            if (version != list._version) { return false; }//异常

            index = list._size + 1;
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
                if (index == 0 || index == list._size + 1) { return false; }//异常

                return Current;
            }
        }

        void System.Collections.IEnumerator.Reset()
        {
            if (version != list._version) { return; }//异常

            index = 0;
            current = default(T);
        }
    }
}
```

