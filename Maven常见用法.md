---
title: Maven常见用法 
date: 2019-01-28 21:21:57
categories: 工程自动化
toc: true
---

首先，我浮夸的标题就是为了吸引一下广大读者了，废话不多说，下面开始从零说起：

# 前言
如果你经常使用第三方jar包的话（前提是你现在不会Maven），那么那些jar包可能会让我们有点难受，下面是一个示例：
![在这里插入图片描述](20190128150655175.png)
假设你的工程需要这么多的Jar包（当然可能远远不止这些），那么Jar是非常难以维护的，有些Jar包又依赖于另一些Jar包，版本也不一样，更难受的是如果你的电脑是**固态硬盘、固态硬盘、固态硬盘、**，那么容量就不像机械硬盘那么阔气，于是本人想了一个好办法，就是把需要要用到的Jar包放在一起，要用的时候就去复制粘贴到项目的lib中，右键`add build path` 简直完美！！！

不过后来又出现了很多问题，比如突然搞不清楚Jar包的版本了，Jar包之间的依赖记混了，不小心删除了Jar包又得去下载(某某网站下载的需要积分不说，而且乱七八糟的下载站你懂得......动不动就是全家桶)，所以这是我最开始选择Maven的第一个理由，当然Maven的其他好处后面再说，如果你遇到和我一样的问题，那么Maven是一种你非常值得选择的方案

# 什么是Maven？
其实，我在没接触Maven这种东西之前都是一个项目就是一个工程，就像这样：
![在这里插入图片描述](20190128155748138.png)
生产环境下开发不再是一个项目一个工程，而是每一个模块创建一个工程，而多个模块整合在一起就需要使用到像Maven这样的**构建工具**。
Maven 是干什么用的？这是我刚开始接触 Maven 时最大的问题。之所以会提出这个问题，是因为即使不使用 Maven 我们仍然可以进行 B / S 结构项目的开发。从表述层、业务逻辑层到持久化层再到数据库都有成熟的解决方案，这也就意味着：不使用 Maven 我们一样可以开发项目
![在这里插入图片描述](20190128160127560.png)

# Maven的作用
看到上面的框架是不是感觉Maven不在其中，是的，因为Maven不是用来辅助编码的，Maven是为了工程模块化开发，这个概念有点笼统，所以下面介绍的比较详细，Maven的出现是为了解决如下问题（这些都是具体问题）：

## 1、添加第三方Jar包
在今天的JavaEE 开发领域，有大量的第三方框架和工具可以供我们使用。要使用这些 Jar 包最简单的方法就是复制粘贴到`WEB-INF/lib`目录下。但是这会导致每次创建一个新的工程就需要将jar包重复复制到该目录下，从而造成工作区中存在大量重复的文件，让我们的工程显得很臃肿。而使用 Maven 后每个Jar包本身只在本地仓库中保存一份，需要 Jar 包的工程只需要以坐标的方式简单的引用一下就可以了。不仅极大的节约了存储空间，让项目更轻巧，更避免了重复文件太多而造成的混乱。
## 2、解决Jar包之间的依赖关系
jar包往往不是孤立存在的，很多 jar 包都需要在其他 jar 包的支持下才能够正常工作，我们称之为jar包的依赖关系。你不可能知道所有 jar 包的依赖关系吗？当你拿到一个新的从未使用过的 jar 包，你如何得知他需要哪些 jar 包的支持呢？如果不了解这个情况，份入的 jar 包不够，那么现有的程序将不能正常工作。如果你的工程里面有1000个jar包，这简直是不可想象的。而引入Maven后，Maven就可以替我们自动的将当前 jar 包所依赖的其他所有jar包全部导入进来，无而人工参与，节约了我们大量的时间和精力。

## 3、获取第三方jar包
JavaEE开发中需要使用到的jar包种类繁多，几乎每个jar包在其本身的官网上获取方式都不尽相同。为了查找一个jar包找遍互联网，以不规范的方式获取到的jar包也是不规范的。使用 Maven我们可以享受到一个完全统一规范的jar包管理体系 。你只需要 在你的项目中以坐标的方式依赖一个jar包， Maven就会自动从就会自动从中央仓库进行 下载，并同时下载这个jar包所依赖的其他jar包！

## 4、项目模块化
JavaEE 项目的规模越来越庞大，开发团队的规模也与日俱增。几百上千的人开发的项目是同一个工程。那么架构师、项目经理该如何划分项目的模块、如何分工呢？这么大的项目必须划分模块，必须将项目拆分成多个工程协同开发。多个模块工程中有的是Java工程，有的是Web工程，此时就需要Maven

# 什么是构建？
构建就是以我们编写的Java代码、框架配置文件、国际化等其他资源文件、JSP页
面和图片等静态资源作为原材料，去生产出一个可以运行的项目的过程。通俗地讲就是：**我们用一堆原材料去制作一个可运行起来的工程！**
工程运行起来后还需要把源代码一起放在服务器上面吗？当然不能！！！
## 关于构建的概念
①清理：删除以前的编译结果，为重新编译做好准备。
②编译：将Java源程序编译为字节码文件。
③测试：针对项目中的关键点进行测试，确保项目在迭代开发过程中关键点的正确性。
④报告：在每一次测试后以标准的格式记录和展示测试结果。
⑤打包：将一个包含诸多文件的工程封装为一个压缩文件用于安装或部署。Java工程对应jar包，Web工程对应 war包。
⑥安装：在 Maven环境下特指将打包的结果——jar包或war包安装到本地仓库中。
⑦部署：将打包的结果部署到远程仓库或将 war包部署到服务器上运行。

## 自动化构建
其实上述环节我们在  Eclipse中都可以找到对应的操作，只是不太标准。那么既然IDE已经可以进行构建了我们为什么还要使用 Maven这样的构建工具呢？

现在假设你就职于某公司，早上收邮件说你的代码有BUG，于是你开始解决这个BUG：
![在这里插入图片描述](20190128162838801-20190612213215174.png)
但是实际上你把时间花在了编译、打包、部署、测试等固定流程上面，解决问题的核心在于分析问题和编码，这是非常浪费时间的，于是出现了Maven这款自动化构建工具帮你完成编译、打包、部署、测试等固定流程，这就是自动化构建：
![在这里插入图片描述](20190128163225378-20190612213215162.png)

# Maven核心概念
## 1、约定的目录结构
下图是一个典型的Maven项目的结构：
![在这里插入图片描述](20190128163546102.png)
约定>配置>编码。意思就是能用配置解决的问题就不编码，能基于约定的就不进行配置。而Maven正是因为指定了特定文件保存的目录才能够对我们的Java工程进行自动化构建。

## 2、POM
POM全称是 Project Object Model：项目对象模型。将Java工程的相关信息封装为对象作为便于操作和管理的模型。
**Maven工程的核心配置，可以说学习Maven就是学习 pom.xml文件的配置。**

## 3、坐标
在一个平面中使用 x、y两个向量可以唯一的确定平面中的一个点。在空间中使用 x、y、z三个向量可以唯一的确定空间中的一个点。Maven的坐标就是要确定唯一的一个Maven工程！使用如下三个向量在 Maven的仓库中唯一的确定一个Maven工程：
* groupid：公司或组织的域名倒序+当前项目名称
* artifactId：当前项目的模块名称
* version：当前模块的版本

![在这里插入图片描述](20190128164208475-20190612213215141.png)
如何通过坐标到仓库中查找jar包？
![在这里插入图片描述](20190128165345100.png)
我们自己的 Maven工程必须执行安装操作才会进入仓库。安装的命令是：mvn  install

## 4、依赖管理
Maven中最关键的部分，我们使用Maven最主要的就是使用它的依赖管理功能。要理解和掌握Maven的依赖管理，我们只需要解决一下几个问题：
### ① 依赖的目的是什么
当Ajar包用到了Bjar包中的某些类时，A就对B产生了依赖，这是概念上的描述。那么如何在项目中以依赖的方式引入一个我们需要的jar包呢？

```xml
<!-- 引入一个servlet-api的jar包 -->
<dependency>
		<groupId>javax.servlet</groupId>
		<artifactId>servlet-api</artifactId>
		<version>2.5</version>
		<scope>provided</scope>
</dependency>
```
### ② 依赖的范围是什么
大家注意到上面的依赖信息中除了目标jar包的坐标还有一个scope设置，这是依赖的范围。依赖的范围有几个可选值，我们用得到的是：compile、test、provided三个。
![在这里插入图片描述](20190128170309113-20190612213215357.png)
举个例子，如上图所示，很显然假设我们需要引入junit包，但是junit包确实测试需要用的包，但是junit包在测试只是在测试的时候需要，编译的时候既需要编译主程序，也要编译测试的程序，所以compile掌控的范围也就宽了，Test范围的东西只需要在测试方面接入就可以了！

compile和provided的区别的区别是什么呢？
![在这里插入图片描述](2019012817055094.png)
如上图所示，开发Web工程的时候我们需要开发环境中就要有servlet-api的jar包，也就是给工程添加的server-runtime环境：
![在这里插入图片描述](20190128170757972.png)
但是项目部署的时候是Tomcat提供的server-runtime环境，所以provided的东西虽然在编译和测试的时候是必不可少的，但是部署的时候却由Servlet容器提供！

### ③ 有效性总结
|          | compile | test | provided |
| -------- | ------- | ---- | -------- |
| 主程序   | √       | ×    | √        |
| 测试程序 | √       | √    | √        |
| 参与部署 | √       | ×    | ×        |
### ④ 依赖的传递性
A依赖B，B依赖C，A能否使用C呢？那要看B依赖C的范围是不是compile，如果是则可用，否则不可用。
![在这里插入图片描述](20190128171540456-20190612213215305.png)

### ⑤ 依赖的排除
如果我们在当前工程中引入了一个依赖是A，而A又依赖了B，那么Maven会自动将A依赖的B引入当前工程，但是个别情况下B有可能是一个不稳定版，或对当前工程有不良影响。这时我们可以在引入A的时候将B排除：
```xml
<dependency>
	<groupId>com.xpu.maven</groupId>
	<artifactId>Hello</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<type>jar</type>
	<scope>compile</scope>
	<exclusions>
		<exclusion>
			<groupId>commons-logging</groupId>
			<artifactId>commons-logging</artifactId>
		</exclusion>
	</exclusions>
</dependency>
```
很明显，我们在使用`<exclusions>`标签的时候发现，其实`<exclusions>`标签就是排除不稳定的依赖，通过`groupId`和`artifactId`就可以定位到这个依赖，`version`标签不在其中，当然就是排除它的所有版本咯！！！
### ⑥ 统一管理所依赖jar包的版本
对同一个框架的一组jar包最好使用相同的版本。为了方便升级框架，可以将jar包的版本信息统一提取出来
* 【1】 先声明一个统一的版本号
```xml
<!-- 其中xpu.spring.version部分是自定义标签 -->
<properties>
	<xpu.spring.version>4.1.1.RELEASE</xpu.spring.version>
</properties>
```

* 【2】应用前面声明的版本号
```xml
<dependency>
	<groupId>org.springframework</groupId>
	<artifactId>spring-core</artifactId>
<version>${xpu.spring.version}</version>
```
### ⑦ 依赖的原则：解决 jar包冲突
【1】路径最短者优先
![在这里插入图片描述](20190128172631602.png)
【2】路径相同时先声明者优先
![在这里插入图片描述](20190128172725976-20190612213215524.png)
注意：这里“声明”的先后顺序指的是 dependency标签配置的先后顺序

## 5、仓库管理
### 仓库的分类
本地仓库：为当前本机电脑上的所有Maven工程服务
远程仓库：
* 私服：架设在当前局域网环境下，为当前局域网范围内的所有 Maven工程服务
![在这里插入图片描述](20190128172955485.png)
* 中央仓库：架设在 Internet上，为全世界所有Maven工程服务
* 中央仓库的镜像：架设在各个大洲，为中央仓库分担流量。减轻中央仓库的压力，同时更快的响应用户请求

### 库中的文件
* Maven的插件
* 我们自己开发的项目的模块
* 第三方框架或工具的 jar包


不管是什么样的 jar包，在仓库中都是按照坐标生成目录结构，所以可以通过统一的方式查询或依赖

## 6、生命周期
Maven生命周期定义了各个构建环节的执行顺序，有了这个清单，Maven就可以自动化的执行构建命令了。


Maven有三套相互独立的生命周期，分别是：
①Clean Lifecycle在进行真正的构建之前进行一些清理工作。
②Default Lifecycle构建的核心部分，编译，测试，打包，安装，部署等等。
③Site Lifecycle生成项目报告，站点，发布站点

它们是相互独立的，你可以仅仅调用 clean来清理工作目录，仅仅调用  site来生成站点。当然你也可以直接运行  `mvn clean install site`运行所有这三套生命周期
每套生命周期都由一组阶段(Phase)组成，我们平时在命令行输入的命令总会对应于一个特定的阶段。比如，运行`mvn clean`，这个 clean是  Clean生命周期的一个阶段。有 Clean生命周期，也有clean阶段:

### Clean生命周期
Clean生命周期一共包含了三个阶段：

① pre-clean执行一些需要在clean之前完成的工作
② clean移除所有上一次构建生成的文件
③ post-clean执行一些需要在clean之后立刻完成的工作

### Site生命周期
① pre-site执行一些需要在生成站点文档之前完成的工作
② site生成项目的站点文档
③ post-site执行一些需要在生成站点文档之后完成的工作，并且为部署做准备
④ site-deploy将生成的站点文档部署到特定的服务器上


这里经常用到的是site阶段和  site-deploy阶段，用以生成和发布   Maven站点，这可是  Maven相当强大的功能，Manager比较喜欢，文档及统计数据自动生成，很好看

### Default生命周期
Default生命周期是  Maven生命周期中最重要的一个，绝大部分工作都发生在这个生命周期中。这里，只解释一些比较重要和常用的阶段：

`process-resources` 复制并处理资源文件，至目标目录，准备打包。
`compile` 编译项目的源代码
`process-test-resources` 复制并处理资源文件，至目标测试目录
`test-compile` 编译测试源代码
`test` 使用合适的单元测试框架运行测试。这些测试代码不会被打包或部署
`package` 接受编译好的代码，打包成可发布的格式，如JAR。
`install` 将包安装至本地仓库，以让其它项目依赖。
`deploy` 将最终的包复制到远程的仓库，以让其它开发人员与项目共享或部署到服务器上运行

### 生命周期与自动化构建
运行任何一个阶段的时候，它前面的所有阶段都会被运行，例如我们运行  `mvn  install`的时候，代码会被编译，测试，打包。这就是  Maven为什么能够自动执行构建过程的各个环节的原因。此外，Maven的插件机制是完全依赖Maven的生命周期的，因此理解生命周期至关重要！


## 7、插件和目标
* Maven的核心仅仅定义了抽象的生命周期，具体的任务都是交由插件完成的。

* 每个插件都能实现多个功能，每个功能就是一个插件目标。

* Maven的生命周期与插件目标相互绑定，以完成某个具体的构建任务


compile就是插件  maven-compiler-plugin的一个目标；pre-clean是插件   maven-clean-plugin的一个目标。

## 8、继承
由于非 compile范围的依赖信息是不能在"依赖链"中传递的，所以有需要的工程只能单独配置，假设三个过程都要依赖junit4.0.0.jar这个jar包，此时如果项目需要将各个模块的 junit版本统一更新为 4.9，那么到各个工程中手动修改无疑是非常不可取的。使用继承机制就可以将这样的依赖信息统一提取到父工程模块中进行统一管理：


【1】创建父工程
创建父工程和创建一般的 Java工程操作一致，唯一需要注意的是：打包方式处要设置为 pom


【2】在子工程中引用父工程
```xml
<parent>
<!--父工程坐标    -->
	<groupId>...</groupId>
	<artifactId>...</artifactId>
	<version>...</version>
	<relativePath>从当前目录到父项目的 pom.xml文件的相对路径</relativePath>
</parent>
```
注意：relativePath是从当前目录到父项目的 pom.xml文件的相对路径
【3】在父工程中管理依赖
```xml
<dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>junit</groupId>
			<artifactId>junit</artifactId>
			<version>4.9</version>
			<scope>test</scope>
		</dependency>
	</dependencies>
</dependencyManagement>
```
在子项目中重新指定需要的依赖，删除范围和版本号:
```xml
<dependencies>
	<dependency>
		<groupId>junit</groupId>
		<artifactId>junit</artifactId>
	</dependency>
</dependencies>
```

## 9、聚合
### 什么是聚合？
将多个工程拆分为模块后，需要手动逐个安装到仓库后依赖才能够生效。修改源码后也需要逐个手动进行clean操作。而使用了聚合之后就可以批量进行Maven工程的安装、清理工作，这就是聚合！
### 如何配置聚合？
在总的聚合工程中使用 modules/module标签组合，指定模块工程的相对路径即可：
```xml
<modules>
	<module>../Project_01</module>
	<module>../Project_02</module>
	<module>../Project_03</module>
	<module>...</module>
</modules>
```
## 10、如何找到我们需要的Jar包的依赖配置？
点击这里进行搜索即可☞[mvnrepository](https://mvnrepository.com/)
![在这里插入图片描述](20190128175619238.png)

# Maven的下载和eclipse/IDEA配置
【1】首先下载Maven，去阿帕奇官网下载就可以了
【2】将压缩包解压，注意解压路径不要有中文或者空格
【3】配置环境变量，你可以配置MAVEN_HOME，也可以M2_HOME
【3】新建一个文件夹作为本地仓库，注意路径不要有中文或者空格
【4】修改Maven的conf目录下的setting.xml，将本地仓库配置到指定的目录,下面是我主要的配置
```xml
<?xml version="1.0" encoding="UTF-8"?>

<!-- 设置本地仓库 -->
<localRepository>D:\software\RepMaven</localRepository>

<!-- 配置阿里云仓库 -->
<mirror>
	<id>alimaven</id>
	<name>aliyun maven</name>
	<url>http://maven.aliyun.com/nexus/content/groups/public/</url>
	<mirrorOf>central</mirrorOf>
</mirror>

<!-- 默认web工程为jdk1.8 -->
<profile>
	<id>jdk-1.8</id>
	<activation>
		<activeByDefault>true</activeByDefault>
		<jdk>1.8</jdk>
	</activation>
	<properties>
		<maven.compiler.source>1.8</maven.compiler.source>
		<maven.compiler.target>1.8</maven.compiler.target>
		<maven.compiler.compilerVersion>1.8</maven.compiler.compilerVersion>
	</properties>
</profile>
```
【5】进入eclipse直接添加即可，如果不够详细，请参考：[《Maven安装与配置》](https://www.cnblogs.com/eagle6688/p/7838224.html)
