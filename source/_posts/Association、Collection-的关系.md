---
title: Association 、Collection区别
date: 2019-01-14 20:25:44
tags: MyBatis
categories: MyBatis
---


![1.jpg](https://i.loli.net/2019/01/14/5c3c863f2e0a1.jpg)


# 序


最近在做课设时，持久层采用MyBatis，遇到了一些关于 association 和 collection 的问题，现在把它记录一下。


<!-- more -->



通常我们都会进行多表查询，这个查询的结果往往涉及到多个类或者集合等。

假设现在有这样一个关系，一个学生属于一个专业，一个专业有多个课程。在Mysql中有4个表的存在

```sql
-- 学生表 --
create table u_student(
   student_id varchar(11) not null,
   student_name varchar(10) not null,
   student_profession_id varchar(5),
   primary key(student_id),
   foreign key (student_profession_id) references u_profession(profession_id)
)

-- 专业表 --
create table u_profession(
  profession_id varchar(5) not null,
  profession_name varchar(10) not null,
  primary key(profession_id)
)

-- 课程表 --
create table u_course(
  course_id varchar(5) not null,
  course_name varchar(10) not null,
  primary key(course_id)
)

-- 上课任务 --
create table u_profession_course(
  profession_id varchar(5),
  course_id varchar(5),
  foreign key (profession_id) references u_profession(profession_id),
  foreign key (course_id) references u_course(course_id)
)
```

这里表的设计为了消除冗余，例如一个专业有多个课程，如果直接把课程放在专业表里面充当属性的话，会有很多重复列。所以创建上课任务表消除冗余。

对应的类为

```java
// 学生
public class Student{
    private String studentId;
    private String studentName;
    private Profession profession;
    // 省略对应的 getter/setter 方法
}

// 专业
public class Profession{
    private String professionId;
    private String progessionName;
    private List<Course> courseList;
    // 省略对应的 getter/setter 方法

}

// 课程
public class Course{
    private String courseId;
    private String courseName;
    // 省略对应的 getter/setter 方法
}

```



```java
public interface StudentDao{
  // 查询特定学号的学生信息
    Student selectStudentById(String stuId);
}
```

对应的 studentmapper.xml 是（如果我们要查询某个学生的信息（包括专业名称），假设sql语句是写在 * mapper.xml中的）

```xml
<!-- 省略 namespace 等 -->
<select id ="selectStudentById" javaType = "int" resultMap = "getStudent">
    select a.*,b.profession_name from u_student a, u_profession b where a.profession_id = b.profession_id and a.student_id = #{stuId}
</select>

<resultMap id = "getStudent" type = "Student">
  <id property = "studentId" column="student_id" />
  <result property = "studentName" column="student_name" />
  <association property = "profession" javaType = "Profession">
    <id property = "professionId" column = "profession_id" />
    <result property = "professionName" column = "profession_name" />
  </association>
</resultMap>
```

一个学生对应一个专业，一个专业含有多个学生。当我们通过学号查询学生信息时，使用 association 去达到 一对多 关系的目的。



```java
public interface ProfessionDao{
    // 查询某专业信息（如果有课程要把课程也输出）
    List<Profession>  getProfessionList();
}
```

对应的 ProfessionMapper.xml 是

```xml
<!-- 省略 namespace 等 -->

<!-- a.*  profession_id 、profession_name -->
<!-- c.* course_id 、course_name -->
<select id = "getProfessionList" resultMap = "professionList" >
    select a.*,c.* from
    u_profession a 
    left join 
    u_profession_course b 
    on a.profession_id = b.profession_id 
    left join 
    u_course c 
    on c.course_id = b.course_id
</select>

<resultMap id = "professionList" type = "Profession">
  <id property = "professionId" column = "profession_id" />
  <reuslt property = "professName" column = "profession_name" />
  <collection property = "courseList" ofType = "Course">
     <id property = "courseId" column = "course_id" />
     <result property = "courseName" column = "course_name" />
  </collection>
</resultMap>
```

可以看到在 Profession 中的 List<Course> 是使用 collection 封装的。



如果在 Profession 中，改成如下:

```java
public class Profession{
   // 其他不变`
    ……
    private List<Course> courseList = new ArrayList<>();
    ……
   // 省略 getter/setter 方法
}
```

```xml
<resultMap id = "professionList" type = "Profession">
  <id property = "professionId" column = "profession_id" />
  <reuslt property = "professName" column = "profession_name" />
  <association id = "courseList" javaType = "Course">
     <id property = "courseId" column = "course_id" />
     <result property = "courseName" column = "course_name" />
  </association>
</resultMap>
```

这里在 Profession 类中初始化 List<Course> ，然后就可以使用 association 关联去做 映射，（即把数据库的值赋值给属性）；



而如果没有初始化 List<Course> 的话，就会报错，出现 Dismatch 的错误，即类型不匹配。



从MyBatis的文档可以看出，association 是一个复杂类型的关联；collection 是一个复杂类型的集合·



但是当你 JavaBean 中集合类型已经初始化的话，发现其实也是可以使用 association 去做关联映射,这是为什么呢？如果不初始化，就没办法用 association 作关联映射。



我的初步猜测是 MyBatis 的 Association 和 Collection 工作的原理是根据我们定义的 javaType（ofType）实例化一个对象出来，再根据这个对象去作映射。

```java
private Profession profession;

<association property = "profession" javaType = "Profession">
```

像这里 MyBatis 应该会实例化一个 Profession 的对象，然后再映射到对应的属性。



```java
private List<Course> courseList;

 <collection property = "courseList" ofType = "Course">
```

这里 MyBatis 会先自动实例化一个集合对象，（具体集合类型不清晰），然后再实例化 Course 对象，就是说这里 MyBatis 会创建两个对象。



```java
private List<Course> courseList;

<association property = "courseList" javaType = "Course">

// 这里不会创建集合对象，只是创建Course对象，但是集合对象为空的话就会报错。所以会报 DisMatch 的错误
```

```java
private List<Course> courseList = new ArrayList<>();

<association property = "courseList" javaType = "Course">

// 这里就能够运行成功，因为集合对象已经创建了，MyBatis 创建 Course 对象并且映射到对应的属性
```



以上都是自己的猜测，等以后再进一步跟进源码去验证正确性。



# 完



> 努力成为一个『不那么差』的程序员 