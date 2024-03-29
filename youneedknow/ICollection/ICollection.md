# ICollection<T>接口

## 定义

命名空间：`Sysetm.Collections.Generic`。
程序集：`System.Runtime.dll`。

定义操作泛型集合的方法，如添加、删除、判断元素是否存在等。
```c#
public interface ICollection<T> : System.Collections.Generic.IEnumerable<T>
```

## 类型参数

`T`：集合中元素的类型。

## 示例

以下示例实现接口以`ICollection<T>`创建名为`BoxCollection`的自定义`Box`对象的集合。每个属性都具有`Box`高度、长度和宽度属性，用于定义相等性。可以将相等性定义为所有维度相同或卷相同。类`Box`实现接口，`IEquatable<T>`以将默认相等性定义为维度相同。

类`BoxCollection`实现`Contains`方法，以确定集合中是否有这个`Box`。此方法由`Add`方法使用以便添加到集合中的每个`Box`都有一组唯一的维度。类`BoxCollection`还提供方法的重载，`Contains`该方法采用指定的`EqualityComparer<T>`对象，例如`BoxSameDimensions`示例中的和`BoxSamVol`类。

此示例还实现`IEnumerator<T>`类的`BoxCollection`接口，以便可以枚举集合。

```c#
using System;
using System.Collections;
using System.Collections.Generic;

class Program
{

    static void Main(string[] args)
    {

        BoxCollection bxList = new BoxCollection();

        bxList.Add(new Box(10, 4, 6));
        bxList.Add(new Box(4, 6, 10));
        bxList.Add(new Box(6, 10, 4));
        bxList.Add(new Box(12, 8, 10));

        // Same dimensions. Cannot be added:
        bxList.Add(new Box(10, 4, 6));

        // Test the Remove method.
        Display(bxList);
        Console.WriteLine("Removing 6x10x4");
        bxList.Remove(new Box(6, 10, 4));
        Display(bxList);

        // Test the Contains method.
        Box BoxCheck = new Box(8, 12, 10);
        Console.WriteLine("Contains {0}x{1}x{2} by dimensions: {3}",
            BoxCheck.Height.ToString(), BoxCheck.Length.ToString(),
            BoxCheck.Width.ToString(), bxList.Contains(BoxCheck).ToString());

        // Test the Contains method overload with a specified equality comparer.
        Console.WriteLine("Contains {0}x{1}x{2} by volume: {3}",
            BoxCheck.Height.ToString(), BoxCheck.Length.ToString(),
            BoxCheck.Width.ToString(), bxList.Contains(BoxCheck,
            new BoxSameVol()).ToString());
    }
    public static void Display(BoxCollection bxList)
    {
        Console.WriteLine("\nHeight\tLength\tWidth\tHash Code");
        foreach (Box bx in bxList)
        {
            Console.WriteLine("{0}\t{1}\t{2}\t{3}",
                bx.Height.ToString(), bx.Length.ToString(),
                bx.Width.ToString(), bx.GetHashCode().ToString());
        }

        // Results by manipulating the enumerator directly:

        //IEnumerator enumerator = bxList.GetEnumerator();
        //Console.WriteLine("\nHeight\tLength\tWidth\tHash Code");
        //while (enumerator.MoveNext())
        //{
        //    Box b = (Box)enumerator.Current;
        //    Console.WriteLine("{0}\t{1}\t{2}\t{3}",
        //    b.Height.ToString(), b.Length.ToString(),
        //    b.Width.ToString(), b.GetHashCode().ToString());
        //}

        Console.WriteLine();
    }

}

public class Box : IEquatable<Box>
{

    public Box(int h, int l, int w)
    {
        this.Height = h;
        this.Length = l;
        this.Width = w;
    }
    public int Height { get; set; }
    public int Length { get; set; }
    public int Width { get; set; }

    // Defines equality using the
    // BoxSameDimensions equality comparer.
    public bool Equals(Box other)
    {
        if (new BoxSameDimensions().Equals(this, other))
        {
            return true;
        }
        else
        {
            return false;
        }
    }

    public override bool Equals(object obj)
    {
        return base.Equals(obj);
    }

    public override int GetHashCode()
    {
        return base.GetHashCode();
    }
}

public class BoxCollection : ICollection<Box>
{
    // The generic enumerator obtained from IEnumerator<Box>
    // by GetEnumerator can also be used with the non-generic IEnumerator.
    // To avoid a naming conflict, the non-generic IEnumerable method
    // is explicitly implemented.

    public IEnumerator<Box> GetEnumerator()
    {
        return new BoxEnumerator(this);
    }
    IEnumerator IEnumerable.GetEnumerator()
    {
        return new BoxEnumerator(this);
    }

    // The inner collection to store objects.
    private List<Box> innerCol;

    public BoxCollection()
    {
        innerCol = new List<Box>();
    }

    // Adds an index to the collection.
    public Box this[int index]
    {
        get { return (Box)innerCol[index]; }
        set { innerCol[index] = value; }
    }

    // Determines if an item is in the collection
    // by using the BoxSameDimensions equality comparer.
    public bool Contains(Box item)
    {
        bool found = false;

        foreach (Box bx in innerCol)
        {
            // Equality defined by the Box
            // class's implmentation of IEquatable<T>.
            if (bx.Equals(item))
            {
                found = true;
            }
        }

        return found;
    }

    // Determines if an item is in the
    // collection by using a specified equality comparer.
    public bool Contains(Box item, EqualityComparer<Box> comp)
    {
        bool found = false;

        foreach (Box bx in innerCol)
        {
            if (comp.Equals(bx, item))
            {
                found = true;
            }
        }

        return found;
    }

    // Adds an item if it is not already in the collection
    // as determined by calling the Contains method.
    public void Add(Box item)
    {

        if (!Contains(item))
        {
            innerCol.Add(item);
        }
        else
        {
            Console.WriteLine("A box with {0}x{1}x{2} dimensions was already added to the collection.",
                item.Height.ToString(), item.Length.ToString(), item.Width.ToString());
        }
    }

    public void Clear()
    {
        innerCol.Clear();
    }

    public void CopyTo(Box[] array, int arrayIndex)
    {
        if (array == null)
           throw new ArgumentNullException("The array cannot be null.");
        if (arrayIndex < 0)
           throw new ArgumentOutOfRangeException("The starting array index cannot be negative.");
        if (Count > array.Length - arrayIndex)
           throw new ArgumentException("The destination array has fewer elements than the collection.");

        for (int i = 0; i < innerCol.Count; i++) {
            array[i + arrayIndex] = innerCol[i];
        }
    }

    public int Count
    {
        get
        {
            return innerCol.Count;
        }
    }

    public bool IsReadOnly
    {
        get { return false; }
    }

    public bool Remove(Box item)
    {
        bool result = false;

        // Iterate the inner collection to
        // find the box to be removed.
        for (int i = 0; i < innerCol.Count; i++)
        {

            Box curBox = (Box)innerCol[i];

            if (new BoxSameDimensions().Equals(curBox, item))
            {
                innerCol.RemoveAt(i);
                result = true;
                break;
            }
        }
        return result;
    }
}


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

// Defines two boxes as equal if they have the same dimensions.
public class BoxSameDimensions : EqualityComparer<Box>
{

    public override bool Equals(Box b1, Box b2)
    {
        if (b1.Height == b2.Height && b1.Length == b2.Length
                            && b1.Width == b2.Width)
        {
            return true;
        }
        else
        {
            return false;
        }
    }

    public override int GetHashCode(Box bx)
    {
        int hCode = bx.Height ^ bx.Length ^ bx.Width;
        return hCode.GetHashCode();
    }
}

// Defines two boxes as equal if they have the same volume.
public class BoxSameVol : EqualityComparer<Box>
{

    public override bool Equals(Box b1, Box b2)
    {
        if ((b1.Height * b1.Length * b1.Width) ==
                (b2.Height * b2.Length * b2.Width))
        {
            return true;
        }
        else
        {
            return false;
        }
    }

    public override int GetHashCode(Box bx)
    {
        int hCode = bx.Height ^ bx.Length ^ bx.Width;
        Console.WriteLine("HC: {0}", hCode.GetHashCode());
        return hCode.GetHashCode();
    }
}


/*
This code example displays the following output:
================================================

A box with 10x4x6 dimensions was already added to the collection.

Height  Length  Width   Hash Code
10      4       6       46104728
4       6       10      12289376
6       10      4       43495525
12      8       10      55915408

Removing 6x10x4

Height  Length  Width   Hash Code
10      4       6       46104728
4       6       10      12289376
12      8       10      55915408

Contains 8x12x10 by dimensions: False
Contains 8x12x10 by volume: True
 */
```

## 注解

接口`ICollection<T>`是命名空间中类`System.Collections.Generic`基接口。

接口`ICollection<T>`扩展`IEnumerable<T>;IDictionary<TKey,TValue>`和`IList<T>`是扩展`ICollection<T>`的更专用的接口，`IDictionary<TKey,TValue>`实现是键/值对的集合，如`Dictionary<TKey,TValue>`类。`IList<T>`实现是值的集合，其成员可以通过索引进行访问。

如果接口`IList<T>`和接口`IDictionary<TKey, TValue>`都不符合所需集合的要求，请从`ICollection<T>`接口派生新的集合类，以获得更大的灵活性。

## 属性

- `Count`：获取`IColletion<T>`中包含的元素数。
- `IsReadOnly`：获取一个值，该值指示`ICollection<T>`是否为只读。

## 方法

- `Add(T)`：将某项添加到`ICollection<T>`中。
- `Clear()`：从`ICollection<T>`中移除所有项。
- `Contains(T)`：确定`ICollection<T>`是否包含特定值。
- `CopyTo(T[], Int32)`：从特定的`ICollection<T>`索引开始，将`Array`的元素复制到一个`Array`中。
- `GetEnumerator()`：返回循环访问集合的枚举数。（继承自`IEnumerable`）。
- `Remove(T)`：从`ICollection<T>`中移除特定对象的第一个匹配项。
