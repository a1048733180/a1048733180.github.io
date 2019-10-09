---
title: Netty-多个ChannelHandler之间传递
date: 2019-10-09 12:45:14
tags: 
- Netty
- handler
categories: Netty
---

![](http://ppe.oss-cn-shenzhen.aliyuncs.com/collections/165/3/thumb.jpg)

# 序

在使用 `Netty` 中的 `ChannelHandler` 做业务处理时，本想通过多个不同的 `ChannelHandler`进行不同的逻辑处理，后发现 `handler` 之间没有进行消息传递（后发现是因为自己的 `handler` 继承了 SimpleChannelInboundHandler 这个模板类），现将这个知识点结合源码进行简单分析。



<!-- more -->



## ChannelHandler

看一段简单的服务端连接代码，其中在第10行中往 pipeline 中添加了一个 handler

~~~java
// 服务端连接代码
 NioEventLoopGroup boosGroup = new NioEventLoopGroup();
 NioEventLoopGroup workerGroup = new NioEventLoopGroup();
 ServerBootstrap serverBootstrap = new ServerBootstrap();
 serverBootstrap.group(bossGroup, workerGroup)
                .channel(NioServerSocketChannel.class)
                .childHandler(new ChannelInitializer<NioSocketChannel>() {
                   @Override
                   protected void initChannel(NioSocketChannel ch) throws Exception {
                    ch.pipeline().addLast(new ServerHandler());
                  }
               });

// ServerHandler
public class ServerHandler extends ChannelInboundHandlerAdapter {
		
}
~~~



这里 handler 是自定义的一个 `handler`，继承了 `ChannelInboundHandlerAdapter`，而`ChannelInboundHandlerAdapter` 就是一个适配类，其父类即为 `ChannelHandler`，

![](<https://i.loli.net/2019/10/09/Z8YgXHJhS2BQWjy.png>)


从语义可以看出这个是入站程序适配器，那么自然还有个出站程序适配器，这里先不对这两个类做详细分析。


通常在这个 `handler` 中我们就可以进行一系列的逻辑操作，可以针对不同的消息类型作出对应的处理。但这样就不符合单一原则。而且 `Netty` 官方也提供了模块化处理，通过 `pipline` 和 `channelHandler`，运用责任链设计模式，形成一个业务逻辑链，处理对应的业务逻辑。



## Pipline

![图片来自掘金小册《[Netty 入门与实战：仿写微信 IM 即时通讯系统]》](https://user-gold-cdn.xitu.io/2018/8/17/1654526f0a67bb52?imageslim)

这里借用掘金小册《[Netty 入门与实战：仿写微信 IM 即时通讯系统](https://juejin.im/book/5b4bc28bf265da0f60130116)》 的一个插图，每个连接对应一个 `channel`，每个`channel`都有其自己的`pipeline`，并且在创建新通道时会自动创建它。此外 `handler`、`ChannelHandlerContext`、`pipeline` 之间是什么关系呢？



`ch.pipeline().addLast(new ServerHandler());` 通过这行代码追踪源码可以发现，`pipeline` 包含 `ChannelHandlerContext` ,`ChannelHandlerContext`包含 `ChannelHandler`，其中 `ChannelHandlerContext` 是一个双向链表，所以结合上图就能知道三者之间的关系。 



当我们调用 `ch.pipeline().addLast(new ServerHandler())` 时， ` ServerHandler` 会被封装成 `ChannelHandlerContext`，并拼接在 `pipeline` 中的 `ChannelHandlerContext` 的 next 中，其中 `ChannelHandlerContext ` 就是 `pipeline ` 中的节点。



## ChannelHandler 之间的事件传播

了解完 `Netty` 中有关 `ChannelHandler` 的构造后，接下来看下是如何进行事件传播的。

![](https://i.loli.net/2019/10/09/VX6UkdNI9Rw1mnv.png)

从官方文档可以看出，通过调用 `ChannelHandlerContext` 的 `fire*` 的方法传播到下个 handler 处理。



`Netty` 提供两种方式供我们实现 `ChannelHandler`，一种是继承 `ChannelInboundHandlerAdapter`，一种是继承 `SimpleChannelInboundHandler`

~~~java
// ChannelInboundHandlerAdapter
public class ServerHandler1 extends ChannelInboundHandlerAdapter {
    // 省略一些方法实现
}


// SimpleChannelInboundHandler
public class ServerHandler2 extends SimpleChannelInboundHandler<Object> {
   // 省略一些抽象方法实现
}
~~~

这两者有什么区别呢？可以看下继承图

![](https://i.loli.net/2019/10/09/rLD9Uko1RamCwdp.png)

其中 `SimpleChannelInboundHandler` 继承 `ChannelInboundHandlerAdapter`，并且有一个抽象方法 `channelRead0` 留给子类实现。

`SimpleChannelInboundHandler` 是一个泛型类，而 `ChannelInboundHandlerAdapter` 并不是一个泛型类。这里的泛型主要用于消息格式转换使用。在使用 `ChannelInboundHandlerAdapter`  进行操作时，我们需要自己转换消息格式，而 `SimpleChannelInboundHandler` 就是 `Netty` 帮我们转换消息格式，只要我们在泛型中声明转换的类型，并且这个类型支持被转换，那么我们就可以使用这个转换后的消息进行操作。下面以两者的 `channelRead()` 方法进行比较。

~~~java
// ChannelInboundHandlerAdapter
public class ChannelInboundHandlerAdapter extends ChannelHandlerAdapter implements ChannelInboundHandler {
    // 直接调用 ChannelHandlerContext 的 fireChannelRead() 方法传递给下一个 handler
	@Skip
    public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        ctx.fireChannelRead(msg);
    }
}
~~~



~~~java
// SimpleChannelInboundHandler 
public abstract class SimpleChannelInboundHandler<I> extends ChannelInboundHandlerAdapter {
    // 重写 channelRead ，并且调用钩子方法 channelRead0，channelRead0 由子类实现
	public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
        boolean release = true;
        try {
            if (this.acceptInboundMessage(msg)) {
                this.channelRead0(ctx, msg);
            } else {
                release = false;
                ctx.fireChannelRead(msg);
            }
        } finally {
            if (this.autoRelease && release) {
                ReferenceCountUtil.release(msg);
            }

        }

    }
	
    // 这里的 var2 是 I 类型，这个 I 就是我们声明的泛型，当我们重写这个方法时，传进来的消息就已经是特定格式的了，此时我们直接使用即可，不需要再自行转换格式
    protected abstract void channelRead0(ChannelHandlerContext var1, I var2) throws Exception;
}
~~~



## 实现

下面我们分别实现上述两个类，看如何编写代码才能将消息在多个 handler 中传播

- `ChannelInboundHandlerAdapter`

  当我们继承这个类时，并且重写对应的 `channelRead()` 方法时，那么此时如果需要继续传播的话，需要调用 `super.channelRead()` 方法，或者自行调用 `ChannelHandlerContext#fireChannelRead()` ；

- `SimpleChannelInboundHandler`

  继承这个类时，从源码可以看出在重写的 `channelRead()` 方法中，在 finally 的判断中,这里再贴下源码

  ~~~java
  public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
          boolean release = true;
          try {
              // 一般返回的都为 true，进入子类实现的 channelRead0 方法中
              if (this.acceptInboundMessage(msg)) {
                  this.channelRead0(ctx, msg);
              } else {
                  release = false;
                  ctx.fireChannelRead(msg);
              }
          } finally {
              // autoRelease 默认为 true，release 也为 true，所以会调用该方法，而该方法是 Netty 持有的一个计数器，并且该计数器与 fire* 有联系，
              if (this.autoRelease && release) {
                  ReferenceCountUtil.release(msg);
              }
  
          }
  
   }
  ~~~

  `ReferenceCountUtil.release(msg);` 这个方法会将计数器相应-1，而如果为负数的话，那么调用 fire* 方法就会报错。所以此时我们不能直接在业务逻辑代码中直接调用`ChannelHandlerContext#fireChannelRead()` 

  关于这种情况的解决方案，有以下两种方法：

  1. 在实现类中修改父类的 `autoRelease` 的值为 `false`，这样就不会运行`ReferenceCountUtil.release(msg)`，此时在 实现类中的 `channelRead0()` 调用 `ChannelHandlerContext#fireChannelRead()`  进行事件传播。

  2. 既然在 finally 中执行 `ReferenceCountUtil.release(msg)`，那我们可以直接在代码执行 `ReferenceCountUtil.retain()` 进行计数器+1，相互抵消。然后再调用`ChannelHandlerContext#fireChannelRead()` 进行事件传播



# 完



>  努力成为一个『不那么差』的程序员 