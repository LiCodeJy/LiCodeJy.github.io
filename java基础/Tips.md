## Java Tips

#### 集合判空用 isEmpty()

#### 集合转Map

在使用 java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要注意当 value 为 null 时会抛 NPE 异常。

#### 集合遍历

不要在 foreach 循环里进行元素的 `remove/add` 操作。remove 元素请使用 `Iterator` 方式，如果并发操作，需要对 `Iterator` 对象加锁。

#### 集合转数组

使用集合转数组的方法，必须使用集合的 `toArray(T[] array)`，传入的是类型完全一致、长度为 0 的空数组。

String[] s=list.toArray(**new String[0]**);

#### 数组转集合

使用工具类 `Arrays.asList()` 把数组转换成集合时，不能使用其修改集合相关的方法， 它的 `add/remove/clear` 方法会抛出 `UnsupportedOperationException` 异常。

1. `Arrays.asList()`是泛型方法，传递的数组必须是对象数组，而不是基本类型。

2. 使用集合的修改方法: `add()`、`remove()`、`clear()`会抛出异常。

   `Arrays.asList()` 方法返回的并不是 `java.util.ArrayList` ，而是 `java.util.Arrays` 的一个内部类,这个内部类并没有实现集合的修改方法或者说并没有重写这些方法。

**正确使用**

1. 简洁

   ```java
   List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
   ```

   

2. java8 的 Stream

   ```java
   Integer [] myArray = { 1, 2, 3 };
   List myList = Arrays.stream(myArray).collect(Collectors.toList());
   //基本类型也可以实现转换（依赖boxed的装箱操作）
   int [] myArray2 = { 1, 2, 3 };
   List myList = Arrays.stream(myArray2).boxed().collect(Collectors.toList());
   ```


### 弱引用、软引用、强引用

- 弱引用：垃圾回收扫描它所在的内存区域时，一旦发现弱引用，就立刻回收
- 软引用：当内存充足时，不会被回收，内存不足时会被回收
- 强引用：绝对不会回收（new 一个对象就是强引用）