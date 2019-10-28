---
title: OpenCV矩阵掩模
date: 2018-08-27 12:44:11
toc: true
categories: 图像及音视频
---

OpenCV是计算机视觉开源库，主要算法涉及图像处理和机器学习相关方法。是Intel公司贡献出来的，俄罗斯工程师贡献大部分C/C++代码。官网：https://opencv.org/ 从这里 https://opencv.org/releases.html 你可以下载到自己想要的版本！

## 环境搭建
本人使用的时VisualStudio2015+OpenCV3.4.1-vc14(vc14对应的VisualStudio版本就是VS2015)，配置环境变量，根据自己的路径来配置
![](https://s2.ax1x.com/2019/05/03/ENzMp8.png)

## 测试环境
新建VS空项目，注意是改为**`X64`**的解决方案，因为直接下载的exe实际上就是一个自解压文件，里面的库都是在X64的环境下编译好的，如何编译自己的VS对应的OpenCV参考：https://www.bilibili.com/video/av17968786，根据自己的路径配置即可！
>**包括头文件**
>D:\opencv3.4\build\include 
>D:\opencv3.4\build\include\opencv 
>D:\opencv3.4\build\include\opencv2 
>**库文件**
>D:\opencv3.4\build\x64\vc14\lib
>**链接器-附加依赖项**
>opencv_world310d.lib

![](https://s2.ax1x.com/2019/05/03/ENzG0s.png)
注意自己的图片路径，如果其他代码与图片所示一致，那么恭喜你的OpenCV开发环境搭建OK了！

## 图像有关的API
### 加载图像cv::imread
imread功能是加载图像文件成为一个Mat对象(什么是Mat对象接下来的文章中将会说到)
第一个参数表示图像文件名称(路径)
第二个参数，表示加载的图像是什么类型，支持常见的三个参数值

* IMREAD_UNCHANGED (<0) 表示加载原图，不做任何改变
* IMREAD_GRAYSCALE ( 0)表示把原图作为灰度图像加载进来
* IMREAD_COLOR (>0) 表示把原图作为RGB图像加载进来

注意：OpenCV支持JPG、PNG、TIFF等常见格式图像文件加载
例如加载灰度图：
![](https://s2.ax1x.com/2019/05/03/ENztkq.png)

### 显示图像 cv::namedWindos 与cv::imshow
namedWindos功能是创建一个OpenCV窗口，它是由OpenCV自动创建与释放，无需手动销毁它。
常见用法namedWindow("窗口标题", WINDOW_AUTOSIZE)

* WINDOW_AUTOSIZE会自动根据图像大小，显示窗口大小，不能人为改变窗口大小
* WINDOW_NORMAL,跟QT集成的时候会使用，允许修改窗口大小。

imshow根据窗口名称显示图像到指定的窗口上去，第一个参数是窗口名称，第二参数是Mat对象

### 修改图像 (cv::cvtColor)
cvtColor的功能是把图像从一个彩色空间转换到另外一个色彩空间，有三个参数，第一个参数表示源图像、第二参数表示色彩空间转换之后的图像、第三个参数表示源和目标色彩空间如：COLOR_BGR2HLS 、COLOR_BGR2GRAY 等
这是原图转换为COLOR_BGR2HLS色彩空间：
![](https://s2.ax1x.com/2019/05/03/ENzdpT.png)
这是原图转换为COLOR_BGR2GRAY 色彩空间：
![](https://img-blog.csdn.net/20180827200124960)
有时容易出现这样的错误：
![](https://s2.ax1x.com/2019/05/03/ENzInH.png)
这个问题一般是由于将已经是灰度图的图片继续转为灰度图时引起的，所以在读取图片转为灰度色彩空间的时候要保证读取到的原图是彩色图像，如果使用灰度图的参数读取那么再次灰度化的时候就会抱这种错误！

### 保存图像(cv::imwrite)
保存图像文件到指定目录路径，只有8位、16位的PNG、JPG、Tiff文件格式而且是单通道或者三通道的BGR的图像才可以通过这种方式保存，保存PNG格式的时候可以保存透明通道的图片可以指定压缩参数！
```c++
#include <iostream>
#include <opencv2\opencv.hpp>
#include <math.h>
using namespace std;
using namespace cv;

int main(int argc, char* argv[])
{
	//加载图像，默认IMREAD_UNCHANGED
	//Mat src = imread("C:/Users/Tim/Desktop/Image/a.jpg");
	Mat src = imread("C:/Users/Tim/Desktop/Image/a.jpg", IMREAD_GRAYSCALE);  //以灰度图读取

	//判断是否加载成功
	if (src.empty())
	{
		cout << "load image filed..." << endl;
		return -1;
	}

	//新建一个OpenCV窗口，大小根据图片自动调节
	namedWindow("Test Opencv", CV_WINDOW_AUTOSIZE);
	//显示图像
	imshow("Test Opencv", src);

	namedWindow("Output Window", CV_WINDOW_AUTOSIZE);
	Mat outImg;
	//色彩空间装换
	//cvtColor(src, outImg, CV_BGR2HLS);
	cvtColor(src, outImg, COLOR_BGR2GRAY);
	imshow("Output Window", outImg);

	//将Mat对象写入文件
	imwrite("C:/Users/Tim/Desktop/Image/a2.png",outImg);
	waitKey(0);
	return 0;
}
```
## 矩阵的掩膜操作
### 掩膜操作解释
所谓掩膜其实就是一个矩阵，然后根据这个矩阵重新计算图片中像素的值，掩模也叫作`kernal`
下图就是通过掩模提高图片对比度的计算公式，根据掩模把图片的对比度提高：
![](https://s2.ax1x.com/2019/05/03/ENzoBd.png)

```c++
#include <opencv2\opencv.hpp>
#include <iostream>
#include <math.h>

using namespace cv;

int main(int argc,char* argv[])
{
	Mat src;
	Mat dst;

	src = imread("C:/Users/Tim/Desktop/Image/e.jpg");
	if (!src.data)
	{
		std::cout << "load iamge filed..." << std::endl;
		return -1;
	} 

	namedWindow("input image", CV_WINDOW_AUTOSIZE);
	imshow("input image", src);

	//获取图像的列数,一定不要忘记图像的通道数
	int cols = (src.cols-1) * src.channels();
	int rows = src.rows;

	//获取通道数目
	int offsetx = src.channels();

	//生成一个和源图像大小相等类型相同的全0矩阵
	dst = Mat::zeros(src.size(),src.type());

	//获取起始时间
	double start = getTickCount();
	
	for (int row = 1; row < (rows - 1); row++)
	{
		//获取每一个通道的像素指针
		const uchar* prev = src.ptr<uchar>(row - 1);
		const uchar* cur = src.ptr<uchar>(row);
		const uchar* next = src.ptr<uchar>(row + 1);

		uchar* output = dst.ptr<uchar>(row);

		for (int col = offsetx; col < cols; col++){
			//output[col] = (5 * cur[col] - (cur[col - offsetx] + cur[col + offsetx] + prev[col] + next[col]));//注意像素值范围在0~255之间！
			output[col] = saturate_cast<uchar>(5 * cur[col] - (cur[col - offsetx] + cur[col + offsetx] + prev[col] + next[col]));
		}
	}

	//获取处理时间
	double run_time = (cvGetTickCount() - start)/getTickFrequency();
	std::cout << run_time<< std::endl;

	namedWindow("output image", CV_WINDOW_AUTOSIZE);
	imshow("output image", dst);
	waitKey(0);
	return 0;
}
```
### 获取图像像素指针

`Mat.ptr <uchar>(int i = 0) `获取像素矩阵的指针，索引i表示第几行，从0开始计行数。
获得当前行指针`const uchar*  current= myImage.ptr<uchar>(row )`;
获取当前像素点`P(row, col)`的像素值 `p(row, col) =current[col]`

### saturate_cast < uchar >
```c++
saturate_cast<uchar>（-100），返回0
saturate_cast<uchar>（288），返回255
saturate_cast<uchar>（100），返回100
```
这个函数的功能是确保RGB值得范围在0~255之间!如果不使用这个函数进行转换，将会得到如下所示的效果，这是由于部分像素点在经过公式转化之后范围已经不再0~255之间，所以需要用到此函数：
![](https://s2.ax1x.com/2019/05/03/ENzX38.png)
经过校正之后的处理效果：
![](https://s2.ax1x.com/2019/05/03/EUSMU1.png)

### filter2D实现掩膜
参数一：源图像
参数二：最终输出的图像,需要再定义一个Mat类变量
参数三：为像素深度，两个像素深度一定要相同，否则出错`src.depth()`  或者 ` -1` 
参数四：掩膜

```c++
#include <opencv2\opencv.hpp>
#include <iostream>
#include <math.h>

using namespace cv;

int main(int argc,char* argv[])
{
	Mat src;
	Mat dst;

	src = imread("C:/Users/Tim/Desktop/Image/e.jpg");
	if (!src.data)
	{
		std::cout << "load iamge filed..." << std::endl;
		return -1;
	} 

	namedWindow("input image", CV_WINDOW_AUTOSIZE);
	imshow("input image", src);

	//获取图像的列数,一定不要忘记图像的通道数
	int cols = (src.cols-1) * src.channels();
	int rows = src.rows;

	//获取通道数目
	int offsetx = src.channels();

	//生成一个和源图像大小相等类型相同的全0矩阵
	dst = Mat::zeros(src.size(),src.type());

	//获取起始时间
	double start = getTickCount();

	//定义一个掩模
	//Mat kernal = (Mat_<char>(3, 3) << 0, 1, 0, -1, 5, -1, 0, -1, 0);//对比度增强
	Mat kernal = (Mat_<char>(3, 3) << 0, -1, 0, -1, 6, -1, 0, -1, 0);//提高锐度
	filter2D(src, dst, src.depth(), kernal);

	//获取处理时间
	double run_time = (cvGetTickCount() - start)/getTickFrequency();
	std::cout << run_time<< std::endl;

	namedWindow("output image", CV_WINDOW_AUTOSIZE);
	imshow("output image", dst);
	waitKey(0);
	return 0;
}
```
### 掩模的主要应用
* 实现图像对比度调整
* 提取感兴趣区,用预先制作的感兴趣区掩模与待处理图像相乘,得到感兴趣区图像,感兴趣区内图像值保持不变,而区外图像值都为0
掩膜是一种图像滤镜的模板，实用掩膜经常处理的是遥感图像。当提取道路或者河流，或者房屋时，通过一个n*n的矩阵来对图像进行像素过滤，然后将我们需要的地物或者标志突出显示出来。这个矩阵就是一种掩膜
* 屏蔽作用,用掩模对图像上某些区域作屏蔽,使其不参加处理或不参加处理参数的计算,或仅对屏蔽区作处理或统计
* 结构特征提取,用相似性变量或图像匹配方法检测和提取图像中与掩模相似的结构特征
* 特殊形状图像的制作
### 各种掩膜的作用
3x3 
邻域平均 全是1  

3x3高斯均值滤波器 
filters(0) = 1: filters(1) = 2:filters(2) = 1 
filters(3) = 2: filters(4) = 4: filters(5) = 2 
filters(6) = 1: filters(7) = 2: filters(8) = 1  

拉普拉斯1型滤波器  高通边缘检测器掩膜 
filters(0) = -1: filters(1) = 0: filters(2) = -1 
filters(3) = 0: filters(4) = 4: filters(5) = 0 
filters(6) = -1: filters(7) = 0: filters(8) = -1  

锐化 (中锐化：filters(4) = 5 ， 高锐化：filters(4) = 6)
filters(0) = 0: filters(1) = -1: filters(2) = 0 
filters(3) = -1: filters(4) = 6: filters(5) = -1 
filters(6) = 0: filters(7) = -1: filters(8) = 0 

垂直掩膜 
filters(0) = 3: filters(1) = -6: filters(2) = 3 
filters(3) = 3: filters(4) = -6: filters(5) = 3 
filters(6) = 3: filters(7) = -6: filters(8) = 3

水平掩膜 
filters(0) = 3: filters(1) = 3: filters(2) = 3 
filters(3) = -6: filters(4) = -6: filters(5) = -6 
filters(6) = 3: filters(7) = 3: filters(8) = 3  

对角线掩膜 
filters(0) = 3: filters(1) = 3: filters(2) = -6 
filters(3) = 3: filters(4) = -6: filters(5) = 3 
filters(6) = -6: filters(7) = 3: filters(8) = 3  

高斯滤镜5x5 
f(0) = 1:   f(1) = 4:     f(2) = 6:     f(3) = 4:     f(4) = 1
f(5) = 4:   f(6) = 16:   f(7) = 24:   f(8) = 16:   f(9) = 4 
f(10) = 6: f(11) = 24: f(12) = 36: f(13) = 24: f(14) = 6 
f(15) = 4: f(16) = 16: f(17) = 24: f(18) = 16: f(19) = 4 
f(20) = 1: f(21) =  4: f(22) =  6:   f(23) = 4:   f(24) = 1

根据掩模的设定不同效果也就不同，比如上述代码中一个掩模是增强对比度，一个掩模是增强锐度：
![](https://s2.ax1x.com/2019/05/03/EUSQ4x.png)
**注意代码中的如何计算图片的处理时间，以及如何生成全0矩阵！**

> 参考：https://blog.csdn.net/weixin_40519315/article/details/79551313