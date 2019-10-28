---
title: Mat与基本图形绘制
toc: true
categories: 图像及音视频
date: 2018-08-30 15:00:00
---

##  Mat对象与IplImage对象
* Mat对象OpenCV2.0之后引进的图像数据结构、自动分配内存、不存在内存泄漏的问题，是面向对象的数据结构。分了两个部分，头部与数据部分
* IplImage是从2001年OpenCV发布之后就一直存在，是C语言风格的数据结构，需要开发者自己分配与管理内存，对大的程序使用它容易导致内存泄漏问题，下面是IplImage结构体的定义：
```c++
typedef struct _IplImage
{
        int nSize;         // IplImage大小
        int ID;            // 版本 (=0)
        int nChannels;     // 大多数OPENCV函数支持1,2,3 或 4 个通道 
        int alphaChannel; // 被OpenCV忽略 
        int depth;         // 像素的位深度，主要有以下支持格式： IPL_DEPTH_8U, IPL_DEPTH_8S, IPL_DEPTH_16U,IPL_DEPTH_16S, IPL_DEPTH_32S,IPL_DEPTH_32F 和IPL_DEPTH_64F 
        char colorModel[4]; // 被OpenCV忽略
        char channelSeq[4]; // 同上
        int dataOrder;     // 0 - 交叉存取颜色通道, 1 - 分开的颜色通道.只有cvCreateImage可以创建交叉存取图像
        int origin;        // 图像原点位置： 0表示顶-左结构,1表示底-左结构
        int align;         // 图像行排列方式 (4 or 8)，在 OpenCV 被忽略，使用 widthStep 代替
        int width;        // 图像宽像素数 
        int height;        // 图像高像素数
        struct _IplROI *roi; // 图像感兴趣区域，当该值非空时，只对该区域进行处理
        struct _IplImage *maskROI; // 在 OpenCV中必须为NULL
        void *imageId;     // 同上
        struct _IplTileInfo *tileInfo; //同上
        int imageSize;     // 图像数据大小(在交叉存取格式下ImageSize=image->height*image->widthStep），单位字节
        char *imageData;    // 指向排列的图像数据
        int widthStep;     // 排列的图像行大小，以字节为单位
        int BorderMode[4]; // 边际结束模式, 在 OpenCV 被忽略
        int BorderConst[4]; // 同上
        char *imageDataOrigin; // 指针指向一个不同的图像数据结构（不是必须排列的），是为了纠正图像内存分配准备的 

    } IplImage;
```
## Mat对象使用
### 基本概念
#### 通道
把图像分解成一个或多个颜色成分！

* 单通道：一个像素点只需一个数值表示，只能表示灰度，0为黑色
* 三通道：RGB模式，把图像分为红绿蓝三个通道，可以表示彩色，全0表示黑色
* 四通道：在RGB基础上加上alpha通道，表示透明度，`alpha=0`表示全透明
* 双通道：双通道图像不常见，通常在程序处理中会用到，如傅里叶变换，可能会用到，一个通道为实数，一个通道为虚数，主要是编程方便
![](https://s2.ax1x.com/2019/05/03/EU9X36.png)
通过画图板的各种格式可以保存出不同的类型
![](https://s2.ax1x.com/2019/05/03/EU9vjO.png)
#### 深度
深度即位数（比特数）

* 位深：一个像素点所占的总位数，也叫像素深度、图像深度等
* 位深 = 通道数 × 每个通道所占位数 
* 256色图：n位的像素点可以表示2^n种颜色，称2^n色图，n=8时为256色图
* 8位RGB与8位图：前者的位数指每个通道所占的位数，后者指整个像素点共占的位数
* 8位RGB是一个24位图，也称为真彩
### Mat对象构造函数
* `Mat()`
无参数构造方法
* `Mat(int rows, int cols, int type) `
 创建行数为 rows，列数为 col，类型为 type 的图像
* `Mat(Size size, int type) `
创建大小为 size，类型为 type 的图像
* `Mat(int rows, int cols, int type, const Scalar& s) `
创建行数为 rows，列数为 col，类型为 type 的图像，并将所有元素初始化为值s
* `Mat(Size size, int type, const Scalar& s) `
创建大小为 size，类型为 type 的图像，并将所有元素初始化为值 s
* `Mat(const Mat& m) `
将m赋值给新创建的对象，此处不会对图像数据进行复制，m和新对象共用图像数据，属于**浅拷贝**
* `Mat(int rows, int cols, int type, void* data, size_t step=AUTO_STEP) `
创建行数为rows，列数为col，类型为type的图像，此构造函数不创建图像数据所需内存，而是直接使用data所指内存，图像的行步长由 step指定
* `Mat(Size size, int type, void* data, size_t step=AUTO_STEP) `
创建大小为size，类型为type的图像，此构造函数不创建图像数据所需内存，而是直接使用data所指内存，图像的行步长由step指定
* `Mat(const Mat& m, const Range& rowRange, const Range& colRange) `
创建的新图像为m的一部分，具体的范围由rowRange和colRange指定，此构造函数也不进行图像数据的复制操作，新图像与m共用图像数据
* `Mat(const Mat& m, const Rect& roi) `
创建的新图像为m的一部分，具体的范围roi指定，此构造函数也不进行图像数据的复制操作，新图像与m共用图像数据

例如：`Mat M(2,2,CV_8UC3, Scalar(0,0,255))`
这些构造函数中，很多都涉及到类型type。type可以是`CV_8UC1`，`CV_16SC1`，`CV_64FC4` 等其中前两个参数分别表示行(row)跟列(column)、第三个`CV_8UC3`中的8表示每个通道占8位、U表示无符号、C表示Char类型、3表示通道数目是3，第四个参数是向量表示初始化每个像素值是多少，向量长度对应通道数目一致
如果你需要更多的通道数，需要用宏 `CV_8UC(n)` ，例如： 
`Mat M(3,2, CV_8UC(5))`  创建行数为 3，列数为 2，通道数为 5 的图像。
### Mat对象常用方法
* 部分复制：一般情况下只会复制Mat对象的头和指针部分，不会复制数据部分
```c++
Mat A= imread("XXX");
Mat B(A)  // 只复制头信息，浅拷贝
```
* 完全复制：如果想把Mat对象的头部和数据部分一起复制，可以通过如下两个API实现
```c++
Mat F = A.clone();  
Mat G;
A.copyTo(G);
```
Mat对象分为头部与数据部分，赋值操作和拷贝构造函数只会复制头部，要想用深拷贝只能使用`clone()`  、 ` copyTo(Mat dst,int type)` !
```c++
#include <iostream>
#include <opencv2\opencv.hpp>

using namespace cv;
using namespace std;

int main(int argc,char **argv)
{
	Mat src = imread("C:/Users/Tim/Desktop/Image/a.jpg");

	if (src.empty())
	{
		std::cout << "load image filed" << std::endl;
		return -1;
	}

	namedWindow("input window",CV_WINDOW_AUTOSIZE);
	imshow("input window", src);

	//通过ctreat函数创建
	Mat m_c;
	m_c.create(4, 4, CV_8UC2);
	cout << m_c << endl;

	//定义了一个dst对象的时候只是创建了Mat对象的头部
	Mat dst;
	dst = Mat(src.size(), src.type());
	dst = Scalar(127, 0, 255);

	namedWindow("output window", CV_WINDOW_AUTOSIZE);
	imshow("output window", dst);

	//都是深拷贝
	dst = src.clone();
	src.copyTo(dst);

	//获取通道数
	cvtColor(src, dst, CV_BGR2GRAY);
	cout << "src.channels():" << src.channels() << endl;
	cout << "dst.channels():" << dst.channels() << endl;

	//获取首行像素指针
	const uchar* firstRow = dst.ptr<uchar>(0);

	//获取行像素、列像素
	int row = dst.rows;
	int col = dst.cols;

	//创建行数为 rows，列数为 col，类型为 type 的图像，并将所有元素初始化
	Mat m(5, 5, CV_8UC3, Scalar(0, 0, 255));
	cout << m << endl;

	namedWindow("smail", CV_WINDOW_AUTOSIZE);
	imshow("smail", m);


	//定义小数组
	Mat C = (Mat_<double>(3, 3) << 0, -1, 0, -1, 5, -1, 0, -1, 0);
	cout << C << endl;
	
	waitKey(0);
	return 0;
}
```
![](https://s2.ax1x.com/2019/05/03/EU9zuD.png)
这是创建的5*5的矩阵
![](https://s2.ax1x.com/2019/05/03/EUCCEd.png)

#### 关于Scalar
查看源码opencv3源码， 发现Scalar做成了模板类，其中有如下构造函数：可以看到，Scalar是一个由长度为4的数组作为元素构成的结构体，Scalar最多可以存储四个值，没有提供的值默认是0。
Scalar常用的使用场景如下：
```c++
Mat M(7,7,CV_32FC2,Scalar(1,3));
```
上面的代码表示：创建一个2通道，且每个通道的值都为（1, 3），深度为32，7行7列的图像矩阵。CV_32F表示每个元素的值的类型为32位浮点数，C2表示通道数为2，Scalar（1,3）表示对矩阵每个元素都赋值为（1, 3），第一个通道中的值都是1，第二个通道中的值都是3.
`dst = Scalar(127, 0, 255)` 上述代码中颜色转换过来就是这样的：
![](https://s2.ax1x.com/2019/05/03/EUCEgf.png)

### Mat对象使用-四个要点
* 输出图像的内存是自动分配的
* 使用OpenCV的C++接口，不需要考虑内存分配问题
* 赋值操作和拷贝构造函数只会复制头部分
* 使用clone与copyTo两个函数实现数据完全复制
## 绘制形状与文字
### Point与Scalar
Point表示2D平面上一个点，其成员就是 x,y 坐标
Scalar表示四个元素的向量，表示 RGB 三个通道
### 代码
```c++
#include <iostream>
#include <opencv2\opencv.hpp>

using namespace cv;
using namespace std;

Mat bgImage;
void MyLines();
void MyRectangle();
void MyEllipse();
void MyCircle();
void MyPolyon();
void Random();
int main(int argc, char **argv){

	bgImage = imread("C:\\Users\\Tim\\Desktop\\Image\\a.jpg");
	if (bgImage.empty()){
		cout << "load image filed..." << endl;
		return -1;
	}
	MyLines();

	MyRectangle();

	MyEllipse();

	MyCircle();

	MyPolyon();

	//绘制文字
	//背景图、文字、起始点、字体（前提是系统必须支持设定的字体）、字体放大倍数、颜色、线粗、线条类型
	putText(bgImage, "HelloWorld", Point(30, 50), CV_FONT_HERSHEY_COMPLEX, 2.0, Scalar(0, 0, 255), 2, 8);
	namedWindow("src_window", CV_WINDOW_AUTOSIZE);
	imshow("src_window", bgImage);

	//Random();

	waitKey(0);
	return 0;
}

//绘制线条
void MyLines(){
	Point p1 = Point(0, 0);
	Point p2 = Point(410, 624);
	Scalar color = Scalar(0, 0, 255);
	//背景图、直线两头坐标、颜色、线粗、
	//line(bgImage, p1, p2, color, 4, LINE_8);
	//line(bgImage, p1, p2, color, 4, LINE_4);
	line(bgImage, p1, p2, color, 4, LINE_AA);//LINE_AA是无锯齿
}

//绘制矩形
void MyRectangle(){
	//起始坐标点，宽和高
	Rect rect = Rect(120, 20, 200, 200);
	Scalar color = Scalar(0, 255, 0);
	rectangle(bgImage, rect, color, 2, LINE_AA);
}

//绘制椭圆
void MyEllipse(){
	Scalar color = Scalar(255, 0, 0);
	//背景图、中心点坐标、长轴和短轴、椭圆的倾斜度、（0-360就是绘制完整椭圆）、颜色、线粗、无锯齿
	ellipse(bgImage, Point(bgImage.cols / 2, bgImage.rows / 2), Size(bgImage.cols / 4, bgImage.rows / 8), 45, 0, 180, color, 2, LINE_AA);
}

//绘制圆
void MyCircle(){
	Scalar color = Scalar(255, 255, 0);
	//背景图、圆心、半径
	Point center = Point(bgImage.cols / 2, bgImage.rows / 2);
	circle(bgImage, center, 150, color, 2, LINE_8);
}

//绘制多边形
void MyPolyon(){
	//定义好多边形顶点的二维数组
	Point pts[1][5];
	pts[0][0] = Point(100, 100);
	pts[0][1] = Point(120, 180);
	pts[0][2] = Point(220, 200);
	pts[0][3] = Point(150, 80);
	pts[0][4] = Point(100, 100);

	const Point* ppts[] = { pts[0] };
	int npt[] = { 5 };
	Scalar color = Scalar(0, 255, 255);
	fillPoly(bgImage, ppts, npt, 1, color, 8);
}

//绘制随机线条
void Random(){
	RNG rng(12345);//设置随机种子
	Point pt1;
	Point pt2;
	Mat bg = Mat::zeros(bgImage.size(), bgImage.type());
	namedWindow("random", CV_WINDOW_AUTOSIZE);

	for (size_t i = 0; i <10000; i++)
	{
		//确保随机数的范围
		pt1.x = rng.uniform(0, bg.cols);
		pt2.x = rng.uniform(0, bg.cols);
		pt1.y = rng.uniform(0, bg.rows);
		pt2.y = rng.uniform(0, bg.rows);

		waitKey(100);
		Scalar color = Scalar(rng.uniform(0, 255), rng.uniform(0, 255), rng.uniform(0, 255));
		line(bg, pt1, pt2, color, 1, 8);
		imshow("random", bg);
	}
}
```
![](https://s2.ax1x.com/2019/05/03/EUPSzV.png)
随机线条
![](https://s2.ax1x.com/2019/05/03/EUP9MT.gif)