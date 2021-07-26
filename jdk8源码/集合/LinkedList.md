# LinkedList

## 概述

为什么需要引入LinkedList？

我们知道ArrayList的插入和删除的需要移动元素，时间最坏复杂度为O(N)。ArrayList不适合频繁的插入和删除元素，因此引入了链表结构。

LinkedList底层数据结构是一个双向链表。和单向链表相比，单向链表不能往回查找元素，而双向链表可以。



单链表的结构如下：

![image-20210711172825403](asserts/image-20210711172825403.png)

我们主要看双链表结构：

![image-20210711173227128](asserts/image-20210711173227128.png)

从结构中我们可以看出，链表中的每个节点都可以向前或向后查找元素，其中有几个相关概念：

- 链表每个节点我们叫做为Node，Node有prev属性，代表前一个节点的位置；next属性，代表后一个节点的位置；
- first是双向链表的头节点，它的前一个节点是null；
- last是双向链表的尾节点，它的后一个节点是null；
- 当链表中没有数据时，first和last是同一个节点，前后指向都是null；
- 因为是个双向链表，只要机器内存足够强大，是没有大小限制的；

```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
    Node(Node<E> prev, E element, Node<E> next) {
        this.item = element;
        this.next = next;
        this.prev = prev;
    }
}
```



LinkedList的继承树

![image-20210711180541079](asserts/image-20210711180541079.png)

- LinkedList继承了AbstractSequentialList抽象类；
- LinkedList实现Queue接口，表明LinkedList符合队列结构；
- LinkedList实现Deque接口，表明LinkedList符合双端队列结构；

为什么需要新添加一个AbstractSequentialList抽象类，而不直接使用AbstractList?

首先，我们需要知道ArrayList、LinkedList的数据结构，数组是支持随机访问的。而链表结构中的内存不是按顺序存放的，而是将引用连在了一起，为了访问某一元素，必须顺着链表的引用逐个找。



## CRUD方法

### 添加

在LinkedList中，支持从头节点前添加一个节点【头插法】、尾节点后添加一个节点【尾插法】、某个节点【非头/非尾】前插入一个节点。

#### linkLast

从链表的尾节点后添加一个节点。

```java
void linkLast(E e) {
    //把尾节点数据据暂时保存下来
    final Node<E> l = last;
    //新建一个节点  l:新节点的前一个节点,e当前新增节点,当前新增节点后一个节点是null
    final Node<E> newNode = new Node<>(l, e, null);
    //新建节点作为尾节点
    last = newNode;
    //如果链表为空(尾节点为空,链表即空表),此时头节点和尾节点都是同一个节点
    if (l == null)
        first = newNode;//新建节点作为头节点
    //否则把尾节点的下一个节点指向当前尾节点.
    else
        l.next = newNode;
    size++;
    modCount++;
}
```

首次添加元素前，头节点和尾节点都为空，添加完后，头节点和尾节点都是同一个元素。结构如下：

![image-20210711182448155](asserts/image-20210711182448155.png)

但有多个节点时，再添加一个节点。结构如下：

![image-20210711182918441](asserts/image-20210711182918441.png)

保留尾节点【last】的作用主要有两点：

1. 判断链表是否为空表；
2. 支持从后往前查找元素；

#### linkFirst

从链表的头节点前添加一个元素。

```java
private void linkFirst(E e) {
    //将链表中的头节点保存下来
    final Node<E> f = first;
    //创建新节点,新节点的next节点就是保存下来的头节点
    final Node<E> newNode = new Node<>(null, e, f);
    //将新节点做为头节点
    first = newNode;
    //如果没有头节点,当前节点即为头节点,也为尾节点.
    if (f == null)
        last = newNode;
    else
        f.prev = newNode;//修改保存下来头节点的prev节点
    //更新节点个数及版本
    size++;
    modCount++;
}
```

该方法可实现队列的操作，先进先出。

首次添加元素前结构与linkLast一致。linkFirst和linkLast添加节点非常类似，前者是更换上一个节点(last:0x01)的prev指向，后者是更换上一个节点(first:0x01)的next指向。结构图如下：

![image-20210725122708745](asserts/image-20210725122708745.png)

#### linkBefore

在链表中某个节点前插入一个节点。

```java
void linkBefore(E e, Node<E> succ) {
  final Node<E> pred = succ.prev;
  final Node<E> newNode = new Node<> (pred, e, succ);
  succ.prev = newNode;
  if (pred == null) {
    first = newNode;
  } else {
    pred.next = newNode;
  }
  size++;
  modCount++;
}
```

示意图如下：

![image-20210725124100746](asserts/image-20210725124100746.png)

### 删除

在链表中，删除一个节点也存在多种方式，分别支持删除头节点、尾节点、中间节点。

#### unlinkLast

```java
private E unlinkLast(Node<E> l) {
  final E element = l.item;
  final Node<E> prev = l.prev;
  l.item = null;
  l.prev = null;
  last = prev;
  if (prev == null) {
    first = null;
  } else {
    prev.next = null;
  }
  size--;
  modCount++;
  return element;
}
```

删除尾节点。

#### unlinkFirst

```java
private E unlinkFirst(Node<E> f) {
  final E element = f.item;
  final Node<E> next = f.next;
  f.item = null;
  f.next = null;
  first = next;
  if (next == null) {
    last = null;
  } else {
    next.prev = null;
  }
  size--;
  modCount++;
  return element;
}
```

删除头节点。



#### unlink

```java
E unlink(Node<E> x) {
	//拿到节点中的元素
  final E element = x.item;
  //拿到要删除节点的下一个节点
  final Node<E> next = x.next;
  //拿到要删除节点的上一个节点
  final Node<E> prev = x.prev;
  //如果要删除节点的上一个节点为null,则将删除节点的下一个节点作为头节点
  if (prev == null) {
      first = next;
  } else {
    	//将删除节点的上一个节点重新指向,指向下一个节点,并将要删除节点的prev指向null
      prev.next = next;
      x.prev = null;
  }
  if (next == null) {
    	//如果要删除节点的下一个节点为空,则将要删除节点的上一个节点作为尾节点
      last = prev;
  } else {
    	//修改要删除节点的下一个节点指向
      next.prev = prev;
      x.next = null;
  }
  //将要删除的节点元素置为null,链表长度减一,修改加1,返回之前要删除的节点元素
  x.item = null;
  size--;
  modCount++;
  return element;
}
```

删除中间节点中的。



### 查找

#### node

```java
Node<E> node(int index) {
  if (index < (size >> 1)) {
    Node<E> x = first;
    for (int i = 0; i < index; i++) {
      x = x.next;
      return x;
    } else {
      Node<E> x = last;
      for (int i = size - 1; i > index; i--) {
        x = x.prev;
      }
      return x;
    }
  }
}
```

根据索引查找某个节点，通常是需要从头开始遍历到索引位置，但是LinkedList实现方式更加高效，也由于双向链表的特性，能够存储当前节点的前一个节点。当要查找的位置大于总个数长度的一半时，从结束位置向前查。小于时，才会从头往后查。

与此同时，LinkedList将first和last节点同时保存下来，使得能够直接拿到尾节点，如果只保存了其中一个节点，势必需要从一端到索引位置。

#### indexOf

```java
public int indexOf(Object o) {
  int index = 0;
  if (o == null) {
    for (Node<E> x = first; x != null; x = x.next) {
      if (x.item = null) {
        return index;
      }
      index++;
    }
  } else {
    for (Node<E> x = first; x != null; x = x.next) {
      if (o.equals(x.item)) {
        return index;
      }
      index++;
    }
  }
  return -1;
}
```

在双向链表中从前往后查找某个元素，返回该元素的索引位置，不存在返回-1；在jdk8中可优化代码量，

```java
public int indexOf(Object o) {
  int index = 0;
  for (Node<E> x = first; x != null; x = x.next) {
      if (Objects.equals(o, x.item)) {
        return index;
      }
      index++;
    }
  return -1;
}
```

### 修改

```java 
public E set(int index, E element) {
  checkElementIndex(index);
  Node<E> x = node(index);
  E oldVal = x.item;
  x.item = element;
  return oldVal;
}
```

修改某个索引中的值，返回旧值。由于链表无法通过索引快速定位到，因此需要通过某个查找方法查到该位置的节点。



## 充当队列



