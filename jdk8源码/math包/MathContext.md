# RoundingMode

roundingMode：**舍入/近似模式，为可能丢弃精度的小数操作指定一种舍入行为**；

对于十进制10/3，在数学中，结果为无限循环小数，但在计算机不能保存无限循环的小数，那么在计算机中要近似成那样呢？

在Java中，近似模式被封装在RoundingMode枚举类中，具体如下：

|     UP      |                        远离零方向舍入                        |
| :---------: | :----------------------------------------------------------: |
|    DOWN     |                         向零方向舍入                         |
|   CEILING   |                       向正无穷方向舍入                       |
|    FLOOR    |                       向负无穷方向舍入                       |
|   HALF_UP   | 向最接近数字方向舍入，如果与两个相邻数字的距离相等，则向上舍入【即四舍五入】 |
|  HALF_DOWN  | 向最接近数字方向舍入，如果与两个相邻数字的距离相等，则向下舍入 |
|  HALF_EVEN  | 向最接近数字方向舍入，如果与两个相邻数字的距离相等，则向相邻的偶数舍入 |
| UNNECESSARY |           请求的操作具有精确的结果，不需要进行舍入           |

这个枚举在jdk中，用来替代BigDecimal中的舍入模式常量；



# MathContext

RoundingMode仅仅描述了舍入的规则，但还有一些其它的规则，例如保留几位有效数字？

MathContext则是针对于计算的更进一步抽象，是封装上下文设置的不可变对象，描述了数字运算的某些规则；

该类封装了数学运算上下文，提供了用于执行任意精度整数算法（BigInteger）和任意精度小数算法（BigDecimal）的类；



precision：是指的整个数字精确后的长度，而非理解的scale。值 0 表示将使用无限精度（所需的位数）；

例如：`00100.032`在DECIMAL32模式下为`100.0320`【结果保留7位数字】，而在DECIMAL64模式下为`100.0320000000000`【结果保留16位数字】；



了解字符串是如何创建MathContext对象的，通过阅读后可以了解到构造参数val必须符合某个格式，否则就会抛出运行时异常【RuntimeException】。

```java
public MathContext(String val) {
      boolean bad = false;
      int setPrecision;
      if (val == null)
          throw new NullPointerException("null String");
      try { // any error here is a string format problem
          if (!val.startsWith("precision=")) throw new RuntimeException();
          int fence = val.indexOf(' ');    // could be -1
          int off = 10;                     // where value starts
          setPrecision = Integer.parseInt(val.substring(10, fence));

          if (!val.startsWith("roundingMode=", fence+1))
              throw new RuntimeException();
          off = fence + 1 + 13;
          String str = val.substring(off, val.length());
          roundingMode = RoundingMode.valueOf(str);
      } catch (RuntimeException re) {
          throw new IllegalArgumentException("bad string format");
      }

      if (setPrecision < MIN_DIGITS)
          throw new IllegalArgumentException("Digits < 0");
      // the other parameters cannot be invalid if we got here
      precision = setPrecision;
  }
```

val格式：`precision=32 roundingMode=UP`，中间用空格做分隔符，与toString方法生成字符串的格式相同；



共享常量：

> MathContext.DECIMAL128：其精度设置与 IEEE 754R Decimal128 格式（即 34 个数字）匹配，舍入模式为HALF_EVEN，这是 IEEE 754R 的默认舍入模式；
>
> MathContext.DECIMAL64：其精度设置与 IEEE 754R Decimal64 格式（即 16 个数字）匹配，舍入模式为HALF_EVEN，这是 IEEE 754R 的默认舍入模式；
>
> MathContext.DECIMAL32：其精度设置与 IEEE 754R Decimal32 格式（即 7 个数字）匹配，舍入模式为HALF_EVEN，这是 IEEE 754R 的默认舍入模式；
>
> MathContext.UNLIMITED：其设置具有无限精度算法所需值的 `MathContext` 对象。



总结：MathContext主要用在BigInteger和BigDecimal中，当BigDecimal除不尽时，就会抛出异常【计算机无法保存】，此时通过MathContext控制保留的位数和小数点的舍入规则将不会抛出异常。

```java
BigDecimal d1 = new BigDecimal("10").divide(new BigDecimal("3")); //runtime exception
BigDecimal d2 = new BigDecimal("10").divide(new BigDecimal("3"), MathContext.DECIMAL32); //pass
```



