---
title: IO 线程模型
date: 2019-08-02 14:24:17
tags: IO 模型
categories: IO 模型
---

![](http://ppe.oss-cn-shenzhen.aliyuncs.com/collections/4/3/thumb.jpg)

# 序

最近在使用 `netty` 开发一个聊天室，练手的同时体会 `netty`  的魅力。记录下基础的 IO 模型。



<!-- more -->

## 何为阻塞、非阻塞

阻塞是指进程被 `cpu`挂起休息，这里有进程切换上下文的消耗。

非阻塞是指进程不被挂起，而是会有机率得到时间片被 `cpu` 调度。（如果此时内核没有准备好数据时，那么此时就相当与空转，浪费 `cpu` 资源），参考轮询。

## 阻塞 I/O

![](https://upload-images.jianshu.io/upload_images/1446087-9522cafa9e14abd0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/552/format/webp)

## 非阻塞 I/O（nonblocking IO）

![](https://upload-images.jianshu.io/upload_images/1446087-0c604ff4a2d8dc5d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/603/format/webp)

尽管在 wait for data阶段，revcfrom 不会被阻塞，但是在 copy data from kernel to user 时，还是会阻塞的，所以这也是 同步IO

## IO 多路复用

![](https://upload-images.jianshu.io/upload_images/1446087-3b0399b077daf0a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/609/format/webp)

IO 多路复用是使用内核系统的 select 函数，线程要进行 IO 操作时，将需要进行操作的 fd 添加到 select 中，然后阻塞。当数据到达时，socket 被激活，然后返回。此时用户线程正式发起 `recvfrom` 请求，读取数据并继续执行。

> 和 同步阻塞区别：
>
> 从流程上看，两者都差不多，不过在 IO 多路复用中，不是发送 `recvfrom`阻塞，而是调用 select，将 fd 添加到 select 中，由 select 进行通知。此时两者都是阻塞的。
>
> 在计算机上，添加多一个中间层往往是能解决很多问题的。从同步阻塞每个线程进来就要创建一个线程调用 read 操作。而在 IO 多路复用中，每个线程进来只要调用 select，将各自的 fd 添加到 select ，由 select 进行管理通知，避免创建线程的消耗，就能同时处理多个 IO 请求。
>
> tip： 这里用户线程阻塞于 select 调用，也就是说这还是一个阻塞操作。不过能够更好的支持并发。如果并发量少的话其性能不会比同步阻塞+多线程好。
>
> 在同步非阻塞中，如果出现很多连接，而此时线程数多的话，导致每个线程的时间片少，cpu频繁进行调度，这个线程切换上下文的消耗是非常大的，如果此时说这些消耗能有回报那还能接受，但是大部分情况下，大部分连接是还没有数据的，那么此时消耗可以看作是 线程上下文切换+cpu 空转 的浪费。如果采用同步阻塞，此时没有 cpu 空转的消耗，但是一开始进行线程挂起时也是需要进行上下文切换的，消耗可看作是 线程上下文切换 的浪费。所以出现了 IO 多路复用，针对线程数多的时候，此时使用一个线程来接收、检查多个文件描述符，如果有一个 fd 就绪，则返回给用户线程。此时对就绪状态的用户连接起一个线程处理（使用线程池，减少创建线程消耗），避免频繁的上下文切换。



在非阻塞IO中，如果用户线程能够不轮询，而先忙自己的事，等消息准备好再去处理，就能提高CPU的利用率了，而这种思路用在 IO 多路复用中，就是使用Reactor设计模式。

![来源网络](https://images0.cnblogs.com/blog/405877/201411/142332350853195.png)

EventHandler 抽象类表示 IO 事件处理器，能通过 `get_handler()` 获取文件句柄，`handle_event()`对 Handle 进行读写操作等。继承于 EventHandler 的子类可以对事件做一个定制化处理。而主角 Reactor 用于管理事件，包括事件的注册、删除等，并使用handle_events实现事件循环，不断调用同步事件多路分离器（一般是内核）的多路分离函数select。只要某个文件句柄被激活（可读/写等），`select` 就返回，`handle_events`就会调用与文件句柄关联的事件处理器的 `handle_event()`进行相关操作。

![来源网络](https://images0.cnblogs.com/blog/405877/201411/142333254136604.png)

​	由于 Reactor 设计模式背后还是使用 select 系统调用，所以总的来说只能称为 异步阻塞IO，也不是真正的 异步 IO，从上图的 read 请求时数据拷贝阻塞用户线程也能知道还是阻塞的。



如果想做到真正的异步 IO，需要系统内核提供支持，原理就是内核接收到数据后，能够把数据复制到用户态的数据区，再通知用户线程直接使用即可。（或者可以使用内存映射，不用进行内核区数据到用户态数据的复制），在异步IO模型中使用 Proactor 设计模式。

![来源网络](https://images0.cnblogs.com/blog/405877/201411/151608309061672.jpg)

用户线程将 `AsynchronousOperation`（读/写等）、`Proactor` 以及操作完成时的 `CompletionHandler` 注册到 `AsynchronousOperationProcessor`,`AsynchronousOperationProcessor`会提供一组异步 API ，用户线程调用后即可继续执行自己的任务，而不会被阻塞在调用上。`AsynchronousOperationProcessor `会开启独立的内核线程执行异步操作，实现真正的异步。当异步IO操作完成时，`AsynchronousOperationProcessor`将用户线程与`AsynchronousOperation` 一起注册的 `Proactor` 和 `CompletionHandler` 取出，然后将 `CompletionHandler` 与IO操作的结果数据一起转发给 `Proactor`，`Proactor` 负责回调每一个异步操作的事件完成处理函数· `handle_event`。

![来源网络](https://images0.cnblogs.com/blog/405877/201411/142333511475767.png)

用户线程调用异步 API read() ，将 `AsynchronousOperatio、Proactor、CompletionHandler` 注册到内核，然后直接返回。操作系统开启独立的内核线程去处理 IO 操作。当 read 请求的数据到达时，由内核负责读取数据，并写入用户态的数据区中。最后内核将 read 的数据和用户线程注册的 `CompletionHandler` 分发给内部`Proactor`，`Proactor` 将IO完成的信息通知给用户线程（一般通过调用用户线程注册的完成事件处理函数），完成异步IO。



# 完



> 努力成为一个『不那么差』的程序员 