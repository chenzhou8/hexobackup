---
title: Thymeleaf对date类型的input格式化支持
toc: true
categories: JavaEE
date: 2019-04-25 12:55:27
---



## 一、解决问题

最近在使用Thymeleaf模板引擎，但是遇到的问题就是我现在有这样一个标签：输入类型是date，我现在要把JavaBean中的java.sql.Date数据放置到这个额输入框中，但是如你上图所见，这个输入框根本不是单纯的文本框，而且一个日期选择框，好吧….我尝试过**th:datetime** 但是不行，即使是按照格式化的方式也是不行，就像这样的：

```html
<input type="date" th:value="${#dates.format(company.companyRegdate, 'yyyy/MM/dd')}"/>
<input type="date" th:value="${company.companyRegdate}"/>
```

果然还是不行，于是卡了半天的stackoverflow终于出来了：

![image-20190425163248809](http://ww1.sinaimg.cn/large/006HbG3Gly1g2f03s9g7sj30sr0ebwhr.jpg)

```java
/**
 * 企业在此平台起始注册时间
 */
@DateTimeFormat(pattern = "yyyy-MM-dd")
private Date companyRegdate;

/**
 * 企业在此平台注册到期时间
 */
@DateTimeFormat(pattern = "yyyy-MM-dd")
private Date companyUnregdate;

/**
 * 企业信息最后修改更新时间
 */
@DateTimeFormat(pattern = "yyyy-MM-dd")
private Date companyUpdatedate;
```

这里推荐的方式就是给JavaBean的属性上注解一个时间日期格式化器，对的，这个很容易理解，我们所看到的日期不过是像*2018/01/01*这样的字符串，或者说像*2018年1月1日* 这样的字符串，我们和老外对时间的格式表示当然会不一样，但是这个世界上统一的时间就是时间戳，所有的时间表示都是通过时间戳转换而来的。所以我们在存储Date时其实保存的是时间戳的数值，至于具体显示出来时间是怎么样的，要看格式化串，就好比一个模板，所以这个解决方式还是很靠谱的！果断改成@DateTimeFormat，哈哈，还是经验不足呀！

## 二、util.Date与sql.Date

后面呢我又发现出了问题，其实@DatetimeFormat是将String转换成Date，一般前台给后台传值时用@JsonFormat(pattern=”yyyy-MM-dd”)  将Date转换成String  一般后台传值给前台时使用，总结一下其实就是这样！对了顺便提一下，这个想要使用这些注解的前提是必须使用java.util.Date类，而不是java.sql.Date类，否则是格式化注解是会报错的，关于java.util.Date类和java.sql.Date类之前在用的时候还真没注意过这两者的区别，现在还是可以总结一下的：

两者都有getTime方法返回毫秒数，可以直接构建对象，`java.sql.Date`是针对SQL语句使用的，它只包含日期而没有时间部分。`java.util.Date` 是 `java.sql.Date` 的父类，`java.sql.Date`转为`java.util.Date`示例：

```java
java.sql.Date date=new java.sql.Date();
java.util.Date d=new java.util.Date (date.getTime());
```

`java.util.Date`转为`java.sql.Date`示例：

```java
java.util.Date utilDate = new Date();
java.sql.Date sqlDate = new java.sql.Date(utilDate.getTime());
java.sql.Time sTime = new java.sql.Time(utilDate.getTime());
java.sql.Timestamp stp = new java.sql.Timestamp(utilDate.getTime());
```

又遇到一个问题，现在输入类型成了datetime-local，那么如何把Date类型注入到这个标签中呢？下面的方式亲测可用！

datetime-local赋值时间格式：`yyyy-MM-d + 'T' + HH:mm:ss ` 年月日和时分秒分别格式化然后拼接大写”T”

```java
${#dates.format(productInfo.overTime, 'yyyy-MM-dd')} + 'T' + ${#dates.format(productInfo.overTime, 'HH:mm:ss')}
```

![image-20190425164807209](http://ww1.sinaimg.cn/large/006HbG3Gly1g2f03sfz3bj30qi0gh4np.jpg)