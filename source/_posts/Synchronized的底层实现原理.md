---
title: Synchronized的底层实现原理
date: 2018-08-06 16:36:20
tags: 多线程
categories: 
- JAVA
- 多线程
---

本文简单描述Synchronized的一些实现方式和底层的原理

<!-- more -->

# Synchronized的底层实现原理



## 1. Synchronized的代码实现

- 修饰代码块

  ```java
  Object obj = new Object();
  synchronized(obj){
      do something;
  }
  ```

  其中代码块中修饰的**obj**是对象，可以是类对象（Class.class），那么锁住的是整个类。

- 修饰方法

  ```java
  // 修饰非静态方法
  public synchronized void testSynchronized(){
      do something;
  }
  // 修饰静态方法
  public static synchronized void testStaticSynchronized(){
      do something;
  }
  ```

  修饰静态方法时，因为静态方法是对象共享的，所以在静态方法这里加上**Synchronized**和非静态方法加上**Synchronized** 作用是一样的，都要先获得锁才能往下执行。

  

  ```java
  // 同时存在非静态方法和静态方法时，参考上面修饰方法代码，无论是采用同步代码块还是同步方法
  public class Test{
      public void test1(){
          synchronized(this){ // 对象锁
              do something;
          }
      }
      
      public void test2(){
          synchronized(Test.class){ // 类锁，也可以说是对象锁，不过锁的是类，即同个类的不同对象在进入该方法时都会竞争锁，得到锁后才能往下执行
              do something;
          }
      }
      
      public static void main(String[] args){
          Test test = new Test();
          Thread thread1 = new Thread(
              new Runnable(){
                  public void run(){
                      test.test1();
                  }
              }
          ).start();
          Thread thread2 = new Thread(
              new Runnable(){
                  public void run(){
                      test.test2();
                  }
              }
          ).start();
      }
  }
  
  // 从结果可以看出对象锁和类锁是相互独立的，thread1和thread2是同时进行的，互不干扰。
  ```

  

## 2.Synchronized的原理

 **Synchronized**用在多线程并发中，避免多个线程对共享资源的访问造成的可见性和原子性等安全问题。



1. 当Synchronized用在 代码块中，在JVM中是采用**monitorenter**和**monitorexit**两个指令来实现同步。

   > 关于``monitorenter`和`monitorexit`两个指令
   >
   > 可以把执行`monitorenter`指令理解为加锁，执行`monitorexit`理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行`monitorenter`）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行`monitorexit`指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。 

   

2. 当Synchronized用在方法上时，在JVM中是采用**ACC_SYNCHRONIZED**标记符来实现同步。

   > 关于`ACC_SYNCHRONIZED`
   >
   > 方法级的同步是隐式的。同步方法的常量池中会有一个`ACC_SYNCHRONIZED`标志。当某个线程要访问某个方法的时候，会检查是否有`ACC_SYNCHRONIZED`，如果有设置，则需要先获得监视器锁，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放。 



## 3. Synchronized的底层实现

 我们通常用`Synchronized`去实现同步，实现原理就是当多个线程访问共享资源时，使用**synchronized**能够让线程在访问共享资源前要先经过一到屏障，这道屏障就是对应的对象锁或者是类锁。线程要先获得锁，才能访问共享资源，而每次锁只能给一个线程，通过锁的运用去实现同步。

那么什么是锁呢？

### Moniter

> 这就是**Synchronized**能够实现同步机制的最终Boss
>
> **moniter**，可以称为监视器锁，也可以抽象理解成锁，线程获得锁，就是获得moniter。

在Java虚拟机(HotSpot)中，Monitor是基于C++实现的，由ObjectMonitor实现的， 

ObjectMonitor中有几个关键属性： 

> _owner：指向持有ObjectMonitor对象的线程
>
> _WaitSet：存放处于wait状态的线程队列
>
> _EntryList：存放处于等待锁block状态的线程队列
>
> _recursions：锁的重入次数
>
> _count：用来记录该线程获取锁的次数

当多个线程同时访问一段同步代码时，首先会进入`_EntryList`队列中，当某个线程获取到对象的monitor后进入`_Owner`区域并把monitor中的`_owner`变量设置为当前线程，同时monitor中的计数器`_count`加1。即获得对象锁。

若持有monitor的线程调用`wait()`方法，将释放当前持有的monitor，`_owner`变量恢复为`null`，`_count`自减1，同时该线程进入`_WaitSet`集合中等待被唤醒。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。

![](https://i.loli.net/2018/08/08/5b6a57a13eee0.jpg)

