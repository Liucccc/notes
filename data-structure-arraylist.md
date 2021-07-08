1. 数组是一种顺序存储的线性表，所有元素的内存地址是连续的
2. 缺点：可能会造成内存空间的大量浪费

![GkGgRO](https://gitee.com/Liuccccc/oss/raw/master/2021/06/23/GkGgRO.png)

```
//默认初始容量
private static final int DEFAULT_CAPACITY = 10;
//用于空实例的空数组
private static final Object[] EMPTY_ELEMENTDATA = {};

//用于默认大小的空实例的空数组实例。我们将其与空元素数据区分开来，以了解添加第一个元素时要充气多少
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

transient Object[] elementData; // non-private to simplify nested class access

private int size;
```

```
private void grow(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```