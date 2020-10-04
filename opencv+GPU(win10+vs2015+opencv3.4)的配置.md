# opencv+GPU(win10+vs2015+opencv3.4+)的配置<dr>
## 用到的相关软件和库<dr>
VS2015<dr>
CMake<dr>
CUDA10.1<dr>
TBB（Intel Threading Building Blocks，是为了方便程序员使用多核处理器的C++库.opencv自带的很多函数利用这个库进行了优化，可以实现一些函数的并行优化，比如opencv3中的canny函数中就有TBB加速.）<dr>
opencv3.4.6<dr>
opencv-contrib<dr>
## 配置前的准备<dr>
## CMake编译带有TBB、cuda和contrib的opencv<dr>

