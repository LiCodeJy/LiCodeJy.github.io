# ArrayList

## 扩容机制

ArrayList的扩容机制提升了性能，如果每次只能扩容一个，频繁的插入会导致频繁的拷贝，降低性能，而扩容机制避免了这种情况

- 默认容量10

```java
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // 如果数组非空就取0
            ? 0
            //数组为空就给定默认值
            : DEFAULT_CAPACITY;

        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }

	//判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //开始扩容
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        //扩容原容量的1/2
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //如果扩容后容量还不满足最小所需，则扩容至最小所需
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //如果扩容后容量大于最大数组大小
        //private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```

由 int newCapacity = oldCapacity + (oldCapacity >> 1) 可见，ArrayList每次扩容之后==容量会变成原来容量的1.5倍左右==

#### 为什么每次扩容为1.5倍

为了充分再利用空间，(1,2)范围内是比较好的选择，1.5 可以用位运算

如果扩容后为原来的1.5倍，假设：

第一次：10	第二次：15	第三次：22

第三次（22）即可使用前两次废弃的空间（10+15=25）

如果扩容后为原来的2倍，假设：

第一次：10	第二次：20	第三次：40

前面废弃的空间永远无法被利用