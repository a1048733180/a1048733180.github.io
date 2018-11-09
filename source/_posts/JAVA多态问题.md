---
title: JAVA多态
date: 2018-02-16 16:52:47
tags: JAVA
categories: 
- JAVA
- SE
---



下面是一些JAVA多态问题的整理。

<!--more-->

- 继承

  在多态中必须存在有**继承**关系的的子类和父类

- 重写

  子类对父类中某些方法进行重新定义，在调用方法时就会调用子类的方法

- 向上转型

  在多态中需要将子类的引用赋给父类对象，只有这样该引用才能够具备技能调用父类的方法和子类的方法（继承也可以是实现接口）

- 向下转型

  前提是父类对象指向子类对象，可以理解为向下转型之前，它得先向上转型，向下转型只能转型为本类对象

经典案例

```java
class A{
  public String show(D obj){
    return ("A and D");
  }
  
  public String show(A obj){
    return("A and A");
  }
}

class B extends A{
  public String show(B obj){
    return("B and B");
  }
  
  public String show(A obj){
    return("B and A");
  }
}

class C extends B{}

class D extends B{}

public class Demo{
  public static void main(String[] args){
    A a1 = new A();
    A a2 = new B();
    B b = new B();
    C c = new C();
    D d = new D();
    
    
    System.out.println("1--" + a1.show(b));
    System.out.println("2--" + a1.show(c));
    System.out.println("3--" + a1.show(d));
    System.out.println("4--" + a2.show(b));
    System.out.println("5--" + a2.show(c));
	System.out.println("6--" + a2.show(d));
    System.out.println("7--" + b.show(b));
    System.out.println("8--" + b.show(c));
    System.out.println("9--" + b.show(d));
  }
}
```



```
运行结果为
1-- A and A
2-- A and A
3-- A and D
4-- B and A
5-- B and A
6-- A and D
7-- B and B
8-- B and B
9-- A and D
```



   **当父类对象引用变量引用子类对象时，被引用对象的类型决定了调用谁的成员方法，引用变量类型决定可调用的方法。如果子类中没有覆盖该方法，那么会去父类中寻找。**

```java
class X {
   public void show(Y y){
       System.out.println("x and y");
   }

   public void show(){
       System.out.println("only x");
   }
}

class Y extends X {
   public void show(Y y){
       System.out.println("y and y");
   }
   public void show(int i){

   }
}

class main{
   public static void main(String[] args) {
       X x = new Y();
       x.show(new Y());
       x.show();
   }
}
```

```
运行结果
y and y
only x
```

Y继承了X，覆盖了X中的show（Y y)方法，但是没有覆盖show（）方法。

这个时候，引用类型为X的x指向的对象为Y，这个时候，调用的方法由Y决定，会先从Y中寻找。执行x.show(new Y());，该方法在Y中定义了，所以执行的是Y里面的方法；

但是执行x.show();的时候，有的人会说，Y中没有这个方法啊？它好像是去父类中找该方法了，因为调用了X中的方法。

事实上，Y类中是有show（）方法的，这个方法继承自X，只不过没有覆盖该方法，所以没有在Y中明确写出来而已，看起来像是调用了X中的方法，实际上调用的还是Y中的。

X x = new Y();

**X 决定能调用的方法，show（）和show(Y y)可以调用，而show(int i)不可以调用。Y 决定调用哪个方法；若Y调用的方法在X中没有体现出来，如show(int i)，则会报错**



### 关于调用优先级

继承链中对象方法的调用的优先级：this.show(O)、super.show(O)、this.show((super)O)、super.show((super)O)。

分析经典例子中的第4-- **a2.show(b)**     B b 

A a2 = new B();

由类A中可知可以调用的方法有show(D obj) 和show(A obj)；按照调用优先级，现在B中查找show(B obj),找到了，但是没用，因为不在调用方法内（A中没有该方法） ，所以this.show(B obj)失败；接下来是下一级别：super.show(O),super指的是A，（因为B继承A），在A中寻找show(B obj)也失败，因为没有定义；下一级别：this.show(**super(O)**),这里的this指的时B，**super(O)**在这里是**super(B obj)**，即this.show(**A obj**)，在B中找到该方法，所以调用该方法输出：B and A



第9-- b.show(d)

因为B b = new B() 

b为类型为B的引用对象，指向类型为B的对象，所以不存在向上转型，只会调用本类中的方法

在B中寻找show(D obj),在B中虽然没看到show(D obj),但是因为B继承A，而在A中有show(D obj),所以B中自然也会有该方法，因为在B中没有重写show(D obj)，所以一般情况不明确写出来。