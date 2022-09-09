
## 集合概述
### 常见的集合有哪些？重要 

Java集合类主要由两个接口 **`Collection`** 和 **`Map`** 派生出来的，Collection有三个子接口：**`List、Set、Queue`**。

- List代表了有序可重复集合，可直接根据元素的索引来访问；
- Set代表无序不可重复集合，只能根据元素本身来访问；
- Queue是队列集合；
- Map代表的是存储key-value对的集合，可根据元素的key来访问value。

集合体系中常用的实现类有ArrayList、LinkedList、HashSet、TreeSet、HashMap、TreeMap等实现类。
### Arraylist 与 LinkedList 区别? 重要

- ArrayList基于 **`动态数组`** 实现；LinkedList基于链表实现。
- 对于随机index访问的get和set方法，ArrayList要比LinkedList快得多，ArrayList遍历最大的优势在于 **`内存的连续性`**，CPU的内部缓存结构会缓存连续的内存片段，可以大幅降低读取内存的性能开销。
	- 因为ArrayList直接通过数组下标直接找到元素；LinkedList要移动指针遍历每个元素直到找到为止。
- <font color=red>**删除元素，LinkedList的速度要优于ArrayList**</font>ArrayList在新增和删除元素时，**`可能扩容和复制数组`** ；LinkedList实例化对象需要时间外，只需要修改指针即可。
### Arraylist 和 Vector 的区别？ArrayList的遍历和LinkedList遍历性能⽐较如何？
- ArrayList在内存不够时默认是扩展50% + 1个，Vector是默认扩展1倍。
- Vector属于线程安全级别的，但是大多数情况下不使用Vector，因为操作Vector效率比较低。
### 说一下Hashtable的锁机制 ?
Hashtable是使用Synchronized来实现线程安全的，<font color=red>**给整个哈希表加了一把大锁**</font>，多线程访问时候，只要有一个线程访问或操作该对象，那其他线程只能阻塞等待需要的锁被释放，在竞争激烈的多线程场景中性能就会非常差！
### ConcurrentHashMap 和Hashtable的效率哪个更高？为什么？
- ConcurrentHashMap 的效率要高于Hashtable
- 因为Hashtable给整个哈希表加了一把大锁从而实现线程安全。
- 而ConcurrentHashMap 的锁粒度更低，在JDK1.7中采用分段锁实现线程安全，在JDK1.8 中采用CAS+Synchronized 实现线程安全。
### 讲一下TreeMap？

TreeMap是一个能比较元素大小的Map集合，会对传入的key进行了大小排序。可以使用元素的自然顺序，也可以使用集合中自定义的比较器来进行排序。

```java
public class TreeMap<K,V>
    extends AbstractMap<K,V>
    implements NavigableMap<K,V>, Cloneable, java.io.Serializable {
}
```
TreeMap 的继承结构：

![在这里插入图片描述](https://img-blog.csdnimg.cn/4b332872da354dc7b7b959d148e743dd.png?x-oss-process=image)
TreeMap的特点：

- TreeMap是**有序的key-value集合**，**通过红黑树实现**。根据键的自然顺序进行排序或根据提供的Comparator进行排序。
- TreeMap继承了AbstractMap，实现了NavigableMap接口，支持一系列的导航方法，给定具体搜索目标，可以返回最接近的匹配项。

### CopyOnWrite是什么 TODO TODO？

写时复制。当我们往容器添加元素时，不直接往容器添加，而是先将当前容器进行复制，复制出一个新的容器，然后往新的容器添加元素，添加完元素之后，再将原容器的引用指向新容器。这样做的好处就是可以对CopyOnWrite容器进行并发的读而不需要加锁，因为当前容器不会被修改。

从JDK1.5开始Java并发包里提供了两个使用CopyOnWrite机制实现的并发容器，它们是CopyOnWriteArrayList和CopyOnWriteArraySet。

CopyOnWriteArrayList中add方法添加的时候是需要加锁的，保证同步，避免了多线程写的时候复制出多个副本。**读的时候不需要加锁，如果读的时候有其他线程正在向CopyOnWriteArrayList添加数据，还是可以读到旧的数据。**

**缺点：**

- 内存占用问题。由于CopyOnWrite的写时复制机制，在进行写操作的时候，内存里会同时驻扎两个对象的内存。
- CopyOnWrite容器不能保证数据的实时一致性，可能读取到旧数据。
> 哪些集合类是线程安全的？哪些不安全？

线性安全的集合类：
- Vector：比ArrayList多了同步机制。
- Hashtable。
- ConcurrentHashMap：是一种高效并且线程安全的集合。
- Stack：栈，也是线程安全的，继承于Vector。

线性不安全的集合类：

- Hashmap
- Arraylist
- LinkedList
- HashSet
- TreeSet
- TreeMap
### 迭代器 Iterator 是什么？

Iterator 模式使用同样的逻辑来遍历集合。它可以把访问逻辑从不同类型的集合类中抽象出来，不需要了解集合内部实现便可以遍历集合元素，统一使用 Iterator 提供的接口去遍历。**它的特点是更加安全，因为它可以保证，在当前遍历的集合元素被更改的时候，就会抛出 ConcurrentModificationException 异常**。

主要有三个方法：hasNext()、next()和remove()。
### Iterator 和 ListIterator 有什么区别？

ListIterator 是 Iterator的增强版。

- ListIterator遍历可以是逆向的，因为有previous()和hasPrevious()方法，而Iterator不可以。
- ListIterator有add()方法，可以向List添加对象，而Iterator却不能。
- ListIterator可以定位当前的索引位置，因为有nextIndex()和previousIndex()方法，而Iterator不可以。
- ListIterator可以实现对象的修改，set()方法可以实现。Iierator仅能遍历，不能修改。
- ListIterator只能用于遍历List及其子类，Iterator可用来遍历所有集合。
## ArrayList
### ArrayList 了解吗？

ArrayList 的底层是 **`动态数组`** ，它的容量能动态增长。在添加大量元素前，应用可以使用 **`ensureCapacity`** 操作增加 ArrayList 实例的容量。ArrayList 继承了 AbstractList ，并实现了 List 接口。
### 成员变量 & 构造函数
> 成员变量

ArrayList 底层是基于数组来实现容量大小动态变化的。

```java
/**
* The size of the ArrayList (the number of elements it contains).
*/
private int size;  // 实际元素个数
transient Object[] elementData; 
// 默认初始容量大小为 10;
private static final int DEFAULT_CAPACITY = 10;
```
注意：上面的 size 是指 elementData 中实际有多少个元素，而 elementData.length 为集合容量，表示最多可以容纳多少个元素。

这个变量是定义在 AbstractList 中的。记录对 List 操作的次数。主要使用是在 Iterator，是防止在迭代的过程中集合被修改。

```java
protected transient int modCount = 0;
```

```java
private static final Object[] EMPTY_ELEMENTDATA = {};

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

```
两个空的数组有什么区别呢？ 简单来讲就是第一次添加元素时知道该 elementData 从空的构造函数还是有参构造函数被初始化的。以便确认如何扩容。
> ArrayList构造函数

以无参数构造方式创建ArrayList时，**实际上初始化赋值的是一个空数组（public ArrayList()）**。**当真正对数组进行添加元素操作时，才真正分配容量。`即向数组中添加第一个元素时，数组容量扩为10。`**
```java
// 初始化值为 10
private static final int DEFAULT_CAPACITY = 10;
//定义一个空数组以供使用
private static final Object[] EMPTY_ELEMENTDATA = {};
//也是一个空数组，跟上边的空数组不同之处在于，这个是在默认构造器时返回的，扩容时需要用到这个作判断，后面会讲到
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
//存放数组中的元素，注意此变量是transient修饰的，不参与序列化
transient Object[] elementData;
//数组的长度，此参数是数组中实际的参数，区别于elementData.length，后边会说到
private int size;
```
- 调用无参构造函数，返回了一个空的数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，此数组长度为0.
	```java
	// 调用此构造函数，返回了一个空的数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA，
	// 此数组长度为0.
	public ArrayList() {
	    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
	}
	```
- 调用有参构造，就是构造一个具有指定长度的空数组，当initialCapacity为0时，返回EMPTY_ELEMENTDATA
	```java
	//带初始容量参数的构造函数。（用户自己指定容量）
	public ArrayList(int initialCapacity) {
	    if (initialCapacity > 0) {
	        this.elementData = new Object[initialCapacity];
	    } else if (initialCapacity == 0) {
	        this.elementData = EMPTY_ELEMENTDATA;
	    } else {
	        throw new IllegalArgumentException("Illegal Capacity: "+
	                                           initialCapacity);
	    }
	} 
	```
- 包含特定集合元素的构造函数，把传入的集合转换为数组，然后通过Arrays.copyOf方法把集合中的元素拷贝到elementData中。同样，若传入的集合长度为0，返回EMPTY_ELEMENTDATA
	```java
	public ArrayList(Collection<? extends E> c) {
	    elementData = c.toArray();
	    if ((size = elementData.length) != 0) {
	        // c.toArray might (incorrectly) not return Object[] (see 6260652)
	        if (elementData.getClass() != Object[].class)
	            elementData = Arrays.copyOf(elementData, size, Object[].class);
	    } else {
	        // replace with empty array.
	        this.elementData = EMPTY_ELEMENTDATA;
	    }
	}
	
	```
### 说说ArrayList的扩容机制？（重要）

ArrayList扩容的本质就是 **`计算出新的扩容数组的size后实例化，并将原有数组内容复制到新数组中去。`**  默认情况下， **`新的容量会是原容量的1.5倍`** 。

- add()方法在添加一个元素时，会调用 **`ensureCapacityInternal(size + 1)`**  方法，**`把数组实际容量加1判断是否能存下下一个数据`** ，如果能装下才进行元素的添加，否则进行扩容；
- 首先 **`计算所需的最小容量`**
	- 如果传入的是个空数组则最小容量 取 **`默认容量 10`** 与 **`minCapacity`** 之间的 **`最大值`**
	- 否则直接返回 minCapacity，minCapacity就是size + 1
- 如果 **`minCapacity < elementData.length`**，直接将元素添加到数组最后
- 否则调用 **`grow(minCapacity);`** 进行扩容（**`minCapacity - elementData.length > 0`**）
	- 计算新容量（原来的1.5倍，旧容量 + 旧容量右移一位），**`int newCapacity = oldCapacity + (oldCapacity >> 1);`**
	- 校验容量是否够，**`if (newCapacity - minCapacity < 0)`**
	- 若预设值大于默认的最大值，检查是否溢出，**`if (newCapacity - MAX_ARRAY_SIZE > 0)`**
	- 最后，调用Arrays.copyOf方法将elementData数组指向新的内存空间，并将elementData的数据复制到新的内存空间。
		```java
		elementData = Arrays.copyOf(elementData, newCapacity);
		```

### 说说删除操作流程？
- 当我们调用 remove(int index) 时，首先会检查 index 是否合法（是否超出边界），然后再判断要删除的元素是否位于数组的最后一个位置。
	- 如果 index 不是最后一个，就再次调用 **`System.arraycopy()`** 方法拷贝数组，将从 index + 1 开始向后所有的元素都向前挪一个位置。然后将数组的最后一个位置空，size - 1。
	- 如果 index 是最后一个元素那么就直接将数组的最后一个位置空，size - 1即可。
- 当我们调用 remove(Object o) 时，会把 o 分为是否为空来分别处理。然后对数组做遍历，找到第一个与 o 对应的下标 index，然后调用 fastRemove 方法，删除下标为 index 的元素。其实仔细观察 fastRemove(int index) 方法和 remove(int index) 方法基本全部相同。
```java
public E remove(int index) {
    rangeCheck(index);

    modCount++;
    E oldValue = elementData(index);

    int numMoved = size - index - 1;
    if (numMoved > 0)
        System.arraycopy(elementData, index+1, elementData, index,
                         numMoved);
    elementData[--size] = null; // clear to let GC do its work

    return oldValue;
}
public boolean remove(Object o) {
	if (o == null) {
		for (int index = 0; index < size; index++)
			if (elementData[index] == null) {
				fastRemove(index);
				return true;
			}
	} else {
		for (int index = 0; index < size; index++)
			if (o.equals(elementData[index])) {
				fastRemove(index);
				return true;
			}
	}
	return false;
}

private void fastRemove(int index) {
	modCount++;
	int numMoved = size - index - 1;
	if (numMoved > 0)
		System.arraycopy(elementData, index+1, elementData, index,numMoved);
	elementData[--size] = null; // clear to let GC do its work
}
```


### 什么是 fail fast？（重要）
- 在用迭代器遍历一个集合对象时，如果遍历过程中对集合对象的内容进行了修改（增加、删除、修
改），则会抛出Concurrent Modification Exception。
- 原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个 modCount 变量。集合在被遍历期间如果内容发生变化，就会改变modCount的值。每当迭代器使用 **`hashNext()/next()`** 遍历下一个元素之前，**`都会检测modCount变量是否为expectedmodCount值`**，是的话就返回遍历；否则抛出异常，终止遍历。
- 注意：这里异常的抛出条件是检测到 **`modCount！=expectedmodCount`** 这个条件。**`如果集合发生变化时修改modCount值刚好又设置为了expectedmodCount值，则异常不会抛出`**。因此，不能依赖于这个异常是否抛出而进行并发操作的编程，这个异常只建议用于检测并发修改的bug。
- 场景：java.util包下的集合类都是快速失败的，不能在多线程下发生并发修改（迭代过程中被修改），比如HashMap、ArrayList 这些集合类。
> 解决方法：

- 使用Colletions.synchronizedList()方法或在修改集合内容的地方加上synchronized。这样的话，增删集合内容的同步锁会阻塞遍历操作，影响性能。
- 使用CopyOnWriteArrayList来替换ArrayList。在对CopyOnWriteArrayList进行修改操作的时候，会拷贝一个新的数组，对新的数组进行操作，操作完成后再把引用移到新的数组。
### 什么是fail safe？（重要）

采用安全失败机制的集合容器，**在遍历时不是直接在集合内容上访问的，而是先复制原有集合内容，在拷贝的集合上进行遍历**。 **`java.util.concurrent`** 包下的容器都是安全失败，可以在多线程下并发使用，并发修改。

**原理**：由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所作的修改并不能被迭代器检测到，所以不会触发Concurrent Modification Exception。

**缺点**：基于拷贝内容的优点是避免了Concurrent Modification Exception，但同样地，迭代器并不能访问到修改后的内容，即：迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。
### 怎么在遍历 ArrayList 时移除一个元素？

foreach删除会导致 **`快速失败问题`** ，可以使用迭代器的 remove() 方法。

```java
Iterator itr = list.iterator();
while(itr.hasNext()) {
      if(itr.next().equals("jay") {
        itr.remove();
      }
}
```



## HashMap
### HashMap是怎么实现的？
 - HashMap是线程不安全的，key、value均可以为null。
 - jdk1.8 之前 HashMap 由 **数组 + 链表** 组成，**数组是 HashMap 的主体，链表则是主要为了解决哈希冲突**
 - jdk1.8 以后在解决哈希冲突时有了较大的变化，**当链表长度大于阈值（转为红黑树的边界值，默认为 8 ）并且当前数组的长度大于64时（`必须同时满足两个条件`） ，此时此索引位置上的所有数据改为使用红黑树存储。**
 - 补充：将链表转换成红黑树前会判断，即便**阈值大于8，但是数组长度小于64，此时并不会将链表变为红黑树，而是选择逬行数组扩容**。
### 你对红黑树了解多少？为什么不用二叉树/平衡树呢？
红黑树本质上是一种二叉查找树，为了保持平衡，它又在二叉查找树的基础上增加了一些规则：
- 每个节点要么是红色，要么是黑色；
- 根节点永远是黑色的；
- 所有的叶子节点都是是黑色的（**`注意这里说叶子节点其实是图中的 NULL 节点`**）；
- 每个红色节点的两个子节点一定都是黑色；
- **`从任一节点到其子树中每个叶子节点的路径都包含相同数量的黑色节点；`**
> 之所以不用二叉树：

红黑树是一种平衡的二叉树，插入、删除、查找的最坏时间复杂度都为 O(logn)，避免了二叉树最坏情况下的O(n)时间复杂度。
> 之所以不用平衡二叉树：

平衡二叉树是比红黑树更严格的平衡树，为了保持保持平衡，需要旋转的次数更多，也就是说平衡二叉树保持平衡的效率更低，所以平衡二叉树插入和删除的效率比红黑树要低。
### 红黑树怎么保持平衡的知道吗？TODO
红黑树有两种方式保持平衡：旋转和染色。

- 旋转：旋转分为两种，左旋和右旋
### HashMap的put流程知道吗？（重要）
- 1.首先进行哈希值的计算，获取一个新的哈希值。
	```java
	(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
	```
- 2.如果table没有初始化就先进行初始化过程；
- 3.根据哈希值计算数组下标，如果对应下标正好没有存放数据，则直接构建节点插入，否则就是出现碰撞冲突了，则需要处理冲突。
	```java
	tab[i = (n - 1) & hash])
	```
	- 判断tab[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点；
	- 否则采用传统的链式方法插入。如果链表中插入节点的时候，链表长度大于等于8，会调用treeifyBin()方法处理。**`在该方法中会判断数组长度是否小于64，如果大于64才转换为红黑树，否则依旧进行扩容`** 。
		```java
		treeifyBin(tab, hash);
		```
- 4.最后所有元素处理完成后，判断是否超过阈值；**`threshold`** ，超过则扩容。
### HashMap中是如何判断key是否相等的呢？（重要 重要 重要）
- 通俗来说，**`Java中的hashCode方法就是根据一定的规则将与对象相关的信息（比如对象的存储地址，对象的字段等）映射成一个数值，这个数值称作为散列值`**。即在散列集合包括HashSet、HashMap以及HashTable里，对每一个存储的桶元素都有一个唯一的"块编号"，即它在集合里面的存储地址；当你调用contains方法的时候，它会根据hashcode找到块的存储地址从而定位到该桶元素。
- 那为什么还需要equals()?该方法是用来判断两个对象内存地址是否相同的方法，如果该方法被重写了，则依据其具体重写的内容来判断其具体功能，一般重写之后变成判断两个对象的值是否相同。
- HashMap中是如何判断两个Key是否相同？
	- **`p.hash == hash`** ：**`比较两个hash值是否相等`**。
	- **`(k = p.key) == key`** ：**`比较两个key的地址值是否相等`**。
	- **`key != null && key.equals(k)`**：**`能够执行到这里说明两个key的地址值不相等，那么先判断后添加的key是否等于null，如果不等于null再调用equals方法判断两个key的内容是否相等`**。

	```java
	if (p.hash == hash &&
	            ((k = p.key) == key || (key != null && key.equals(k))))
	```
- hashCode() 和 equals() 的区别？
	- 如果两个对象的hashcode()返回值一样，其equals()返回结果不一定一样。
	- 如果两个对象的hashcode()返回值不一样，其equals()返回结果一定不一样。
	- 如果两个对象的equals()返回值一样，其hashcode()返回结果一定一样。
	- 如果两个对象的equals()返回值不一样，其hashcode()返回结果不一定不一样。
### 为什么HashMap的容量是2的幂次方呢？
Hash 值的范围值比较大，使用之前需要先对数组的长度取模运算，得到的余数才是元素存放的位置也就是对应的数组下标。这个数组下标的计算方法是(n - 1) & hash。

**`将HashMap的长度定为2 的幂次方，这样就可以使用(n - 1)&hash位运算代替%取余的操作，提高性能。`**
- **`(n - 1) & hash`**：根据hash值计算索引值，在n为2的整次幂的时候相当于取模。
### 如果初始化HashMap，传一个17的值（不是2的整次幂）new HashMap<>，它会怎么处理？
传的不是2的倍数时，HashMap会向上寻找离得最近的2的倍数，所以传入17，但HashMap的实际容量是32。

源码中是通过一系列 **`右移 + 按位或`** 实现的。

### HashMap的 哈希/扰动 函数是怎么设计的?如何计算数组（索引）下标？为什么这样设计？
-  hash()函数 是先拿到 key 的hashcode，是一个32位的int类型的数值，然后让hashcode的高16位和低16位进行异或操作。
	```java
	static final int hash(Object key) {
	    int h;
	    // key的hashCode和key的hashCode右移16位做异或运算
	    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
	}
	```


![计算过程](https://img-blog.csdnimg.cn/20200628183634565.png#pic_center)
> 为什么这样设计？

如果当 n 即数组长度很小，假设是 16 的话，那么 n - 1 即为 1111 ，这样的值和 hashCode 直接做按位与操作，**`实际上只使用了哈希值的后 4 位。`** **`如果当哈希值的高位变化很大，低位变化很小，这样就很容易造成哈希冲突了，所以这里把高低位都利用起来，从而解决了这个问题。`**

### 为什么HashMap链表转红黑树的阈值为8呢？为什么HashMap红黑树转链表阈值为6呢?
红黑树节点的大小大概是普通节点大小的两倍，所以转红黑树，牺牲了空间换时间，更多的是一种兜底的策略，保证极端情况下的查找效率。（空间换时间）

阈值为什么要选8呢？和统计学有关。理想情况下，使用随机哈希码，链表里的节点符合泊松分布，出现节点个数的概率是递减的，节点个数为8的情况，发生概率仅为0.00000006。

至于红黑树转回链表的阈值为什么是6，而不是8？是因为如果这个阈值也设置成8，假如发生碰撞，节点增减刚好在8附近，会发生链表和红黑树的不断转换，导致资源浪费。
### HashMap怎么查找元素的呢？
HashMap的查找就简单很多：

1. 使用hash()函数计算key的哈希值；
2. 计算数组下标，获取节点；
3. 当前节点和key匹配，直接返回；
4. 否则，当前节点是否为树节点，查找红黑树；
5. 否则，遍历链表查找
### HashMap在什么时候需要会扩容？
1. 当 HashMap 中的元素个数超过 **`临界值（threshold ）= 数组大小(数组长度) * loadFactor(负载因子)`** 时，就会进行数组扩容，loadFactor 的默认值是 0.75。默认为 16 * 0.75 = 12。
2. 当链表长度大于8，且数组长度小于64时会进行扩容。

### 为什么扩容因子是0.75？

- 假如设的比较大，元素比较多，空位比较少的时候才扩容，那么发生哈希冲突的概率就增加了，查找的 **`时间成本就增加了`** 。
- 假如设的比较小，元素比较少，空位比较多的时候就扩容了，发生哈希碰撞的概率就降低了，查找时间成本降低，但是就需要更多的空间去存储元素，**`空间成本就增加了`** 。
### 详细说说HashMap的扩容机制？它是如何计算新下标的？

以JDK1.8为例，当往HashMap放入元素时，如果元素个数大于threshold时，会进行扩容，使用2倍容量的数组代替原有数组。表现在二进制上就是多了一个高位参与数组下标计算。

也就是说，在元素拷贝过程不需要重新计算元素在数组中的位置，只需要看看原来的hash值新增的那个bit是1还是0，是0的话索引没变，是1的话索引变成“原索引+oldCap”（根据e.hash & oldCap  == 0判断）。

因此，我们在扩充 HashMap 的时候，**`不需要重新计算 hash 值`** ，只需要用 **`原来的 hash 值 和 原数组扩容前长度进行 与操作（&）`**，然后 **`看高位的那个 bit 是 1 还是 0 就可以了`** ，**`是 0 的话索引没变，是 1 的话索引变成 “原位置 + 旧容量” 。`**
```java
if ((e.hash & oldCap) == 0) {
	// 在 原位置
} else {
	// 在 原位置 + oldCap 的位置
}
```

优点：
- 省去了重新计算 hash 值的时间。
- 由于新增的 1bit 是 0 还是 1 可以认为 **`是随机的`** ，在 resize 的过程中保证了 rehash 之后每个桶上的结点数一定小于等于原来桶上的结点数，保证了 rehash 之后不会出现更严重的 hash 冲突，均匀的把之前的冲突的结点分散到新的桶中了。

### 在解决 hash 冲突的时候，为什么选择先用链表，再转红黑树?

- 因为红黑树需要进行左旋，右旋，变色这些操作来保持平衡，而单链表不需要。
- 当元素小于 8 个的时候，链表结构可以保证查询性能。
- 当元素大于 8 个的时候， **`红黑树搜索时间复杂度是 O(logn)，而链表是 O(n)`** ，此时需要红黑树来 **`加快查询速度`** ，**`但是插入和删除节点的效率变慢了`** 。
### HashMap 是线程安全的吗？多线程下会有什么问题？
HashMap不是线程安全的，可能会发生这些问题：
- **`多线程下扩容死循环`** 。JDK1.7中的 HashMap 使用头插法插入元素，在多线程的环境下，扩容的时候有可能导致环形链表的出现，形成死循环。JDK1.8 使用尾插法插入元素，在扩容时会保持链表元素原本的顺序，不会出现环形链表的问题。
- **`多线程的 put 可能导致元素的丢失`** 。多线程同时执行 put 操作，如果计算出来的索引位置是相同的，那会造成前一个 key 被后一个 key 覆盖，从而导致元素的丢失。此问题在 JDK 1.7 和 JDK 1.8 中都存在。
- **`put 和 get 并发时，可能导致 get 为 null`** 。线程 1 执行 put 时，因为元素个数超出 threshold 而导致 rehash，线程 2 此时执行 get，有可能导致这个问题。这个问题在 JDK 1.7 和 JDK 1.8 中都存在。
### 有什么办法能解决HashMap线程不安全的问题呢？
Java 中有 HashTable、Collections.synchronizedMap、以及 ConcurrentHashMap 可以实现线程安全的 Map。
- HashTable 是 **`直接在操作方法上加 synchronized 关键字，锁住整个table数组`** ，粒度比较大；
- Collections.synchronizedMap 是使用 Collections 集合工具的内部类，通过传入 Map 封装出一个 SynchronizedMap 对象，内部定义了一个对象锁，方法内通过对象锁实现；
- ConcurrentHashMap 在jdk1.7中使用 **`分段锁`** ，在jdk1.8中使用 **`CAS+synchronized`** 。
### jdk1.8对HashMap主要做了哪些优化呢？为什么？（总结，不仅仅局限于数据结构）
- **`数据结构`** ：数组 + 链表改成了数组 + 链表或红黑树
	- 原因：发生 hash 冲突，元素会存入链表，链表过长转为红黑树，将时间复杂度由O(n)降为O(logn)
- **`链表插入方式`** ：链表的插入方式从头插法改成了尾插法
	- 原因：因为 1.7 头插法扩容时，头插法会使链表发生反转，多线程环境下会产生环。
- **`扩容rehash`**
	- 扩容的时候 1.7 需要对原数组中的元素进行重新 hash 定位在新数组的位置，1.8 采用更简单的判断逻辑，不需要重新通过哈希函数计算位置，新的位置不变或索引 + 新增容量大小。
	- 原因：提高扩容的效率，更快地扩容。
- 扩容时机：在插入时，1.7 先判断是否需要扩容，再插入，1.8 先进行插入，插入完成再判断是否需要扩容；
## ConcurrentHashMap
### 说说ConcurrentHashMap怎么实现的？
- jdk1.7版本是基于 **`分段锁`** 实现。
- jdk1.8是基于 **`CAS + synchronized`** 实现。
### jdk1.7中的分段锁
从结构上说，1.7版本的ConcurrentHashMap采用分段锁机制， **`里面包含一个Segment数组`** ，Segment继承于 **`ReentrantLock`** ，Segment则包含 **`HashEntry的数组`** ， **`HashEntry本身就是一个链表的结构`** ，具有保存key、value的能力，能指向下一个节点的指针。

**`实际上就是相当于每个Segment都是一个HashMap，默认的Segment长度是16，也就是支持16个线程的并发写，Segment之间相互不会受到影响。`**
![在这里插入图片描述](https://img-blog.csdnimg.cn/f317828816fd4dcc97a7a1f8114aa91d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5Liq5bCP56CB5Yac55qE6L-b6Zi25LmL5peF,size_20,color_FFFFFF,t_70,g_se,x_16)
### jdk1.7的put流程
1. 计算hash，定位到segment，segment如果是空就先初始化
2. 使用ReentrantLock加锁，如果获取锁失败则尝试自旋，自旋超过次数就阻塞获取，保证一定获取锁成功
3. 遍历HashEntry，就是和HashMap一样，数组中key和hash一样就直接替换，不存在就再插入链表，链表同样操作

### jdk1.7的get流程
get也很简单，key通过hash定位到segment，再遍历链表定位到具体的元素上，需要注意的是value是volatile的，所以get是不需要加锁的。
### jdk1.8 CAS+synchronized
jdk1.8实现线程安全不是在数据结构上下功夫，它的数据结构和HashMap是一样的，数组+链表+红黑树。它实现线程安全的 **`关键点在于put流程`** 。
### jdk1.8的put流程 （重要）
1. 首先根据key计算hash值；
2. 如果node数组为空，先进行初始化；
3. 遍历node数组，拿到首节点f，判断首节点：
	- [ ] 如果为 null ，则通过cas的方式尝试添加。
	- [ ] 如果为 f.hash = MOVED = -1 ，说明其他线程在扩容，参与一起扩容。
	- [ ] 如果都不满足 ，**`synchronized 锁住 f 节点`**，判断是链表还是红黑树，遍历插入。
4. 当在链表长度达到8的时候，数组扩容或者将链表转换为红黑树。
### ConcurrentHashMap 的 get 方法是否要加锁，为什么？重要
- get 方法不需要加锁。
- **`因为 Node 的元素 val 和指针 next 是用 volatile 修饰的`**，在多线程环境下线程A修改结点的val或者新增节点的时候是对线程B可见的。
- 这也是它比其他并发集合比如 Hashtable、用 Collections.synchronizedMap()包装的 HashMap 安全效
率高的原因之一。

```java
static class Node<K,V> implements Map.Entry<K,V> {
	final int hash;
	final K key;
	//可以看到这些都用了volatile修饰
	volatile V val;
	volatile Node<K,V> next;
}
```

### get方法不需要加锁与volatile修饰的哈希桶有关吗？

没有关系。哈希桶table 用volatile修饰主要是保证在数组扩容的时候保证可见性。

### ConcurrentHashMap 不支持 key 或者 value 为null 的原因？
- 我们先来说value 为什么不能为 null ，因为ConcurrentHashMap 是用于多线程的 ，**`如果map.get(key) 得到了 null ，无法判断，是映射的value是 null ，还是没有找到对应的key而为 null ，这就有了二义性`**。
- 而用于单线程状态的HashMap 却可以用containsKey(key) 去判断到底是否包含了这个 null 。

我们用反证法来推理：
- 假设ConcurrentHashMap 允许存放值为 null 的value，这时有A、B两个线程，线程A调用ConcurrentHashMap .get(key)方法，返回为 null ，我们不知道这个 null 是没有映射的 null ，还是存的值就是 null 。
- **`假设此时，返回为 null 的真实情况是没有找到对应的key`**。那么，我们可以用ConcurrentHashMap
.containsKey(key)来验证我们的假设是否成立，我们期望的结果是返回false。
- 但是在我们调用ConcurrentHashMap .get(key)方法之后，containsKey方法之前，线程B执行了
ConcurrentHashMap .put(key, null )的操作。那么我们调用containsKey方法返回的就是true了，这就与我们的假设的真实情况不符合了，这就有了二义性。
### ConcurrentHashMap 的并发度是多少？
- 在JDK1.7中，并发度默认是16，这个值可以在构造函数中设置。
- 如果自己设置了并发度，ConcurrentHashMap 会使用大于等于该值的最小的2的幂指数作为实际并发度，也就是比如你设置的值是17，那么实际并发度是32。

### ConcurrentHashMap 和Hashtable的效率哪个更高？为什么？
ConcurrentHashMap 的效率要高于Hashtable，因为Hashtable给整个哈希表加了一把大锁从而实现线
程安全。而ConcurrentHashMap 的锁粒度更低，在JDK1.7中采用分段锁实现线程安全，在JDK1.8 中采
用CAS+Synchronized 实现线程安全。
