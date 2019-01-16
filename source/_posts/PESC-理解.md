---
title: PESC 原则理解
date: 2019-01-17 01:46:46
tags: 
- JAVA
- PESC
categories: 
- JAVA
- 泛型
---

# 序

PECS 原则，是在泛型中定义的一种原则。相对应的是泛型中的通配符概念，即 **< ? extends T >** 和 **< ? super T >**

经常被这个概念搞晕，为了加深印象，决定把自己那一瞬间的感悟记起来。

<!-- more -->

## 1、何为 PECS原则

PECS （Producer Extends Customer Super） ，字面上的意思就是 生产者、消费者，而 Extends 和 Super 在 Java 中分别对应的是继承和调用父类的函数。

 **PECS** 原则就是：

1. **频繁往外读取内容的，适合用上界 extends。**
2. **经常往里插入的，适合用下界 super。**



## 2、 < ? extends T >

< ? extends T >  “上界通配符 ”，即类型是 T 或者 T 的子类。< ？extends Number> ，类型可能是 Long ，或者可能是 Integer 。具体是什么类型就不清楚了。

```java
List<? extends Number> e = new ArrayList<>();
e.add(1); // 编译器会提示你 xxx cannot be applied xxx 如下图
```

![1547658004202](https://i.loli.net/2019/01/17/5c3f6d77df394.png)

因为我们只知道类型是 Number及其子类，但具体哪一个不清楚。**所以不能执行添加方法**。

对于读取方法，结果只能存放在 T 或者基类中，如 Object

```java
Number number = e.get(0);
Object obj = e.get(0);
```



## 3、< ? super T >

< ? super T> “下界通配符”，即类型是 T 或者是 T 的父类。 而 Object 是所有类的父类

```java
List<? super Number> s = new ArrayList<>();
s.add(1);
Object o = s.get(0); // 这里是用 Object 去接收拿到的东西
```

因为 Object 是所有类的父类，具有很大的兼容性。或者已知结果类型，用来替代 Object，不过这样就失去了灵活性，因为可能不止一种类型存在。所以用 Object 去接收。不过这样子的话就会丢失原本类型的信息，（本来是 Integer , 用  Object 接收，那 Number 的全部特性都会丢失 。）



## 4、总结

|               | 读取 | 写入 |
| ------------- | :--: | :--: |
| <? extends T> |  √   |  ×   |
| <? super T >  |  ×   |  √   |

<? extends T> 不能插入，但能读取到，并且用 T 或者 T 的基类接收。

<? super T>能插入，但读取的时候只能用 Object 接收，丢失 T 的全部信息。

所以 **PECS** 原则就是：

1. **频繁往外读取内容的，适合用上界 extends。**
2. **经常往里插入的，适合用下界 super。**



附上一段 jdk 源码

```java
/**
 * Collections 源码的 copy(),可以看到 dest 拿来写入，src 拿来读取
 */
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
        int srcSize = src.size();
        if (srcSize > dest.size())
            throw new IndexOutOfBoundsException("Source does not fit in dest");

        if (srcSize < COPY_THRESHOLD ||
            (src instanceof RandomAccess && dest instanceof RandomAccess)) {
            for (int i=0; i<srcSize; i++)
                dest.set(i, src.get(i));
        } else {
            ListIterator<? super T> di=dest.listIterator();
            ListIterator<? extends T> si=src.listIterator();
            for (int i=0; i<srcSize; i++) {
                di.next();
                di.set(si.next());
            }
        }
    }
```

# 完

> 努力成为一个『不那么差』的程序员 

