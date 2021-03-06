ArrayList
定义
1 public class ArrayList<E> extends AbstractList<E>
2 implements List<E>, RandomAccess, Cloneable, java.io.Serializable
ArrayList实际上是一个动态数组，容量可以动态的增长，其继承了AbstractList，实现了List, RandomAccess, Cloneable, java.io.Serializable这些接口。

RandomAccess接口，被List实现之后，为List提供了随机访问功能，也就是通过下标获取元素对象的功能。

实现了Cloneable, java.io.Serializable意味着可以被克隆和序列化。

初始化
在使用中我们经常需要new出来各种泛型的ArrayList，那么在初始化过程是怎样的呢？

如下一行代码，创建一个ArrayList对象

List<Person> list = new ArrayList<>();
 

我们来看源码，是如何初始化的，找到构造方法

//无参构造方法
public ArrayList() {
  super();
  this.elementData = EMPTY_ELEMENTDATA;
}
 

看到这些代码的时候，我也是不解的，elementData和EMPTY_ELEMENTDATA是什么啊？但是很明显EMPTY_ELEMENTDATA是一个常量，追本溯源我们去看一下成员属性。

//如果是无参构造方法创建对象的话，ArrayList的初始化长度为10，这是一个静态常量
private static final int DEFAULT_CAPACITY = 10;
​
//在这里可以看到我们不解的EMPTY_ELEMENTDATA实际上是一个空的对象数组
    private static final Object[] EMPTY_ELEMENTDATA = {};
​
//保存ArrayList数据的对象数组缓冲区 空的ArrayList的elementData = EMPTY_ELEMENTDATA 这就是为什么说ArrayList底层是数组实现的了。elementData的初始容量为10，大小会根据ArrayList容量的增长而动态的增长。
    private transient Object[] elementData;
//集合的长度
    private int size;

 

执行完构造方法，如下图



可以说ArrayList的作者真的是很贴心，连缓存都处理好了，多次new出来的新对象，都指向同一个引用。

添加方法add
add(E e)

 /**
     * Appends the specified element to the end of this list.
     */
//增加元素到集合的最后
public boolean add(E e) {
  ensureCapacityInternal(size + 1);  // Increments modCount!!
  //因为++运算符的特点 先使用后运算  这里实际上是
  //elementData[size] = e
  //size+1
  elementData[size++] = e;
  return true;
}

 

先不管ensureCapacityInternal的话，这个方法就是将一个元素增加到数组的size++位置上。

再说回ensureCapacityInternal，它是用来扩容的，准确说是用来进行扩容检查的。下面我们来看一下整个扩容的过程


//初始长度是10，size默认值0，假定添加的是第一个元素，那么minCapacity=1 这是最小容量 如果小于这个容量就会报错
//如果底层数组就是默认数组，那么选一个大的值，就是10
private void ensureCapacityInternal(int minCapacity) {
        if (elementData == EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        //调用另一个方法ensureExplicitCapacity
        ensureExplicitCapacity(minCapacity);
    }
​
    private void ensureExplicitCapacity(int minCapacity) {
      //记录修改的次数
        modCount++;
​
        // overflow-conscious code
      //如果传过来的值大于初始长度 执行grow方法（参数为传过来的值）扩容
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }
//真正的扩容
 private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
   //新的容量是在原有的容量基础上+50% 右移一位就是二分之一
        int newCapacity = oldCapacity + (oldCapacity >> 1);
   //如果新容量小于最小容量，按照最小容量进行扩容
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
   //这里是重点 调用工具类Arrays的copyOf扩容
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
​
//Arrays
public static <T,U> T[] copyOf(U[] original, int newLength, Class<? extends T[]> newType) {
  T[] copy = ((Object)newType == (Object)Object[].class)
    ? (T[]) new Object[newLength]
    : (T[]) Array.newInstance(newType.getComponentType(), newLength);
  System.arraycopy(original, 0, copy, 0,
                   Math.min(original.length, newLength));
  return copy;
}
​



add(int index, E element)
添加到指定的位置


public void add(int index, E element) {
  //判断索引是否越界，如果越界就会抛出下标越界异常
  rangeCheckForAdd(index);
//扩容检查
  ensureCapacityInternal(size + 1);  // Increments modCount!!
  //将指定下标空出 具体作法就是index及其后的所有元素后移一位
  System.arraycopy(elementData, index, elementData, index + 1,size - index);
  //将要添加元素赋值到空出来的指定下标处
  elementData[index] = element;
  //长度加1
  size++;
}
//判断是否出现下标是否越界
private void rangeCheckForAdd(int index) {
  //如果下标超过了集合的尺寸 或者 小于0就是越界  
  if (index > size || index < 0)
    throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}

 

remove(int index)
ArrayList支持两种删除元素的方式

remove(int index) 按照下标删除 常用

remove(Object o) 按照元素删除 会删除和参数匹配的第一个元素

下面我们看一下ArrayList的实现


 /**
 移除list中指定位置的元素
     * Removes the element at the specified position in this list.
     所有后续元素左移 下标减1
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *参数是要移除元素的下标
     * @param index the index of the element to be removed
     返回值是被移除的元素
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
public E remove(int index) {
  //下标越界检查
  rangeCheck(index);
//修改次数统计
  modCount++;
  //获取这个下标上的值
  E oldValue = elementData(index);
//计算出需要移动的元素个数 （因为需要将后续元素左移一位 此处计算出来的是后续元素的位数）
  int numMoved = size - index - 1;
  //如果这个值大于0 说明后续有元素需要左移  是0说明被移除的对象就是最后一位元素
  if (numMoved > 0)
    //索引index只有的所有元素左移一位  覆盖掉index位置上的元素
    System.arraycopy(elementData, index+1, elementData, index,numMoved);
 // 将最后一个元素赋值为null  这样就可以被gc回收了
  elementData[--size] = null; // clear to let GC do its work
//返回index位置上的元素
  return oldValue;
}
​
//移除特定元素
public boolean remove(Object o) {
  //如果元素是null 遍历数组移除第一个null
  if (o == null) {
    for (int index = 0; index < size; index++)
      if (elementData[index] == null) {
        //遍历找到第一个null元素的下标 调用下标移除元素的方法
        fastRemove(index);
        return true;
      }
  } else {
    //找到元素对应的下标 调用下标移除元素的方法
    for (int index = 0; index < size; index++)
      if (o.equals(elementData[index])) {
        fastRemove(index);
        return true;
      }
  }
  return false;
}
​
//按照下标移除元素
private void fastRemove(int index) {
  modCount++;
  int numMoved = size - index - 1;
  if (numMoved > 0)
    System.arraycopy(elementData, index+1, elementData, index,
                     numMoved);
  elementData[--size] = null; // clear to let GC do its work
}

 

ArrayList总结
底层数组实现，使用默认构造方法初始化出来的容量是10

扩容的长度是在原长度基础上加二分之一

实现了RandomAccess接口，底层又是数组，get读取元素性能很好

线程不安全，所有的方法均不是同步方法也没有加锁，因此多线程下慎用

顺序添加很方便

删除和插入需要复制数组 性能很差（可以使用LinkindList）