---
title: equals与hashCode之间的联系
date: 2018-07-12 14:07:51
tags: equals
categories: 
- JAVA
- SE
---

equals和hashCode都是Object提供的两个方法，equals()方法用于判断两个对象是否相等，hashCode()方法用于计算对象的哈希码，两个方法都不是final方法，可以被重写（OverWrite）

<!-- more -->

#### equals()方法

- Object类中的equals方法：

~~~java
public boolean equals(Object obj){
  return(this == obj);
}
~~~

可以看出Object类中的equals方法内部也就是比较两个对象的地址值，即两个对象是否为同一个对象，如果不是同一个，就返回false。

  虽然我们在定义类时，可以重写equals方法，但是重写该方法有一些必须要遵守的约定：

1. 自反性：x.equals(x)必须返回true
2. 对称性：x.equals(y)与y.equals(x)的返回值必须相等
3. 传递性：x.equals(y)为true，y.equals(z)为true，那么x.equals(z)必须也为true
4. 一致性：如果对象x和y在equals()中使用的信息都没有改变，那么x.equals(y)的值始终不变
5. 非null：x不是null,y为null，则x.equals(y)必须为false（y.equals(x)会报空指针异常的错误）




#### hashCode()方法

- Object类中的hashCode()方法：

~~~java
public native int hashCode();
~~~

hashCode()是一个native方法，而且返回值类型是整形；实际上，该native方法将对象在内存中的地址作为哈希码返回，可以保证不同对象的返回值不同。

  重写hashCode（）时，有一些注意事项：

1. hashCode()在哈希表中起作用
2. 如果对象在equals中使用的信息都没有改变，那么hashCode()值也始终不变
3. 如果两个对象使用equals()方法判断为相等，那么hashCode()也应该为相等
4. 如果两个对象使用equals()方法判断为不相等时，那么hashCode()方法不一定不相等，但是一般条件下应该让不同的对象产生不同的hashCode()，可以提高哈希表的性能

- hashCode()的作用

  当我们向哈希表中添加对象a时，一般先通过计算对象a的哈希码以直接定位在哈希表中的存储位置，如果该位置没有对象，那么可以直接将a插入该位置；如果该位置存在对象，那么根据哈希表的冲突解决，可以将对象a与各个冲突的元素比较，调用equals()进行比较，看是否为true，如果为true，那么不需要保存对象a；否则将对象a加入到哈希表中。

  **这里可以看出为什么说equals()相等,hashCode()必须相等**，因为如果你hashCode不相等的话，哈希表会插入到不同的位置，这样会造成一个元素在哈希表中多次出现。

- 默认的重写hashCode()方法使用了数字31

  1. 使用质数计算哈希码，会让它与其他数字相乘后得出的结果唯一性概率较大，这样哈希表冲突的概率就较小。
  2. 使用质数越大，哈希表冲突越小，但是计算的速度可能就会越慢；31是哈希冲突和性能的折中，实际上是实验观测的结果。
  3. JVM会自动对31进行优化：31* i == (i << 5) - i

- 重写hashCode()方法的一些原则

  1. 如果重写了equals()方法，那么重写hashCode()方法中用到的域要与equals方法中重写用到的域相同，例如重写了person类中的equals()方法用到了属性年龄age和姓名name，那么hashCode()重写中尽量也用age和name去计算。

