---
title: Validate做表单校验
date: 2018-08-23 12:13:19
toc: true
categories: 前端
---
用户在注册的时候，会有个表单页面，然后有些选项是必填的，有些要填的内容是有规范的，这些都要在用户提交之前进行验证才行，如果不符合要求，需要在右边显示一个友好的提示，让用户修改。此时Validate做表单校验的特效药就发挥作用了！如果我们自己来写js代码的话，那么可就显得有些冲复造轮子了，所以现在Validate这个库可以很好的帮我们省略自己去写js校验代码，Validate这个库属于jQuery的插件库！

## Validate下载
Validate的官网里面有示例代码，看这个就可以**：https://jqueryvalidation.org ** 官网的介绍是：这个jQuery插件简化了客户端表单验证，同时还提供了大量的自定义选项。如果您正在从头开始构建校验代码，而且当您尝试将某些内容集成到具有大量现有代码的现有应用程序中时，它将是一个不错的选择。该插件捆绑了一组有用的验证方法，包括URL和电子邮件验证，同时提供API来编写自己的方法。所有捆绑方法都带有英语的默认错误消息，并翻译成其他37种语言。由此可见它的强大之处了！

## Validate特点

* 内置验证规则：拥有必填、数字、email、url和信用卡号码等19类内置验证规则
* 自定义验证规则：可以很方便的自定义验证规则
* 简单强大的验证信息提示：默认了验证信息提示，并提供自定义覆盖默认提示信息的功能
* 实时验证：可以通过keyup或bulr事件触发验证，而不仅仅在表单提交的时候验证

## Validate使用
```html
<!--注意导入的顺序-->
<script type="text/javascript" src="js/jquery-1.8.3.js" ></script>
<script type="text/javascript" src="js/jquery.validate.min.js" ></script>
<script type="text/javascript" src="js/messages_zh.js" ></script>
```

这是我的简易示例表单：
![](https://s2.ax1x.com/2019/04/30/EGRGE6.png)

```html
<!DOCTYPE html>
<html>
  <head>
      <meta charset="UTF-8">
      <title>Validate使用示例</title>
      <!--注意导入的顺序-->
      <script type="text/javascript" src="js/jquery-1.8.3.js" ></script>
      <script type="text/javascript" src="js/jquery.validate.min.js" ></script>
      <script type="text/javascript" src="js/messages_zh.js" ></script>
      <script>
          $(function(){
              $("#checkForm").validate({
                  rules:{
                      username:{
                          required:true,//不能为空
                          minlength:5,//长度必须大于等于5
                          maxlength:10,//最大长度是10
                      },
                      password:{
                          required:true,
                          digits:true,
                          minlength:6,
                      },
                      password2:{
                          required:true,
                          digits:true,
                          equalTo:"[name='password']"
                      }
                  },
                  messages:{
                      username:{
                          required:"用户名不能为空",
                          minlength:"用户名不得少于5个字符",
                          maxlength:"用户名最多10个字符"
                      },
                      password:{
                          required:"密码不能为空",
                          digits:"密码必须是整数",
                          minlength:"密码不得少于6位"
                      },
                      password2:{
                          required:"确认密码不能为空",
                          equalTo:"两次输入密码不一致"
                      }
                  }
              });
          });
      </script>
  </head>
  <body>
    <form action="#" id="checkForm">
      <table border="0px">
        <tr>
        <td>用户名</td>
        <td><input type="text"  name="username"/><br></td>
        </tr>
        <tr>
          <td>密码</td>
          <td><input type="password" name="password"/></td>
          <td></td>
        </tr>
        <tr>
          <td>确认密码</td>
          <td><input type="password" name="password2"/></td>
        </tr>
        <tr>
          <td>邮箱</td>
          <td><input type="email" name="email"></td>
        </tr>
        <tr>
          <td><input type="submit" value="提交" /></td>
        </tr>
      </table>
    </form>
  </body>
</html>
```
下面是校验效果：
![](https://s2.ax1x.com/2019/04/30/EGRY4O.png)
重点就在于设定校验的规则，常见的属性：  


## 属性相关设定
|属性|释义|
|-|-|
|required:true|必须输入的字段。|
|remote:"check.php"|使用 ajax 方法调用 check.php 验证输入值。|
|email:true|必须输入正确格式的电子邮件。|
|url:true|必须输入正确格式的网址。|
|date:true|必须输入正确格式的日期。日期校验 ie6 出错，慎用。|
|dateISO:true|必须输入正确格式的日期（ISO），例如：2009-06-23，1998/01/22。只验证格式，不验证有效性。|
|number:true|必须输入合法的数字（负数，小数）。|
|digits:true|必须输入整数。|
|creditcard:|必须输入合法的信用卡号。|
|equalTo:"#field"|输入值必须和 #field 相同。|
|accept:|输入拥有合法后缀名的字符串（上传文件的后缀）。|
|maxlength:5|输入长度最多是 5 的字符串（汉字算一个字符）。|
|minlength:10|输入长度最小是 10 的字符串（汉字算一个字符）。|
|rangelength:[5,10]|输入长度必须介于 5 和 10 之间的字符串（汉字算一个字符）。|
|range:[5,10]|输入值必须介于 5 和 10 之间。|
|max:5|输入值不能大于 5。|
|min:10|输入值不能小于 10。|
