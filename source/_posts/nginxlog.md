---
title: nginxlog
date: 2019-03-11 11:20:59
tags: 
- Nginx
- SpringBoot
categories: Nginx
---

![](http://ppe.oss-cn-shenzhen.aliyuncs.com/collections/137/3/thumb.jpg)



# 序

[ nginxlog  一个日志可视化工具  ](https://github.com/isLuoxy/nginxlog)



<!-- more -->

## 1、关于 nginxlog

**nginxlog** ，顾名思义就是跟 nginx 的日志文件有关。这是一个将 nginx log 日志可视化的小工具



## 2、 为什么有 nginxlog 

这个当时是放假期间面试某公司时给我的一个小作业，就是将 nginx 日志文件切割以友好的界面展示出来。所以就有了这个 nginxlog。

当时对这个作业只要求能够运行对应的 jar 包读取就行了。选用的技术栈是 SpringBoot + BootStrap，开发过程中遇到的坑也很多，例如 SpringBoot 的一些页面跳转问题，还有读取 jar 包内的文件问题。关于这些坑，之前已经填完了[【点击查看】](https://www.luoxy.top/2019/01/25/SpringBoot-%E5%B0%8F%E8%AE%B0/)。

虽说完成了这个功能，但是日志文件是在 jar 包内的，可以说是耦合性高，要用的时候只能把文件添加到项目中，并且修改对应的的文件路径，不是很方便，所以想有没有另外一种方式能够实现同样的功能，同时又能实现以下几点：

1. 能够读取外部文件，这里的读取是指在 jar 包中读取外部文件，而不用去修改原本 jar 包的内容。
2. 如果读取的外部文件不存在，有什么优雅点的方法处理错误而不是直接报错。



## 3、 nginxlog 的成长

对于上面的两个需求，难点在于如何 **文件名和文件位置** ，即我们要让这个工具知道文件名是什么，应该从哪里读取文件，才能实现读取功能。



因为 SpringBoot 支持从配置文件中注入值，采用  @ConfigurationProperties 或者 @Value 注解即可实现，关于这两个的做法这里不仔细讲解，网上有很详细的教程。



所以我们可以把日志文件的路径放在配置文件中，然后让 SpringBoot 把配置文件的值注入到特定的 bean 属性中，这样就能做到动态指定文件路径。

![文件路径的Bean](https://i.loli.net/2019/03/11/5c861baab8974.png)



那么现在就是如何动态创建配置文件，而不修改原项目。基本思路是利用 SpringBoot 的命令行启动指定配置文件进行加载，关于 SpringBoot 这个要点就不详细说了，这种指定资源文件的加载存在一个**优先级**的问题，高优先级的配置会覆盖低优先级的配置，所有配置会形成互补配置。[官方说明](https://docs.spring.io/spring-boot/docs/1.5.9.RELEASE/reference/htmlsingle/#boot-features-external-config)



到这里所有问题就解决了，具体思路就是在运行 jar 包时指定一个 配置文件，然后这个配置文件配置 日志的路径，运行时 SpringBoot 会动态将这个配置文件的值注入到特定的 bean 属性中，最后在读取路径时就直接操作这个 bean 中的值，从而实现动态读取文件。



关于 nginxlog 的介绍就到这了，关于这个工具还存在很多不足和需要优化的地方，有需求还会继续更新的。



# 完



> 努力成为一个『不那么差』的程序员 



















