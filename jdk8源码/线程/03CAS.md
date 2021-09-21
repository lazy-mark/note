# CAS

针对单个变量的操作，使用CAS可以保证原子性，CAS是相比Synchronized同步锁来说，是一种乐观锁，CAS更加高效，是基于比较并交换的算法来解决线程冲突。

## CAS的定义和使用

CAS 是 Compare And Swap 的缩写。在JUC包中使用非常普遍，对一个变量进行CAS操作，需要传入三个参数：

- 内存位置
- 预期值
- 新值

```java
compareAndSwap(long memoryAddress, long expect, long newValue);
```

假如对Long类型进行CAS操作，如果内存位置的值和预期值相等，那么处理器会自动将该位置的值更新为新值。否则，处理器不做任何操作。



CAS操作是利用了处理器提供的`CPMXCHG`指令实现的，CAS操作有可能成功，也可能失败，在失败后继续做CAS操作，直到成功为止，这种操作也叫自旋CAS。CAS的底层指令有效避免了并发冲突，将`读取后写入`转化为原子操作，规避了同步锁带来的开销。



CAS底层借助Unsafe类实现的，Unsafe类通过本地方法调用，使得CAS算法可以在Java程序中使用。CAS在Atomic类应用较多，我们一般不会直接调用Unsafe操作CAS，而是通过AtomicXXX原子类来实现CAS。



### AtomicInteger源码

自增操作

```java
private static final jdk.internal.misc.Unsafe U = jdk.internal.misc.Unsafe.getUnsafe();
public final int incrementAndGet() {
    return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
}
```

valueOffset是AtomicInteger内置value的内存地址，通过Unsafe类的jni方法objectFieldOffset来获取。

```java
private static final long valueOffset;
static {
    try {
        valueOffset = unsafe.objectFieldOffset
            (AtomicInteger.class.getDeclaredField("value"));
    } catch (Exception ex) { throw new Error(ex); }
}
```

unsafe的getAndAddInt方法

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

compareAndSwapInt就是一个CAS操作，getIntVolatile获取每次最新的值，然后将预期值var5和更新值var5+var4传人compareAndSwapInt，其中var1和var2组合起来作为地址参数，当执行CAS操作compareAndSwapInt失败后，说明其他线程也在做CAS操作，当前线程会继续执行，直到成功为止。

## CAS面临的问题

CAS能实现单一变量的原子性，但它也有三个问题；

- ABA问题

  CAS 操作是检查值没有发生变化则更新，如果一个值初始值是 A，之后变成 B，又变成 A, 使用 CAS 做检查时是不会发生变化的，CAS 操作依然是成功的，当然 ABA 在绝大多数场景是可以容忍的，只有少数场景在比如保险箱中的被盗又被还了回来，说明保险箱已经不够安全了。可以通过AtomicStampedReference解决，对变量前面加上版本号，每次更新版本+1；

- 自旋消耗CPU

  CAS 失败会重试，在多线程同时对同一变量做频繁操作时，CAS 失败的概率会增加，循环重试次数也会随着增多，导致 CPU 消耗增加；

- 只能保证单一共享变量的原子性

  CAS 操作无法保证多个变量的原子性，如果涉及到多个共享变量，需要使用锁来实现原子操作；



## CAS的底层Unsafe

CAS操作是通过Unsafe类来实现的，位于sun.misc包下，实现了内存操作、线程调度、对象操作和数组操作。



内存操作：分配内存、拷贝内存、释放内存；

线程调度：线程挂起、线程恢复；

对象操作：获取对象内存地址；

数组操作：数组内存大小、数据地址偏移量；

系统操作：返回指针大小、返回页大小；

Class操作：定义类、定义匿名类、检查类是否初始化；

CAS操作：cas整数型、cas长整型、cas对象类型；



由于该类默认是 BootstrapClassLoader 加载的，虽然Unsafe提供了 `getUnsafe` 方法，但是getUnsafe方法内部会对调用来源类做校验，来源类也必须是由BootstrapClassLoader或ExtClassLoader加载，如果尝试获取抛出SecurityException异常，但用户代码使用的类加载器大部分是AppClassLoader，如果需要在代码中获取Unsafe实例，只能通过反射方式；

```java
@CallerSensitive
public static Unsafe getUnsafe() {
    Class<?> caller = Reflection.getCallerClass();
    if (!VM.isSystemDomainLoader(caller.getClassLoader()))
        throw new SecurityException("Unsafe");
    return theUnsafe;
}
```

在JDK9中，扩展类加载器被平台类加载器取代。反射获取Unsafe的代码如下：

```java
try {
      Field field = Unsafe.class.getDeclaredField("theUnsafe");
      field.setAccessible(true);
      Unsafe unsafe = (Unsafe) field.get(null);
      System.out.println(unsafe);
  } catch (NoSuchFieldException | IllegalAccessException e) {
      e.printStackTrace();
  }
```



