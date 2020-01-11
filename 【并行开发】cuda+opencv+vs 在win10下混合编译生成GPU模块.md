### 生成Opencv GPU模块(opencv/contrib/cuda/vs/win10)


#### 环境配置

* *Visual Studio* 2015（*vs*一定要在*cuda*之前装）
* *CUDA* 10.1；安装时选择自定义安装，只勾选`CUDA`，不修改安装路径。 
* *OpenCV* 4.1.0+*OpenCV-contrib* 4.1.0
* *CMake* 3.11 ，安装最后一步勾选`Add CMAKE to the PATH for all users`。       

[ 查看本机是否支持cuda开发](https://developer.nvidia.com/cuda-gpus) || [ 下载CUDA ToolKit](https://developer.nvidia.com/cuda-toolkit) || [cuda安装教程](https://blog.csdn.net/qq_30623591/article/details/82084113) ||[github-opencv](https://github.com/opencv)    

------------------------------

#### 一、OpenCV+OpenCV Contrib库混合编译(with CUDA)

##### 1.1 下载原材料

获取`OpenCV`和`OpenCV-Contrib`库的**源码**（链接如上），两者注意选择同一版本，如`opencv4.1.0`，那么`Contrib`也要选择4.1.0。切记不要在*github*里直接选择*Clone and download*,那样版本会是最新的。注意后续配置全程退出360。

##### 1.2 *CMAKE*编译

* 打开*cmake*，*“Where is the source code"*处选`OpenCV`的`source`文件夹，*“where to”*则选择库编译的`目的文件夹`。
* 选好后点左下角*configure*，如果目的文件夹不存在会提示新建，确定即可。
* 关键一步，选择你的*VS*版本，一定要选择*WIN*64和 “*use defalut native compilers*" 。
* 点左下角*finish*，等读条完成后出现一堆红色项目，且下方输出框也没有红色信息（*download failed*）时，即**第一次编译成功**。检查窗口下方的输出框是否有*“CUDA detected：10.1”*的信息，说明*CUDA*安装成功且被检测到。
* 在红色堆堆里找`OPENCV_EXTRA_MODULES_PATH`，填写路径，就是你下载的`opencv_contrib`源码解压后存放的文件夹中的`modules`文件夹路径。
* 在红色堆堆那个窗口中找`WITH_CUDA`,勾选它！这一步也有教程说勾选*create opencv_world*的选项，没必要，这个*dll*的作用就是~~一个替代N个~~， 尤其是在个人主机上跑的，后面很可能因为创建*dll*时链接过长失败。同样的，如果想缩短编译和重新生成解决方案的时间，建议将*BUILD_EXAMPLES*去除。
* 再次点击左下角*configure*，*configure done*且没有红色，且下方输出框也没有红色信息（*download failed*）时，即**第二次编译成功**。
* 点击*generate*，完成后会在`目的文件夹`生成*VS*项目。

两次编译成功前有一句`且下方输出框也没有红色信息（*download failed*），`，这是个巨大坑：下方信息框输出大量“*download failed*”,究其原因就是没有梯子的*PC*从*github*下载一些辅助文件速度太慢失败。解决方案是自己到[github-opencv](https://github.com/opencv) 上对应模块下载文件，并放入相应文件夹中，具体参照了[B站教程:OpenCV从入门到放弃](https://www.bilibili.com/video/av17972938/?p=2)。   

##### 1.3 *VS*生成动态库

* *CMAKE*结束后，可点击左下角*Open Project*，或打开`目的文件夹`找到对应*sln*文件打开编译好的项目。
* 在*VS*里，“生成”->“配置管理器”->解决方案选择*DEBUG*，活动解决平台选择*X64*，勾选“*ALL BUILD*”和“*INSTALL*”，生成即可。或者在右侧解决方案中分别找到“*ALL BUILD*”和“*INSTALL*”，右键“仅生成”。       
* 生成成功后在`目的文件夹`下会出现一个*INSTALL*文件夹，路径*“install/x64/vc14/”*下的*bin*和*lib*两个文件夹内即生成的动态链接库文件。后续配置环境变量都要用到（lib是编译时用的，dll是运行时用的）。  

重新生成过程中如果遇到提示“某个项目外部环境已更改”，选择全部重新加载即可。配置*CUDA*的情况下，重新生成解决方案的时间很长，一般都要2-3小时，渣一点的七八个小时也有，耐心等待。如果最后的生成结果缺少1-2个lib，直接到`目的文件夹`的*module*目录下找到未成功的项目名称，打开对应的*.sln*重新生成解决方案，重新点击*INSTALL*即可。

------------------------------

#### 二、配置opencv运行环境

##### 2.1  配置系统环境变量

进入系统环境变量对话框，在*PATH*中添加生成的*bin*路径（*1.3*中的*3*步骤。）

##### 2.2  VS2015包含库路径配置

* 打开一个*VS*项目，上方工具栏“视图”-> “其他窗口”-> “属性管理器”，在*DEBUG|X64*版本下右键“属性”。

* 添加包含目录：“VC++目录”-> “包含目录”-> 选择编辑，添加*include*文件夹，具体为*“目的文件夹/install/include/”*；*“目的文件夹/install/include/opencv2* (如果有的话) ”

* 添加库目录： “VC++目录”-> “库目录”-> 选择编辑，添加路径*“install/x64/vc14/lib”* 

* 添加链接器宏定义：“链接器”->“输入”->“选择编辑”，手动敲入*“install/x64/vc14/lib”* 下的`.lib名称`。 这些库文件全是自己通过*VS2015*编译出来的。这个也是检查自己编译是否成功的标准，如果编译不成功，则没有库文件；编译成功则有需要的库文件。    

那么问题来了，要手打多少个*lib*才是头呢？ 善良笔者在线提供*.py*代码，帮助大家扒取文件名。

```python
import os  
def file_name(file_dir):   
    for root, dirs, files in os.walk(file_dir):  
        print(root) #当前目录路径  
        print(dirs) #当前路径下所有子目录  
        print(files) #当前路径下所有非目录子文件
```
------------------------------

#### 三、测试

测试OpenCV的GPU模块是否可以正确调用并启动GPU加速，测试代码如下：

```c++
//测试显卡方法1(此方法可以读取显卡型号)
cv::cuda::printShortCudaDeviceInfo(cv::cuda::getDevice());

//测试显卡方法2
int iDevicesNum = cv::cuda::getCudaEnabledDeviceCount();
cout << iDevicesNum << endl;

//测试显卡方法3
cv::cuda::DeviceInfo _deviceInfo;
bool _isDeviceOK = _deviceInfo.isCompatible();
std::cout << "IsGPUDeviceOK : " << _isDeviceOK << std::endl;`
```
如果抛出*no cuda support*的错误，可以将"*`目的文件夹`/install/x64/vc14/bin"*下的*opencv_core410d.dll*复制到*C*盘*Windows*的*system32*和*64*目录下。这一步笔者一直报错找不到某*.dll*文件，最后将全部的*.dll*复制过去才成功。

如果代码正常运行，恭喜你，混合编译生成*OpenCV*的*GPU*模块到此结束~

---------------------------

##### 附录

笔者的*.lib*目录：

`opencv_aruco410d.lib
opencv_bgsegm410d.lib
opencv_bioinspired410d.lib
opencv_calib3d410d.lib
opencv_ccalib410d.lib
opencv_core410d.lib
opencv_cudaarithm410d.lib
opencv_cudabgsegm410d.lib
opencv_cudacodec410d.lib
opencv_cudafeatures2d410d.lib
opencv_cudafilters410d.lib
opencv_cudaimgproc410d.lib
opencv_cudalegacy410d.lib
opencv_cudaobjdetect410d.lib
opencv_cudaoptflow410d.lib
opencv_cudastereo410d.lib
opencv_cudawarping410d.lib
opencv_cudev410d.lib
opencv_datasets410d.lib
opencv_dnn410d.lib
opencv_dnn_objdetect410d.lib
opencv_dpm410d.lib
opencv_features2d410d.lib
opencv_flann410d.lib
opencv_fuzzy410d.lib
opencv_gapi410d.lib
opencv_hfs410d.lib
opencv_highgui410d.lib
opencv_imgcodecs410d.lib
opencv_imgproc410d.lib
opencv_img_hash410d.lib
opencv_line_descriptor410d.lib
opencv_ml410d.lib
opencv_objdetect410d.lib
opencv_optflow410d.lib
opencv_phase_unwrapping410d.lib
opencv_photo410d.lib
opencv_plot410d.lib
opencv_quality410d.lib
opencv_reg410d.lib
opencv_rgbd410d.lib
opencv_saliency410d.lib
opencv_shape410d.lib
opencv_stereo410d.lib
opencv_stitching410d.lib
opencv_structured_light410d.lib
opencv_superres410d.lib
opencv_surface_matching410d.lib
opencv_text410d.lib
opencv_tracking410d.lib
opencv_video410d.lib
opencv_videoio410d.lib
opencv_videostab410d.lib
opencv_xfeatures2d410d.lib
opencv_ximgproc410d.lib
opencv_xobjdetect410d.lib
opencv_xphoto410d.lib`

