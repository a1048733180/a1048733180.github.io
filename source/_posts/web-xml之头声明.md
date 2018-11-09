---
title: web.xml之头声明
date: 2018-08-29 10:23:08
tags: 
- Servlet
- web.xml
categories: JAVA WEB
---



> 主题：web.xml头部声明的一些坑

<!--more-->

#### servlet 3.1

 Java EE 7 XML schema，命名空间是 <http://xmlns.jcp.org/xml/ns/javaee/>  

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee" 
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
        version="3.1">

</web-app>
```



#### servlet 3.0

Java EE 6 XML schema，命名空间是 <http://java.sun.com/xml/ns/javaee>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
          http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
          version="3.0">

</web-app>
```



#### servlet 2.5

Java EE 5 XML schema，命名空间是 <http://java.sun.com/xml/ns/javaee>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
          http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
          version="2.5">

</web-app>
```



#### servlet 2.4

Java EE 1.4 XML schema, 命名空间是 <http://java.sun.com/xml/ns/j2ee>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee
          http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
          version="2.4">

</web-app>
```



servlet的不同版本会有不同的web.xml头部声明

==可以看出2.4和2.5,3.0大概一致，不过一个是j2ee,一个是javaee。==

而从3.1以后，命名空间改成了 <http://xmlns.jcp.org/xml/ns/javaee/>  

所以在使用框架集成时要注意一些版本的坑，像`async-supported` 是在3.0之后才有的新特性。



------





