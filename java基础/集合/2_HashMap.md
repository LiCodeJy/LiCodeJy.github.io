# HashMap

### 底层结构

JDK1.7之前使用数组+链表，遇到哈希冲突使用拉链法

JDK1.8之后使用数组+链表/红黑树，遇到冲突先使用链表，如果链表长度大于8并且数组长度大于64，则将链表转为红黑树，提高搜索效率

### hash()

扰动函数==(h = key.hashCode()) ^ (h >>> 16)==

```java
    static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```

扰动函数是为了防止比较差的hashCode()，使高16位也能够参与计算，__减少碰撞__

### 关于HashMap长度

#### HashMap使用==tableSizeFor()==方法确保长度为2^n

```
static final int tableSizeFor(int cap) {
    int n = cap - 1;
    n |= n >>> 1;
    n |= n >>> 2;
    n |= n >>> 4;
    n |= n >>> 8;
    n |= n >>> 16;
    return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

其中的一系列右移取或操作可实现，将n的最高位1之后全部填充为1，最后加上1即为大于cap且最接近的2^n

int n = cap - 1是为了防止cap本身已经是2^n，最后+1导致出错

#### HashMap的长度为什么是2^n，

因为在散列时，HashMap使用==(n - 1) & hash==确定存储位置

当n为2^n时，(n - 1) & hash就相当于hash%n

#### 类属性

- loadFactor

  加载因子，控制数组存放数据的疏密程度，默认为0.75f

- capacity

  数组容量，默认为16

- threshold

  扩容阈值，threshold = capacity * loadFactor，当size大于threshold时，说明数组需要扩容了

#### 扩容

扩容时容量变为原来的两倍

数据在数组中的位置要么不变，要么变成 index+oldCapacity

例如为原来为36%16=4,扩容后为36%32依然为4

