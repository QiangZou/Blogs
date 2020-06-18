# C# LinkdeList链表源码分析 自定义LinkdeList链表

## 源码地址

https://referencesource.microsoft.com/#System/compmod/system/collections/generic/linkedlist.cs

## 关键点

- 双链表：每个节点包含上一节点信息和下一节点信息
- 循环链表：头节点的上一节点信息为尾节点
- 插入删除O(1)
- 遍历0(N)
- 获取Count属性O(1)
- 使用场景：频繁插入删除数据 遍历的适合插入删除数据
  - 日志系统
  - 定时器

## 自定义LinkdeList链表

```c#
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Runtime.Serialization;

/// <summary>
/// 自定义链接
/// 双重：每个节点都指向上个节点和下个节点
/// 循环：首尾相连
/// </summary>
/// <typeparam name="T"></typeparam>
public class MyLinkedList<T>
{
    internal MyLinkedListNode<T> head;
    internal int count;
    internal int version;

    private SerializationInfo siInfo;

    const String VersionName = "Version";
    const String CountName = "Count";
    const String ValuesName = "Data";

    public MyLinkedList()
    {
    }

    public MyLinkedList(IEnumerable<T> collection)
    {
        if (collection == null)
        {
            throw new ArgumentNullException("collection");
        }

        foreach (T item in collection)//循环添加项
        {
            AddLast(item);
        }
    }

    protected MyLinkedList(SerializationInfo info, StreamingContext context)
    {
        siInfo = info;
    }

    /// <summary>
    /// 获取链表数量
    /// </summary>
    public int Count
    {
        get { return count; }
    }

    /// <summary>
    /// 获取第一个节点
    /// </summary>
    public MyLinkedListNode<T> First
    {
        get { return head; }
    }

    /// <summary>
    /// 获取最后一个节点
    /// </summary>
    public MyLinkedListNode<T> Last
    {
        get { return head == null ? null : head.prev; }
    }

    /// <summary>
    /// 添加在node节点之后
    /// </summary>
    /// <param name="node">本链表的节点</param>
    /// <param name="value">新值</param>
    /// <returns></returns>
    public MyLinkedListNode<T> AddAfter(MyLinkedListNode<T> node, T value)
    {
        ValidateNode(node);//验证node节点是否是本链表
        MyLinkedListNode<T> result = new MyLinkedListNode<T>(node.list, value);//新建链表
        InternalInsertNodeBefore(node.next, result);//插入链表
        return result;
    }
    /// <summary>
    /// 添加在node节点之后
    /// </summary>
    /// <param name="node">本链表的节点</param>
    /// <param name="newNode">新节点</param>
    public void AddAfter(MyLinkedListNode<T> node, MyLinkedListNode<T> newNode)
    {
        ValidateNode(node);
        ValidateNewNode(newNode);//验证新节点是否为空
        InternalInsertNodeBefore(node.next, newNode);
        newNode.list = this;//赋值新节点的链表为本身
    }

    /// <summary>
    ///  添加在node节点之前
    /// </summary>
    /// <param name="node">本链表的节点</param>
    /// <param name="value">新值</param>
    /// <returns></returns>
    public MyLinkedListNode<T> AddBefore(MyLinkedListNode<T> node, T value)
    {
        ValidateNode(node);
        MyLinkedListNode<T> result = new MyLinkedListNode<T>(node.list, value);
        InternalInsertNodeBefore(node, result);
        if (node == head)
        {
            head = result;//如果插入的是头节点  则替换头节点
        }
        return result;
    }

    /// <summary>
    /// 添加在node节点之前
    /// </summary>
    /// <param name="node">本链表的节点</param>
    /// <param name="newNode">新节点</param>
    public void AddBefore(MyLinkedListNode<T> node, MyLinkedListNode<T> newNode)
    {
        ValidateNode(node);
        ValidateNewNode(newNode);
        InternalInsertNodeBefore(node, newNode);
        newNode.list = this;
        if (node == head)
        {
            head = newNode;
        }
    }

    /// <summary>
    /// 添加到头节点
    /// </summary>
    /// <param name="value">新值</param>
    /// <returns></returns>
    public MyLinkedListNode<T> AddFirst(T value)
    {
        MyLinkedListNode<T> result = new MyLinkedListNode<T>(this, value);
        if (head == null)
        {
            InternalInsertNodeToEmptyList(result);//头节点为空 赋值头节点
        }
        else
        {
            InternalInsertNodeBefore(head, result);
            head = result;//赋值头节点
        }
        return result;
    }

    /// <summary>
    /// 添加到头节点
    /// </summary>
    /// <param name="node">新节点</param>
    public void AddFirst(MyLinkedListNode<T> node)
    {
        ValidateNewNode(node);

        if (head == null)
        {
            InternalInsertNodeToEmptyList(node);
        }
        else
        {
            InternalInsertNodeBefore(head, node);
            head = node;
        }
        node.list = this;
    }

    /// <summary>
    /// 添加节点到末尾
    /// </summary>
    /// <param name="value">新值</param>
    /// <returns></returns>
    public MyLinkedListNode<T> AddLast(T value)
    {
        MyLinkedListNode<T> result = new MyLinkedListNode<T>(this, value);
        if (head == null)
        {
            InternalInsertNodeToEmptyList(result);//头节点为空 赋值头节点
        }
        else
        {
            InternalInsertNodeBefore(head, result);//插入节点
        }
        return result;
    }

    /// <summary>
    /// 添加节点到末尾
    /// </summary>
    /// <param name="node">新节点</param>
    public void AddLast(MyLinkedListNode<T> node)
    {
        ValidateNewNode(node);//验证新节点

        if (head == null)
        {
            InternalInsertNodeToEmptyList(node);
        }
        else
        {
            InternalInsertNodeBefore(head, node);
        }
        node.list = this;
    }

    /// <summary>
    /// 清理链表
    /// </summary>
    public void Clear()
    {
        MyLinkedListNode<T> current = head;
        while (current != null)
        {
            MyLinkedListNode<T> temp = current;
            current = current.Next; //遍历下一项
            temp.Invalidate();
        }

        head = null;
        count = 0;
        version++;
    }

    /// <summary>
    /// 判断是否包含项
    /// </summary>
    /// <param name="value"></param>
    /// <returns></returns>
    public bool Contains(T value)
    {
        return Find(value) != null;
    }

    /// <summary>
    /// 拷贝到数组
    /// </summary>
    /// <param name="array">数组容器</param>
    /// <param name="index">数组开始拷贝的索引</param>
    public void CopyTo(T[] array, int index)
    {
        if (array == null)
        {
            throw new ArgumentNullException("array");
        }

        if (index < 0 || index > array.Length)
        {
            throw new ArgumentOutOfRangeException("index", "index < 0 || index > array.Length");
        }

        if (array.Length - index < Count)
        {
            throw new ArgumentException("array.Length - index < Count");
        }

        MyLinkedListNode<T> node = head;
        if (node != null)
        {
            do
            {
                array[index++] = node.item;//赋值到数组中
                node = node.next;
            } while (node != head);
        }
    }

    /// <summary>
    /// 查找值的节点
    /// </summary>
    /// <param name="value">查找值</param>
    /// <returns></returns>
    public MyLinkedListNode<T> Find(T value)
    {
        MyLinkedListNode<T> node = head;
        EqualityComparer<T> c = EqualityComparer<T>.Default;//T类型的对比函数
        if (node != null)//头节点不为空
        {
            if (value != null)//查找值不为空
            {
                do
                {
                    if (c.Equals(node.item, value))
                    {
                        return node;
                    }
                    node = node.next;
                } while (node != head);
            }
            else
            {
                do
                {
                    if (node.item == null)//查找值为空则判断 节点值是否为空
                    {
                        return node;
                    }
                    node = node.next;
                } while (node != head);
            }
        }
        return null;
    }

    /// <summary>
    /// 查找值的节点从后
    /// </summary>
    /// <param name="value"></param>
    /// <returns></returns>
    public MyLinkedListNode<T> FindLast(T value)
    {
        if (head == null) return null;

        MyLinkedListNode<T> last = head.prev;//缓存最后一个节点
        MyLinkedListNode<T> node = last;
        EqualityComparer<T> c = EqualityComparer<T>.Default;
        if (node != null)
        {
            if (value != null)
            {
                do
                {
                    if (c.Equals(node.item, value))
                    {
                        return node;
                    }

                    node = node.prev;//赋值上一节点
                } while (node != last);
            }
            else
            {
                do
                {
                    if (node.item == null)
                    {
                        return node;
                    }
                    node = node.prev;
                } while (node != last);
            }
        }
        return null;
    }

    /// <summary>
    /// 获取循环枚举数
    /// </summary>
    /// <returns></returns>
    public Enumerator GetEnumerator()
    {
        return new Enumerator(this);
    }

    /// <summary>
    /// 获取序列化数据
    /// </summary>
    /// <param name="info"></param>
    /// <param name="context"></param>
    public virtual void GetObjectData(SerializationInfo info, StreamingContext context)
    {
        if (info == null)
        {
            throw new ArgumentNullException("info");
        }
        info.AddValue(VersionName, version);
        info.AddValue(CountName, count); 
        if (count != 0)
        {
            T[] array = new T[Count];
            CopyTo(array, 0);
            info.AddValue(ValuesName, array, typeof(T[]));
        }
    }

    /// <summary>
    /// 反序列化
    /// </summary>
    /// <param name="sender"></param>
    public virtual void OnDeserialization(Object sender)
    {
        if (siInfo == null)
        {
            return; 
        }

        int realVersion = siInfo.GetInt32(VersionName);
        int count = siInfo.GetInt32(CountName);

        if (count != 0)
        {
            T[] array = (T[])siInfo.GetValue(ValuesName, typeof(T[]));

            if (array == null)
            {
                throw new SerializationException("array == null");
            }
            for (int i = 0; i < array.Length; i++)
            {
                AddLast(array[i]);
            }
        }
        else
        {
            head = null;
        }

        version = realVersion;
        siInfo = null;
    }

    /// <summary>
    /// 删除值对应的节点
    /// </summary>
    /// <param name="value"></param>
    /// <returns></returns>
    public bool Remove(T value)
    {
        MyLinkedListNode<T> node = Find(value);
        if (node != null)
        {
            InternalRemoveNode(node);
            return true;
        }
        return false;
    }

    /// <summary>
    /// 删除节点
    /// </summary>
    /// <param name="node"></param>
    public void Remove(MyLinkedListNode<T> node)
    {
        ValidateNode(node);
        InternalRemoveNode(node);
    }

    /// <summary>
    /// 删除头节点
    /// </summary>
    public void RemoveFirst()
    {
        if (head == null) { throw new InvalidOperationException("head == null"); }
        InternalRemoveNode(head);
    }

    /// <summary>
    /// 删除尾节点
    /// </summary>
    public void RemoveLast()
    {
        if (head == null) { throw new InvalidOperationException("head == null"); }
        InternalRemoveNode(head.prev);
    }



    /// <summary>
    /// 内部插入节点之前
    /// </summary>
    /// <param name="node"></param>
    /// <param name="newNode"></param>
    private void InternalInsertNodeBefore(MyLinkedListNode<T> node, MyLinkedListNode<T> newNode)
    {
        newNode.next = node;//赋值新节点的下一节点
        newNode.prev = node.prev;//赋值新节点的上一节点
        node.prev.next = newNode;//赋值node的上一节点的下一节点为新节点
        node.prev = newNode;//赋值node的上衣节点为新节点
        version++;
        count++;
    }

    /// <summary>
    /// 内部插入节点到空列表
    /// </summary>
    /// <param name="newNode"></param>
    private void InternalInsertNodeToEmptyList(MyLinkedListNode<T> newNode)
    {
        Debug.Assert(head == null && count == 0, "LinkedList must be empty when this method is called!");
        newNode.next = newNode;//首位相连
        newNode.prev = newNode;//首位相连
        head = newNode;//赋值新节点为该链表的头节点
        version++;
        count++;
    }

    /// <summary>
    /// 内部删除节点
    /// </summary>
    /// <param name="node"></param>
    internal void InternalRemoveNode(MyLinkedListNode<T> node)
    {
        Debug.Assert(node.list == this, "Deleting the node from another list!");
        Debug.Assert(head != null, "This method shouldn't be called on empty list!");
        if (node.next == node)//节点数为1
        {
            Debug.Assert(count == 1 && head == node, "this should only be true for a list with only one node");
            head = null;
        }
        else
        {
            node.next.prev = node.prev;
            node.prev.next = node.next;
            if (head == node)
            {
                head = node.next;//删除节点为头节点
            }
        }
        node.Invalidate();
        count--;
        version++;
    }

    /// <summary>
    /// 验证新节点
    /// </summary>
    /// <param name="node"></param>
    internal void ValidateNewNode(MyLinkedListNode<T> node)
    {
        if (node == null)
        {
            throw new ArgumentNullException("node");
        }

        if (node.list != null)
        {
            throw new InvalidOperationException("node.list");
        }
    }

    /// <summary>
    /// 验证节点
    /// </summary>
    /// <param name="node"></param>
    internal void ValidateNode(MyLinkedListNode<T> node)
    {
        if (node == null)
        {
            throw new ArgumentNullException("node");
        }

        if (node.list != this)
        {
            throw new InvalidOperationException("node.list");
        }
    }


    /// <summary>
    /// 枚举数
    /// </summary>
    public struct Enumerator : IEnumerator<T>, System.Collections.IEnumerator, ISerializable, IDeserializationCallback
    {
        private MyLinkedList<T> list;
        private MyLinkedListNode<T> node;
        private int version;
        private T current;
        private int index;

        private SerializationInfo siInfo; //A temporary variable which we need during deserialization.


        const string LinkedListName = "LinkedList";
        const string CurrentValueName = "Current";
        const string VersionName = "Version";
        const string IndexName = "Index";

        internal Enumerator(MyLinkedList<T> list)
        {
            this.list = list;
            version = list.version;
            node = list.head;
            current = default(T);
            index = 0;

            siInfo = null;

        }


        internal Enumerator(SerializationInfo info, StreamingContext context)
        {
            siInfo = info;
            list = null;
            version = 0;
            node = null;
            current = default(T);
            index = 0;
        }


        public T Current
        {
            get { return current; }
        }

        object System.Collections.IEnumerator.Current
        {
            get
            {
                if (index == 0 || (index == list.Count + 1))
                {
                    throw new ArgumentNullException("index == 0 || (index == list.Count + 1)");
                }

                return current;
            }
        }

        public bool MoveNext()
        {
            if (version != list.version)
            {
                throw new InvalidOperationException("version != list.version");
            }

            if (node == null)
            {
                index = list.Count + 1;
                return false;
            }

            ++index;
            current = node.item;
            node = node.next;
            if (node == list.head)
            {
                node = null;
            }
            return true;
        }

        void System.Collections.IEnumerator.Reset()
        {
            if (version != list.version)
            {
                throw new InvalidOperationException("version != list.version");
            }

            current = default(T);
            node = list.head;
            index = 0;
        }

        public void Dispose()
        {
        }


        void ISerializable.GetObjectData(SerializationInfo info, StreamingContext context)
        {
            if (info == null)
            {
                throw new ArgumentNullException("info");
            }

            info.AddValue(LinkedListName, list);
            info.AddValue(VersionName, version);
            info.AddValue(CurrentValueName, current);
            info.AddValue(IndexName, index);
        }

        void IDeserializationCallback.OnDeserialization(Object sender)
        {
            if (list != null)
            {
                return; //Somebody had a dependency on this Dictionary and fixed us up before the ObjectManager got to it.
            }

            if (siInfo == null)
            {
                throw new SerializationException("version != list.version");
            }

            list = (MyLinkedList<T>)siInfo.GetValue(LinkedListName, typeof(MyLinkedList<T>));
            version = siInfo.GetInt32(VersionName);
            current = (T)siInfo.GetValue(CurrentValueName, typeof(T));
            index = siInfo.GetInt32(IndexName);

            if (list.siInfo != null)
            {
                list.OnDeserialization(sender);
            }

            if (index == list.Count + 1)
            {  // end of enumeration
                node = null;
            }
            else
            {
                node = list.First;
                // We don't care if we can point to the correct node if the LinkedList was changed   
                // MoveNext will throw upon next call and Current has the correct value. 
                if (node != null && index != 0)
                {
                    for (int i = 0; i < index; i++)
                    {
                        node = node.next;
                    }
                    if (node == list.First)
                    {
                        node = null;
                    }
                }
            }
            siInfo = null;
        }
    }
}


/// <summary>
/// 自定义节点
/// </summary>
/// <typeparam name="T"></typeparam>
public sealed class MyLinkedListNode<T>
{
    internal MyLinkedList<T> list;
    internal MyLinkedListNode<T> next;
    internal MyLinkedListNode<T> prev;
    internal T item;

    public MyLinkedListNode(T value)
    {
        this.item = value;
    }

    internal MyLinkedListNode(MyLinkedList<T> list, T value)
    {
        this.list = list;
        this.item = value;
    }

    public MyLinkedList<T> List
    {
        get { return list; }
    }

    public MyLinkedListNode<T> Next
    {
        get { return next == null || next == list.head ? null : next; }
    }

    public MyLinkedListNode<T> Previous
    {
        get { return prev == null || this == list.head ? null : prev; }
    }

    public T Value
    {
        get { return item; }
        set { item = value; }
    }

    /// <summary>
    /// 无效数据
    /// </summary>
    internal void Invalidate()
    {
        list = null;
        next = null;
        prev = null;
    }
}
```

