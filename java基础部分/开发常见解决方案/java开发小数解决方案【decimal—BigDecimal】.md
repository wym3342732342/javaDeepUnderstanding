## 一、java中BigDcimal的使用

&emsp;&emsp;浮点数都会失去一定的精度，java提供`BigDcimal`处理超过16位的数进行精确运算【包含小数】。但其是一个对象，不能使用运算符运算。



### 1.1、BigDecimal转换Double数据

```java
Double num = 123456789.98;
BigDecimal bg1 = new BigDecimal(num);
BigDecimal bg2 = new BigDecimal(num + "");
System.out.pringtln(num);
System.out.pringtln(bg1);
System.out.pringtln(bg2);

/***************************************** 输出结果 ********************************/
1.2345678998E8
123456789.98000000417232513427734375
123456789.98
```

> 所以正确的取Double数据的方法是：`BigDecimal bg2 = new BigDecimal(num + "");`



### 1.2、BigDecimal去掉科学计数法

```java
NumberFormat numberFormat = NumberFormat.getInstance();
numberFormat.setGroupingUsed(false);
System.out.println("d:= " + numberFormat.format(129128391.10));
```



### 1.3、BigDecimal加减乘除

```java
//加：add()
//减：subtract()
//乘：multiply()
//除：divide()
```





## 二、Mysql-decimal

### 2.1、声明语法

```mysql
DECIMAL(M,D)
```

- `M`：数字的最大数（精度）。其范围为1～65（在较旧的MySQL版本中，允许的范围是1～254），默认值是10。
- `D`：小数点右侧数字的数目（标度）。其范围是0～30，但不得超过M

> `decimail(M,D)`占M+2个字节
>
> 也就是说，`decimail(6,1)`占8个字节，最大值`999999.9`



### 2.2 在mysql5.6的小bug

1. 当存储的decimal值超级大的时候，在没有索引的情况下，mysql会认为所有的值一样大。
2. 当使用decimal 做分区key的时候，分区key无法正确过滤分区的问题



