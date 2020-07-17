# 双目相机标定以及深度图计算
(C++,opencv)双目相机标定与深度计算，代码实现的功能就是对立体双目相机进行标定以及利用标定参数计算视差图，再从视差图转为深度图(包含了一些踩过的坑)。


## 双目相机标定(stereo_calib.cpp)
stereo_calib.cpp是opencv的例程，在"opencv-3.2.0/samples/cpp"目录下，left01.jpg--left14.jpgm,right01-jpg--right14.jpg以及stereo_calib.xml在"opencv-3.2.0/samples/data"目录下。

### 运行示例
把项目下载到本地后，首先在build目录下用cmake编译
```
cmake ..
make
```
切回data目录下，运行
```
../build/stereo_calib -w=9 -h=6 stereo_calib.xml
```
标定生成两个文件，分别是内参和外参。
### 运行自己的图片
和data平行建立文件夹data1，然后放入图片和stereo_calib.xml，并修改stereo_calib.xml中的图片名，运行同上，需要注意的是，图片的质量很重要，应尽量保证以下几点：
* 良好的光照
* 静止拍照
* 标定板清晰
* 标定板平整
如果标定的结果校正出来的图片很扭曲，甚至看不到原图的样子，可能是角点的模糊导致，继续优化拍的图片。项目的data1文件夹下有效果还好的图片。


## 深度图计算(stereo_match.cpp)
stereo_match.cpp只能给出视差图disp，且由于该视差图被量化为16个等级，所以在计算深度时需要考虑。
视觉几何的推导不必介绍，下面是计算深度需要的参数
* baseline：左右相机距离，该参数在标定的相机外参extrinsics.yml中T向量第一个值，取绝对值，单位是mm
* f：归一化焦距，由于只在x方向有视差，用fx即可，至于用左相机还是右相机，还没找到解决方法，左右相机x方向焦距差距不大时用两者平均，fx的单位是pixel，原因在[这里](https://blog.csdn.net/tercel_zhang/article/details/90523181).
* d：视差，单位为pixel，从disp中读取，除以16的原因在[这里](https://blog.csdn.net/bennygato/article/details/37704259)，读取方式为：
```
d = (float)disp.at<short int>(h,w)*0.0625;
```
公式：depth(mm) = baseline(mm) * f(pixel) / d(pixel)
深度的存储格式为float，不易以mat形式存储，一般用视差直接计算。
