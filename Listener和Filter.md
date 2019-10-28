---
title: Listener和Filter
date: 2018-04-30 08:52:14
toc: true
categories: Java核心技术
---

## 监听器 listener  

其中 servlet规范包括三个技术点：servlet  listener  filter

### 什么是监听器？

监听器就是监听某个对象的的状态变化的组件

监听器的相关概念：
事件源：被监听的对象  ----- 三个域对象 request、session、servletContext
监听器：监听事件源对象  事件源对象的状态的变化都会触发监听器 ---- 6+2
注册监听器：将监听器与事件源进行绑定
响应行为：监听器监听到事件源的状态变化时 所涉及的功能代码 ---- 程序员编写代	码

### 监听器有哪些？

第一维度：按照被监听的对象划分：ServletRequest域   HttpSession域   	ServletContext域
第二维度：监听的内容分：监听域对象的创建与销毁的    监听域对象的属性变	化的

|                    | ServletContext域                | HTTPSession域                | ServletRequest域       |
| :----------------- | :------------------------------ | :--------------------------- | :--------------------- |
| 域对象的创建和销毁 | ServletContextListener          | HTTPSessionListener          | ServletRequestListener |
| 域对象内属性的变化 | ServletContextAttributeListener | HTTPSessionAttributeListener | ServletRequestListener |

### 监听三大域对象的创建与销毁

#### 监听ServletContext域的创建与销毁的监听器：ServletContextListener

##### Servlet域的生命周期

何时创建：服务器启动创建
何时销毁：服务器关闭销毁

##### 监听器的编写步骤（重点）

a、编写一个监听器类去实现监听器接口
b、覆盖监听器的方法
c、需要在web.xml中进行配置---注册

##### 监听的方法：

```java
public class MyServletContextListener implements ServletContextListener{

	@Override
	//监听context域对象的销毁
	public void contextDestroyed(ServletContextEvent con) {
		System.out.println("context销毁...");
	}

	@Override
	//监听context域对象的创建
	public void contextInitialized(ServletContextEvent arg0) {
		System.out.println("context创建...");
	}
}
```

```xml
<!-- 在web.xml中注册监听器 -->
<listener>
    <listener-class>com.xpu.create.MyServletContextListener</listener-class>
</listener>
```

##### ServletContextListener监听器的主要作用

* 初始化的工作：初始化对象 初始化数据 ---- 加载数据库驱动  连接池的初始化
* 加载一些初始化的配置文件 --- spring的配置文件
* 任务调度----定时器----Timer/TimerTask





#### 监听Httpsession域的创建于销毁的监听器：HttpSessionListener

##### HttpSession对象的生命周期

何时创建：第一次调用request.getSession时创建

何时销毁：服务器关闭销毁、session过期、手动销毁

##### HttpSessionListener的方法

```java
public class MyHTTPSessionListener implements HttpSessionListener {

	@Override
	public void sessionCreated(HttpSessionEvent se) {
		System.out.println("sessionCreated:"+se.getSession().getId());
	}

	@Override
	public void sessionDestroyed(HttpSessionEvent se) {
		System.out.println("sessionDestroyed:"+se.getSession().getId());
	}
}
```

#### 监听ServletRequest域创建与销毁的监听器：ServletRequestListener

##### ServletRequest的生命周期

何时创建：每一次请求都会创建request

何时销毁：请求结束

##### ServletRequestListener的方法

```java
public class MyServletRequqestListener implements ServletRequestListener {

	@Override
	public void requestDestroyed(ServletRequestEvent sre) {
		System.out.println("创建request");
	}

	@Override
	public void requestInitialized(ServletRequestEvent sre) {
		System.out.println("销毁request");
	}
}
```

### 监听三大域对象的属性变化

#### 域对象的通用的方法

setAttribute(name,value) 方法创建或改变某个新属性 

getAttribute(name) 方法通过名称获取属性的值 

removeAttribute(name)  方法通过名称删除属性的值

#### ServletContextAttibuteListener的方法

```java
public class MyServletContextAttributeListener implements ServletContextAttributeListener{

	@Override
	public void attributeAdded(ServletContextAttributeEvent scae) {
		System.out.println(scae.getName());//放到域中的name
		System.out.println(scae.getValue());//放到域中的value
	}

	@Override
	public void attributeRemoved(ServletContextAttributeEvent scae) {
		System.out.println(scae.getName());//获得修改前的name
		System.out.println(scae.getValue());//获得修改前的value
	}

	@Override
	public void attributeReplaced(ServletContextAttributeEvent scae) {
		System.out.println(scae.getName());//修改域中的name
		System.out.println(scae.getValue());//修改域中的value
	}
}
```

HttpSessionAttributeListener、ServletRequestAriibuteListenr监听器方法同上，故此不再赘述

### 与session中的绑定的对象相关监听器

与session中的绑定的对象相关监听器即是对象感知监听器



绑定状态：就一个对象被放到session域中

解绑状态：就是这个对象从session域中移除了

钝化状态：是将session内存中的对象持久化（序列化）到磁盘

活化状态：就是将磁盘上的对象再次恢复到session内存中



当用户很多时，怎样对服务器进行优化？这就涉及到对象的钝化与活化，只要把内存中的对象存储到磁盘中就可以从很大程度上减轻内存的消耗，从而达到服务器优化的目的！

#### 绑定与解绑的监听器：HttpSessionBindingListener

```java
public class Person implements HttpSessionBindingListener{
	
	@Override
	//绑定的方法
	public void valueBound(HttpSessionBindingEvent event) {
		System.out.println("Person 被绑定");
	}
	
	@Override
	//解绑的方法
	public void valueUnbound(HttpSessionBindingEvent event) {
		System.out.println("Person 解绑");
	}
}
```

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		HttpSession session = request.getSession();
		Person p = new Person("tom", 19);
		
		//触发绑定的监听器方法valueBound()
		session.setAttribute("person", p);
		//触发解除绑定的监听器方法valueUnbound()
		session.removeAttribute("person");
}
```

#### 钝化与活化的监听器:HttpSessionActivationListener

```java
public class Students implements HttpSessionActivationListener,Serializable{
	private String name;
	private int age;
	//钝化（内存---->硬盘）
	@Override
	public void sessionWillPassivate(HttpSessionEvent se) {
		System.out.println("Student对象被钝化了！");
	}
	//活化（硬盘---->内存）
	@Override
	public void sessionDidActivate(HttpSessionEvent se) {
		System.out.println("Student对象被活化了！");
	}
}
```

**想要实现对象的钝化和活化的时候需要实现Serializable接口**，这个属于对象序列化的接口就不赘述了

```java
public class AServlet extends HttpServlet {
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		HttpSession session = request.getSession();
		Students students = new Students("tom", 19);
		
		session.setAttribute("students", students);
		System.out.println("students被放入session域中");
	}
}
```

首先我们访问这个AServlet，然后关掉服务器，注意：不要点击控制台那个停止，如图所示：
![](https://s2.ax1x.com/2019/05/08/E6fxtx.png)

此时session域的对象会转储到`apache-tomcat-7.0.92\work\Catalina\localhost\你的工程路径`这个文件夹中

重新启动服务器后访问BServlet的时候，对象便会活化，重新加载到内存，监听器监听到后便会执行相关的函数：

```java
public class BServlet extends HttpServlet {
	protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		//从session域中获得students
		HttpSession session = request.getSession();
		Students attribute = (Students)session.getAttribute("students");
		
		System.out.println(attribute.getName()+":"+attribute.getAge());
	}
}
```

可以通过配置文件指定对象钝化时间 ---> 对象多长时间不用被钝化

在META-INF下创建一个context.xml：

```xml
<Context>
	<!-- maxIdleSwap:session中的对象多长时间不使用就钝化(以分钟为单位) -->
	<!-- directory:钝化后的对象的文件写到磁盘的哪个目录下 配置钝化的对象文件在 work/catalina/localhost/钝化文件 -->
	<Manager
		className="org.apache.catalina.session.PersistentManager"
		maxIdleSwap="1">
		<Store className="org.apache.catalina.session.FileStore"
			directory="mydirectory" />
	</Manager>
</Context>
```

## 过滤器 filter

### filter的简介

过滤器可以拦截所有访问web资源的请求或响应操作。执行过滤任务的对象，这些任务是针对对某一资源（servlet 或静态内容）的请求或来自某一资源的响应执行的，抑或同时针对这两者执行 ，是对客户端访问资源的过滤，符合条件放行，不符合条件不放行，并且可以对目标资源访问前后进行逻辑处理，下面是Filter的基本使用流程：

1. 编写一个过滤器的类实现Filter接口
2. 实现接口中尚未实现的方法(着重实现doFilter方法)
3. 在web.xml中进行配置(主要是配置要对哪些资源进行过滤)

![](https://s2.ax1x.com/2019/05/08/E6hp9K.png)

### filter生命周期及其API

Filter接口有三个方法，并且这个三个都是与Filter的生命相关的方法

init(Filterconfig)：代表filter对象初始化方法 filter对象创建时执行

doFilter(ServletRequest,ServletResponse,FilterCha)：代表filter执行过滤的核心方法，如果某资源在已经被配置到这个filter进行过滤的话，那么每次访问这个资源都会执行doFilter方法

destory()：代表是filter销毁方法 当filter对象销毁时执行该方法

Filter对象的生命周期：

Filter何时创建：服务器启动时就创建该filter对象

Filter何时销毁：服务器关闭时filter销毁

```java
public class QuickFilter1 implements Filter{

	@Override
	public void init(FilterConfig filterConfig) throws ServletException {
		System.out.println("init");
	}

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		//拦截别人的访问
		System.out.println("quick1 running ...");
		//放行请求
		chain.doFilter(request, response);
	}

	@Override
	public void destroy() {
		System.out.println("destroy");
	}
}
```

web.xml文件的配置：

**注意：Filter的过滤顺序是按照filter-mapping的顺序过滤的**

```xml
<filter>
    <filter-name>QuickFilter</filter-name>
    <filter-class>com.xpu.web.filter.QuickFilter1</filter-class>
</filter>
<filter-mapping>
    <filter-name>QuickFilter1</filter-name>
    <!-- 默认过滤所有url -->
    <url-pattern>/*</url-pattern>
</filter-mapping>

<filter-mapping>
    <filter-name>QuickFilter2</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

### Filter的API详解

#### init(FilterConfig)方法

其中参数config代表 该Filter对象的配置信息的对象，内部封装是该filter的配置信息。

假设web.xml配置了如下的Filter：

```xml
<filter>
    <display-name>QuickFilter3</display-name>
    <filter-name>QuickFilter3</filter-name>
    <filter-class>com.xpu.web.filter.QuickFilter3</filter-class>
    <init-param>
        <param-name>aaa</param-name>
        <param-value>AAA</param-value>
    </init-param>
  </filter>
```

那么在filterConfig中就油API可以获取这些配置：

```java
//Filter的过滤顺序是按照filter-mapping的顺序过滤的
public void init(FilterConfig fConfig) throws ServletException {
	//fConfig维护着Filter的配置信息
	System.out.println(fConfig.getFilterName());//<filter-name>QuickFilter3</filter-name>
	
	//获取初始化参数
	System.out.println(fConfig.getInitParameter("aaa")); //AAA
	
	//获取servletContext
	ServletContext servletContext = fConfig.getServletContext();
	System.out.println("init");
}
```

#### destory()方法

filter对象销毁时执行

#### doFilter方法

```java'
doFilter(ServletRequest,ServletResponse,FilterChain)
```

其中的参数：

ServletRequest/ServletResponse：每次在执行doFilter方法时 web容器负责创建一个request和一个response对象作为doFilter的参数传递进来。该request个该response就是在访问目标资源的service方法时的request和response。

FilterChain：过滤器链对象，通过该对象的doFilter方法可以放行该请求

![](https://s2.ax1x.com/2019/05/08/E6h91O.png)

### Filer的配置

url-pattern配置时：

* 完全匹配  `/sertvle1`
* 目录匹配  `/aaa/bbb/*` 使用的最多的方法

`/user/*`：访问前台的资源进入此过滤器

`/admin/*`：访问后台的资源时执行此过滤器

* 扩展名匹配  `*.txt`  、`*.jsp`

注意：url-pattern可以使用servlet-name替代，也可以混用

dispatcher：该属性指定了对某种访问方式的过滤：

REQUEST：默认值，代表直接访问某个资源时执行filter

FORWARD：转发时才执行filter

INCLUDE: 包含资源时执行filter

ERROR：发生错误时 进行跳转是执行filter
![](https://s2.ax1x.com/2019/05/08/E6hPje.png)

### Filter的作用

* 公共代码的提取

* 可以对request和response中的方法进行增强(装饰者模式/动态代理)

  这个特点用一个示例来解决get请求或者post请求乱码的问题

对于POST请求：

```java
//第一种
request.setCharacterEncoding("utf-8");

//第二种
request.setCharacterEncoding(this.getServletContext().getInitParameter("charset"));
//备注：这种获取方式是因为在web.xml中进行了如下配置
```

```xml
<!-- 设置编码 -->
<context-param>
	<param-name>charset</param-name>
	<param-value>UTF-8</param-value>
</context-param>
```

对于GET请求：

```java
public class EncodingFilter implements Filter{

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		//在传递request之前对request的getParameter方法进行增强
		/*
		 * 装饰者模式(包装)
		 * 
		 * 1、增强类与被增强的类要实现统一接口
		 * 2、在增强类中传入被增强的类
		 * 3、需要增强的方法重写 不需要增强的方法调用被增强对象的
		 */
		
		//被增强的对象
		HttpServletRequest req = (HttpServletRequest) request;
		//增强对象
		EnhanceRequest enhanceRequest = new EnhanceRequest(req);
	
		chain.doFilter(enhanceRequest, response);	
	}

	@Override
	public void destroy() { }
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException { }
}

class EnhanceRequest extends HttpServletRequestWrapper{
	
	private HttpServletRequest request;

	public EnhanceRequest(HttpServletRequest request) {
		super(request);
		this.request = request;
	}
	
	//对getParaameter增强
	@Override
	public String getParameter(String name) {
		String parameter = request.getParameter(name);//乱码
		if(value == null || value.trim().equals("")){
         	value="";
		}
		try {
			parameter = new String(parameter.getBytes("iso8859-1"),"UTF-8");
		} catch (UnsupportedEncodingException e) {
			e.printStackTrace();
		}
		return parameter;
	}
}
```

* 进行权限控制

下面是一个使用了过滤器自动登录的示例：
![](https://s2.ax1x.com/2019/05/08/E6hFnH.png)

过滤器的核心代码：（其他业务逻辑如上图所示）

```java
public class AutoLoginFilter implements Filter{

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		
		HttpServletRequest req = (HttpServletRequest) request;
		HttpServletResponse resp = (HttpServletResponse) response;
		HttpSession session = req.getSession();
		
		//获得cookie中用户名和密码 进行登录的操作
		//定义cookie_username
		String cookie_username = null;
		//定义cookie_password
		String cookie_password = null;
		//获得cookie
		Cookie[] cookies = req.getCookies();
		if(cookies!=null){
			for(Cookie cookie : cookies){
				//获得名字是cookie_username和cookie_password
				if("cookie_username".equals(cookie.getName())){
					cookie_username = cookie.getValue();
					//恢复中文用户名
					cookie_username = URLDecoder.decode(cookie_username, "UTF-8");
				}
				if("cookie_password".equals(cookie.getName())){
					cookie_password = cookie.getValue();
				}
			}
		}
		
		//判断username和password是否是null
		if(cookie_username!=null&&cookie_password!=null){
			//登录的代码
			UserService service = new UserService();
			User user = null;
			try {
				user = service.login(cookie_username,cookie_password);
			} catch (SQLException e) {
				e.printStackTrace();
			}
			//将登录的用户的user对象存到session中
			session.setAttribute("user", user);
		}
        
		//放行
		chain.doFilter(req, resp);	
	}
	
	@Override
	public void init(FilterConfig filterConfig) throws ServletException { }

	@Override
	public void destroy() { }	
}
```