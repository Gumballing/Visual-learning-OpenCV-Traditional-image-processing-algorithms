# opencv+GPU(win10+vs2015+opencv3.4)的配置

## 用到的相关软件和库
VS2015<dr>
CMake<dr>
CUDA10.1<dr>
TBB<dr>
opencv3.4.6<dr>
opencv-contrib<dr>
  
## 配置前的准备
（1）安装好VS之后安装CUDA10（必须先安装VS，再安装CUDA）<dr>
在英伟达开发者社区下载官方的CUDA10，安装的时候选择默认路径，不要更改。后续的选项全部默认就好。<dr>
安装完成后打开属性--高级系统设置--环境变量，能够看到系统变量中多了CUDA_PATH和CUDA_PATH_V9_1两个变量，然后在系统变量中添加以下变量：<dr>
```
CUDA_SDK_PATH = C:\ProgramData\NVIDIA Corporation\CUDA Samples\v9.0 
CUDA_LIB_PATH = %CUDA_PATH%\lib\x64 
CUDA_BIN_PATH = %CUDA_PATH%\bin 
CUDA_SDK_BIN_PATH = %CUDA_SDK_PATH%\bin\win64 
CUDA_SDK_LIB_PATH = %CUDA_SDK_PATH%\common\lib\x64 
```
查看是否安装成功：打开cmd，输入：nvcc -V。如果结果显示有CUDA的版本则安装成功。
（2）下载TBB<dr>
TBB，Intel Threading Building Blocks，是为了方便程序员使用多核处理器的C++库.opencv自带的很多函数利用这个库进行了优化，可以实现一些函数的并行，比如opencv3中的canny函数中就有TBB加速.<dr>
在TBB官网或者GitHub上下载TBB安装包至本地解压后，可以放在任意目录，这里我放在了D:\SOFTWARE\TBB。<dr>
添加TBB至系统环境：在系统-> 高级系统设置->环境变量->path 后边加入下边地址：D:\SOFTWARE\TBB\tbb44_20160413oss\bin\intel64\vc14（这里注意选择对应的32位 或者64 位和对应的VS版本，VS2015对应vc14）。<dr>
（3）下载opencv和opencv-contrib源码<dr>
新建一个文件夹把这两个源码文件都放进去。

## CMake编译带有TBB、cuda和contrib的opencv
打开CMAKE-gui后在source code选择opencv-source所在文件夹下的opencv文件夹。下面where to build the binaries选择一个要存放生成的opencv安装包的文件夹。<dr>
点击configure，弹出界面选择环境，这里选择VS 2015 。第一次configure done后勾选WITH_TBB 和WITH_CUDA。<dr>
在OPENCV     OPENCV_EXTRA_MODULES_PATH填上opencv_contrib的modules的路径：D:/opencv3.4-source/opencv_contrib/modules   <dr>
填写TBB路径：![](https://github.com/Gumballing/image_storage/blob/main/TBB%E8%B7%AF%E5%BE%84.jpg)    <dr>
填写完成后再次点击configure，提示configure done无报错后点击generate，生成完成后点击open project在VS中进行生成。<dr>
在VS中选择不同的编译模式，分别生成debug和release下的安装包。<dr>
DEBUG下：点击ALL_BUILD右键生成，等一会，生成成功部分，有部分生成失败，不用管。<dr>
点击INSTALL，右键，生成。此时可能会报一个错误LNK1104：无法打开文件python35_d.lib<dr>
这是因为电脑安装了Anaconda自带的python，找到ANACONDA/include/pyconfig.h文件打开后找到图中那两句注释掉，重新右键生成INSTALL，生成成功。<dr>
![](https://github.com/Gumballing/image_storage/blob/main/LINK1104.png)
在Release下也进行同样的操作，ALL_BUILD和INSTALL都生成成功无失败后再切换到Debug下重新生成ALL_BUILD和INSTALL，全部生成成功后本地的安装包即为最终的配置安装包。<dr>

## 在VS中配置GPU版的opencv
此时可以参照“OpenCV库在Windows、VS上的配置.md”文档进行一劳永逸的配置即可。<dr>

## 测试
新建一个CPP文件，使用以下代码测试配置是否成功。<dr>

```cpp
#include <opencv2\opencv.hpp>
#include <iostream>

using namespace std;
using namespace cv;
using namespace cv::cuda;

int main()
{
	/*-------------------------以下四种验证方式任意选取一种即可-------------------------*/
	//获取显卡简单信息
	cuda::printShortCudaDeviceInfo(cuda::getDevice());  //有显卡信息表示GPU模块配置成功

														//获取显卡详细信息
	cuda::printCudaDeviceInfo(cuda::getDevice());  //有显卡信息表示GPU模块配置成功

												   //获取显卡设备数量
	int Device_Num = cuda::getCudaEnabledDeviceCount();
	cout << Device_Num << endl;  //返回值大于0表示GPU模块配置成功

								 //获取显卡设备状态
	cuda::DeviceInfo Device_State;
	bool Device_OK = Device_State.isCompatible();
	cout << "Device_State: " << Device_OK << endl;  //返回值大于0表示GPU模块配置成功

	system("pause");
	return 0;
}
```
如果运行成功且输出显卡信息则GPU模块运行正常。<dr>
现在测试GPU版本的扩展模块是否能正常运行，使用一个着特征点的例子进行测试。<dr>
新建一个surf_test.cpp文件，使用以下代码：<dr>
```cpp
#include <iostream>
#include <signal.h>
#include <vector>
 
#include <opencv2/opencv.hpp>
 
using namespace cv;
using namespace std;

bool stop = false;
void sigIntHandler(int signal)
{
	stop = true;
	cout<<"Honestly, you are out!"<<endl;
}
 
int main()
{
	Mat img_1 = imread("model.jpg");
	Mat img_2 = imread("ORB_test.jpg");
	if (!img_1.data || !img_2.data)
	{
		cout << "error reading images " << endl;
		return -1;
	}
	int times = 0;
	double startime = cv::getTickCount();
	signal(SIGINT, sigIntHandler);
 
	int64 start, end;
	double time;
 
	vector<Point2f> recognized;
	vector<Point2f> scene;
 
	for(times = 0;!stop; times++)
	{
		start = getTickCount();
 
		recognized.resize(500);
		scene.resize(500);
 
		Mat d_srcL, d_srcR;
		Mat img_matches, des_L, des_R;
		cvtColor(img_1, d_srcL, COLOR_BGR2GRAY);
		cvtColor(img_2, d_srcR, COLOR_BGR2GRAY);
 
		Ptr<ORB> d_orb = ORB::create(500,1.2f,6,31,0,2);
		Mat d_descriptorsL, d_descriptorsR, d_descriptorsL_32F, d_descriptorsR_32F;
 
		vector<KeyPoint> keyPoints_1, keyPoints_2;
 
		Ptr<DescriptorMatcher> d_matcher = DescriptorMatcher::create(NORM_L2);
 
		std::vector<DMatch> matches;
		std::vector<DMatch> good_matches;
 
		d_orb -> detectAndCompute(d_srcL, Mat(), keyPoints_1, d_descriptorsL);
		d_orb -> detectAndCompute(d_srcR, Mat(), keyPoints_2, d_descriptorsR);
		d_matcher -> match(d_descriptorsL, d_descriptorsR, matches);
 
		int sz = matches.size();
		double max_dist = 0; double min_dist = 100;
 
		for (int i = 0; i < sz; i++)
		{
			double dist = matches[i].distance;
			if (dist < min_dist) min_dist = dist;
			if (dist > max_dist) max_dist = dist;
		}
 
		cout << "\n-- Max dist : " << max_dist << endl;
		cout << "\n-- Min dist : " << min_dist << endl;
 
		for (int i = 0; i < sz; i++)
		{
			if (matches[i].distance < 0.6*max_dist)
			{
				good_matches.push_back(matches[i]);
			}
		}
 
		for (size_t i = 0; i < good_matches.size(); ++i)
		{
			scene.push_back(keyPoints_2[ good_matches[i].trainIdx ].pt);
		}
 
		for(unsigned int j = 0; j < scene.size(); j++)
			cv::circle(img_2, scene[j], 2, cv::Scalar(0, 255, 0), 2);
 
		imshow("img_2", img_2);
		waitKey(1);
 
		end = getTickCount();
		time = (double)(end - start) * 1000 / getTickFrequency();
		cout << "Total time : " << time << " ms"<<endl;
 
		if (times == 1000)
		{
			double maxvalue =  (cv::getTickCount() - startime)/cv::getTickFrequency();
			cout <<"zhenshu " << times/maxvalue <<"  zhen"<<endl;
		}
		cout <<"The number of frame is :  " <<times<<endl;
	}
	return 0;
}
```

如果程序运行成功且显示图片出来，则编译配置成功。<dr>

