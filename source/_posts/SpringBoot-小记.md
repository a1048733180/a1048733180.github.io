---
title: SpringBoot 小记
date: 2019-01-25 00:22:11
tags: SpringBoot
categories: something
---

![泼辣有图](http://ppe.oss-cn-shenzhen.aliyuncs.com/collections/2/4/full_res.jpg)

# 序

最近捡起一点 SpringBoot 的知识 ，做一些小记。

<!-- more -->



## 1、关于 SpringBoot 静态页面问题

一般来说，SpringBoot 是使用模版去进行前后端的交互；而静态的 html 文件只能通过 ajax 进行交互。SpringBoot 支持可以直接通过访问 xxx.html 到服务器的某静态文件，所以一般来说会有拦截器进行拦截判断。下面对几种不同的控制器返回页面方式进行小记

-  不使用模版（静态 html ）

  ==一般来说，SpringBoot 建议全部静态文件放在 static 目录下（src/main/resources/static） ，而 SpringBoot 对该目录的访问路径默认为 /==，

  意思就是如果你要访问该路径下的文件，例如：

  ![](https://i.loli.net/2019/01/24/5c49d5b601bee.png)

  要访问 chart.html ，其路径是（ src/main/resources/html/chart.html ），但是我们访问时只需要（ /html/chart.html ）即可访问。

  ![](https://i.loli.net/2019/01/24/5c49d67b34e15.png)

  

  如果是通过控制器路由的话，在配置文件中 （ application.properties / application.yml ）配置好页面的前缀和后缀。

  ![](https://i.loli.net/2019/01/24/5c49d7a85e594.png)

  控制器直接返回字符串即可，==这里不能使用 @ResponseBody 注解方法，或者使用 @RestController 注解控制器类==，因为这样会直接将字符串返回回去，不经过页面解析。

  ![](https://i.loli.net/2019/01/24/5c49d841a8347.png)

  控制器写法就不多说了，重点就是返回的字符串是视图的名字，经过页面解析后会组装成 /chart.html,这样就跟前端直接访问资源文件一样，所以现在一般都是后端给定接口和数据，路由由前端控制。

  > 如果要更改默认的静态资源访问路径，可以在配置文件中更改 spring.resources.static-locations
  >
  > ![](https://i.loli.net/2019/01/24/5c49dd0d19f42.png)
  >
  > 这样默认的静态资源访问路径就修改了，这样可以通过 localhost:8080/chart.html 访问到该页面
  >
  > ![](https://i.loli.net/2019/01/24/5c49dd9e414a6.png)
  >
  > 
  >
  > 如果使用控制器路由同理只要修改配置文件中的前后缀文件即可。==重点是要知道当前的静态资源访问路径是什么，做出对应修改就行了。==

- 使用模版（Thymeleaf）

  SpringBoot 中 Thymeleaf 模版默认会使用 templates 作为视图文件，

  ![](https://i.loli.net/2019/01/24/5c49da71a2408.png)

  在配置文件配置后对应视图文件的前后缀，通过控制器返回视图名即可访问

  ![](https://i.loli.net/2019/01/24/5c49de4a1fcb0.png)

## 2、jar 包访问 resources 目录下的文件

- 为什么要打包成 jar 包

SpringBoot 内置 tomcat，所以我们可以直接将一个 SpringBoot 项目打包成 jar 包，然后通过命令行启动，而不用过多的去配置 Tomcat 或其他服务器

```cmd
java -jar xxx.jar 
```

- 背景

  假设我们现在有一个读取本地文件的需求，（通常我们用数据库代替），这里是假设我们通过读取本地文件。然后在本地开发我们读取该文件很容易，只要提供其相对路径或者绝对路径。但是如果要在另一台服务器启动，那么该本地文件自然也要在项目里面，（也可以放在 jar 包外通过命令行时指定本地文件，这里不探讨这种做法）。

  假设我们把文件放在 resources 下，文件名为 test.log ,那么我们读取该文件时一般来说都是通过 FileInputStream 操作的，这样在开发时是正常的，因为文件是真实存在与某个磁盘上的。

  但是当你打包成 jar 包后，会发现报错，**java.io.FileNotFoundException** ,并且路径会多了一些不存在的 ！号   

   \BOOT-INF\classes!/test.log，类似这样。这种问题产生的原因是因为当你打包成 jar 包后，磁盘是没有真实路径的，所以无法通过 FileInputStream 去获取

- 解决方法

  我们可以修改 FileInputStream 为 InputStreamReader,先获取对应文件的 InputStream 

  ```java
  // 会指定要加载的资源路径与当前类所在包的路径一致。因此能正常读取文件
  InputStream stream = getClass().getClassLoader().getResourceAsStream("文件名");
  bufferedReader in = new BufferedReader(new InputStreamReader(stream, "UTF-8"));
  ```

  这样打包成 jar 包后也能正常访问文件了。

  
  
# 完

> 努力成为一个『不那么差』的程序员 