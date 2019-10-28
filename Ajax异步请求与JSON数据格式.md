---
title: Ajax异步请求与JSON数据格式
date: 2017-12-21 17:39:50
toc: true
categories: 前端
---

![](https://s2.ax1x.com/2019/05/08/E6fBkt.jpg)

百度的预搜索是怎么实现的呢？如下图：

![](https://s2.ax1x.com/2019/05/08/E6fKm9.gif)
这个场景应该是大家非常熟悉的吧，为什么我们没有点击搜索但是却可以弹出相关的搜索内容条目呢？其中就用到了ajax引擎！接下来我们就可以看一下这个ajax，哈哈！

## 一、Ajax概述
### 什么是同步，什么是异步
同步现象：客户端发送请求到服务器端，当服务器返回响应之前，客户端都处于等待卡死状态
异步现象：客户端发送请求到服务器端，无论服务器是否返回响应，客户端都可以随	意做其他事情，不会被卡死

### Ajax的运行原理
页面发起请求，会将请求发送给浏览器内核中的Ajax引擎，Ajax引擎会提交请求到	服务器端，在这段时间里，客户端可以任意进行任意操作，直到服务器端将数据返回	给Ajax引擎后，会触发你设置的事件，从而执行自定义的js逻辑代码完成某种页面功能。

## 二、js原生的Ajax技术
js原生的Ajax其实就是围绕浏览器内内置的Ajax引擎对象进行学习的，要使用js原生的Ajax完成异步操作，有如下几个步骤：
* 创建Ajax引擎对象
* 为Ajax引擎对象绑定监听（监听服务器已将数据响应给引擎）
* 绑定提交地址
* 发送请求

下面是一个使用原生Ajax的示例：
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript">
	//异步请求
	function fn1(){
		//(1)创建引擎对象
		var xmlhttp = new XMLHttpRequest();
		
		//(2)绑定监听
		xmlhttp.onreadystatechange = function(){
			//(5)接受相应数据
			if(xmlhttp.readyState == 4 && xmlhttp.status == 200){
				var res = xmlhttp.responseText;
				document.getElementById("span1").innerHTML = res;
			}
		}
		
		//(3)绑定地址
		xmlhttp.open("GET", "/WEB21/ajaxservlet", true);
		
		//(4)发送请求
		xmlhttp.send();
	}
	
	//同步请求
	function fn2(){
		//(1)创建引擎对象
		var xmlhttp = new XMLHttpRequest();
		
		//(2)绑定监听
		xmlhttp.onreadystatechange = function(){
			//(5)接受相应数据
			if(xmlhttp.readyState == 4 && xmlhttp.status == 200){
				var res = xmlhttp.responseText;
				document.getElementById("span2").innerHTML = res;
			}
		}
		
		//(3)绑定地址
		xmlhttp.open("GET", "/WEB21/ajaxservlet", false);
		
		//(4)发送请求
		xmlhttp.send();
	}
</script>
</head>
<body>
	<input type="button" value="异步访问服务器" onclick="fn1()"/><span id="span1"></span>
	<br>
	<input type="button" value="同步访问服务器" onclick="fn2()"/><span id="span2"></span>
	<br>
	<input type="button" value="测试按钮" onclick="alert()"/>
</body>
</html>
```
servlet如下：
```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
		// TODO Auto-generated method stub
		//response.getWriter().write("XPU");
		try {
			Thread.sleep(5000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		response.getWriter().write(Math.random()+"");
	}
```
这样的话会得到下图所示的效果：
![](https://s2.ax1x.com/2019/05/08/E6f3Y6.gif)
现象很简单，我的测试按钮其实就是弹出一个空的提示框，当我点击异步访问的时候，浏览器会向服务器的servlet发送一个请求，这个请求是获取一个随机数，而且为了模拟服务器长达3秒的运算我让服务器睡眠了3秒钟，点击异步访问后可以立马点击测试按钮弹出提示框，但是点击同步访问后立马点击测试按钮却要等待3秒才会弹出提示框，而且我点击了多次就弹出了多次提示框，相信这个额例子是非常容易理解同步和异步的特点的！

为什么要这样做呢？有时候我们在加载网页的时候，可能有些图片非常大，在网速不是很好的情况下需要很长的时间才可以加载，如果使用ajax引擎发起对图片的异步请求，即使在图片还没有加载完毕的情况下也可以使用其他的功能，这就是异步请求的一个应用，接下来用图片说明一下：
![](https://s2.ax1x.com/2019/05/08/E6fYlD.png)
很显然，如果没有ajax引擎的情况下发起请求而且等到收到响应对于浏览器来说是非常浪费时间的，尤其是发起的请求计算量过大，网速特别慢的时候是非常影响用户体验的，有了ajax引擎替我们发起请求和接受协议，浏览器就有机会去做其他的事情，而不是傻傻的等待！

原生ajax的GET与POST请求：
GET 请求比较简单，如下格式即可（上面的代码中用的就是GET请求）

```javascript
xmlhttp.open("GET","test1.txt",true);
xmlhttp.send();
```
但是使用GET请求可能的到的是缓存结果，为了避免这样的情况出现，应该使用如下示例设置不同的ID：
```javascript
xmlhttp.open("GET","demo_get.asp?t=" + Math.random(),true);
xmlhttp.send();
```
简单地POST请求：
```javascript
xmlhttp.open("POST","demo_post.asp",true);
xmlhttp.send();
```
带参数的POST请求(**切记不要忘记添加请求头**）：
```javascript
xmlhttp.open("POST","ajax_test.asp",true);
xmlhttp.setRequestHeader("Content-type","application/x-www-form-urlencoded");
xmlhttp.send("fname=Bill&lname=Gates");
```

最后说一个关于获取XMLHttpRequest 对象的问题：
创建 XMLHttpRequest 对象：所有现代浏览器（IE7+、Firefox、Chrome、Safari 以及 Opera）均内建 XMLHttpRequest 对象。

### XMLHttpRequest 对象三个重要的属性
| 属性               | 描述                                                         |
| :----------------- | :----------------------------------------------------------- |
| onreadystatechange | 存储函数（或函数名），每当 readyState 属性改变时，就会调用该函数。 |
| readyState         | 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。0: 请求未初始化 1: 服务器连接已建立 2: 请求已接收 3: 请求处理中 4: 请求已完成，且响应已就绪 |
| status             | 200: "OK" 404: 未找到页面                                    |
创建 XMLHttpRequest 对象的语法：
```javascript
variable=new XMLHttpRequest();
```
老版本的 Internet Explorer （IE5 和 IE6）使用 ActiveX 对象：
```javascript
variable=new ActiveXObject("Microsoft.XMLHTTP");
```
为了应对所有的现代浏览器，包括 IE5 和 IE6，请检查浏览器是否支持 XMLHttpRequest 对象。如果支持，则创建 XMLHttpRequest 对象。如果不支持，则创建 ActiveXObject ：
```javascript
var xmlhttp;
if (window.XMLHttpRequest){
	// code for IE7+, Firefox, Chrome, Opera, Safari
	xmlhttp=new XMLHttpRequest();
}else{
	// code for IE6, IE5
	xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
}
```

使用XMLHttpRequest对象用于在后台与服务器交换数据，应用场景：
* 在不重新加载页面的情况下更新网页
* 在页面已加载后从服务器请求数据
* 在页面已加载后从服务器接收数据
* 在后台向服务器发送数据

所有异步访问都是靠ajax引擎！
## 三、JSON数据格式
json是一种与语言无关的数据交换的格式，作用：
* 使用ajax进行前后台数据交换
* 移动端与服务端的数据交换

### Json的格式与解析
json有两种格式：
1）对象格式：{"key1":obj,"key2":obj,"key3":obj...}
2）数组/集合格式：[obj,obj,obj...]

例如：user对象 用json数据格式表示
```json
{"username":"zhangsan","age":28,"password":"123","addr":"北京"}
```
例如：List<Product> 用json数据格式表示
```json
[{"pid":"10","pname":"小米4C"},{},{}]
```
注意：对象格式和数组格式可以互相嵌套
注意：json的key是字符串  jaon的value是Object

json的解析：
json是js的原生内容，也就意味着js可以直接取出json对象中的数据

### Json的转换插件
将java的对象或集合转成json形式字符串

json的转换插件是通过java的一些工具，直接将java对象或集合转换成json字符串。
常用的json转换工具有如下几种：
1）jsonlib
2）Gson：google
3）fastjson：阿里巴巴
4）cJSON：腾讯的

## 四、Jquery的Ajax技术
jquery是一个优秀的js框架，自然对js原生的ajax进行了封装，封装后的ajax的操	作方法更简洁，功能更强大，与ajax操作相关的jquery方法有如下几种，但开发中经常使用的有三种 ：
```javascript
$.get(url, [data], [callback], [type])
$.post(url, [data], [callback], [type])
```
url：代表请求的服务器端地址
data：代表请求服务器端的数据（可以是key=value形式也可以是json格式）
callback：表示服务器端成功响应所触发的函数（只有正常成功返回才执行）
type：表示服务器端返回的数据类型（jquery会根据指定的类型自动类型转换）
常用的返回类型：text、json、html等

```javascript
$.ajax( { option1:value1,option2:value2... } );
```
async：是否异步，默认是true代表异步
data：发送到服务器的参数，建议使用json格式
dataType：服务器端返回的数据类型，常用text和json
success：成功响应执行的函数，对应的类型是function类型
type：请求方式，POST/GET
url：请求服务器端地址

下面是一个使用示例：
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
</head>
<script type="text/javascript" src="jquery-1.11.3.min.js"></script>
<script type="text/javascript">
	function fn1() {
		//get异步访问
		$.get(
				"/WEB21/ajaxservlet",//url地址
				{"name":"邹长林","age":20},//请求参数
				function(data){ //成功后的回调函数
					alert(data.firstname+" "+data.lastname+" "+data.age);
				},
				"json"
			);
	}
	
	function fn2() {
		$.post(
				"/WEB21/ajaxservlet",//url地址
				{"name":"邹长林","age":20},//请求参数
				function(data){ //成功后的回调函数
					alert(data.firstname+" "+data.lastname+" "+data.age);
				},
				"json"
			);
	}
	
	function fn3() {
		$.ajax({
			url:"/WEB21/ajaxservlet", //请求的地址
			type:"GET", //请求类型
			async:true, //是否同步，默认同步
			data:{"name":"tim", "age":18}, //请求的数据，JSON格式
			success:function(data){ //请求成功的回调函数
				alert(data.firstname);
			},
			error:function(){ //请求失败的回调函数
				alert("请求失败");
			},
			dataType:"json",//从服务器接受返回的数据类型，一般为JSON或者text
			
		});
	}
</script>
<body>
	<input type="button" value="GET访问服务器" onclick="fn1()"/>
	<span id="span1"></span>
	<br>
	<input type="button" value="POST访问服务器" onclick="fn2()"/>
	<span id="span2"></span>
	<br>
	<input type="button" value="Ajax访问服务器" onclick="fn3()"/>
</body>
</html>
```
在使用Ajax三种请求的时候需要注意的地方：
GET方式提交的数据到服务器可能会出现乱码，使用编解码的方式就可以解决
POST方式提交的数据已经经过ajax处理了，无需我们再自己处理一遍
获取数据的时候的乱码问题也是很好解决的：
```java
request.setCharacterEncoding("UTF-8");
```