# CMakeLists.txt 文件的书写

## 1 使用CMake编译

​			首先，要在工程文件夹中插入CMakeLists.txt文件并编写好，同时新建build文件夹。
​		然后，在build文件夹下（注：不是工程文件夹！）打开命令行窗口，并依次输入：

```
cmake ..
make
```

​		即完成编译。然后回到工程文件夹，输入`.\filename`运行生成的可执行文件。

## 2 CMakeLists.txt 基本格式

```cmake
camke_minimun_required( VERSION 3.4.5)	#声明要求的cmake最低版本

project(post_fusion)	#声明一个工程文件夹名为post_fusion的工程

include_directories( "/usr/include/eigen3" )	#添加引用的第三方头文件

aux_source_directory( src DIR_SRCS )	#添加源文件目录

set(TEST_MATH ${DIR_SRCS})	#设置环境变量

add_library(irfusion comfunc.c post_sins_gnss.cpp)	#将这些源文件编译成一个叫"irfusion"的静态库。
add_library(irfusion_shared SHARED comfunc.c post_sins_gnss.cpp)	#将这些源文件编译成一个叫"irfusion"的共享库。
#静态库以.a作后缀，共享库以.so结尾。静态库每次被调用都会生成一个副本，而共享库只有一个副本

add_executable(irfusion main.cpp)	#添加可执行程序
target_link_libraries(irfusion irfusion_shared)	#将可执行程序链接到irfusion_shared.so库
```

注：set()常用设置项：

set( CMAKE_CXX_FLAGS "-std=c++11")	#添加c++11标准支持

set(CMAKE_BUILD_TYPE "Debug")	#设置编译器编译格式
#Debug模式用于编译，程序运行较慢，但可以在IDE中进行断点调试；Release模式速度较快，但是没有调试信息。不设置则默认为Debug模式

























