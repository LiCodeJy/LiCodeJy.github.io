## Tips



### 需要对集合进行排序操作可使用Collections工具类

```java
void reverse(List list)//反转
void sort(List list)//按自然排序的升序排序
void sort(List list, Comparator c)//定制排序，由Comparator控制排序逻辑
void swap(List list, int i , int j)//交换两个索引位置的元素
```



### 对数组填充

Array.fill(数组，值);



### 容器的选择

- 存储无序键值对时用HashMap，键值对有序时用TreeMap

- 存储有序不唯一数据，并且频繁读取时用ArrayList

  需要对尾部进行操作时（如回溯算法）用LinkedList

  需要队列FIFO操作时（如BFS）用Queue q = new LinkedList

- 存储唯一数据，用Set，无序用HashSet，有序用TreeSet