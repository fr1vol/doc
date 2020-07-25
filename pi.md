#### 树莓派交叉编记录

环境：
*  VMware 虚拟机 ubuntu18.04
*  树莓派3B+


安装arm编译工具链
```
sudo apt-get install gcc-arm-linux-gnueabihf
sudo apt-get install g++-arm-linux-gnueabihf
```

编译boost1.65.1版本
```
cd boost_1_65_1
./bootstrap.sh --without-libraries=python
```

修改project-config.jam，用下面替换 using gcc
```
using gcc : arm : arm-linux-gnueabihf-g++ ;
```
编译boost
```
./b2 --built-type=complete --no-samples --no-tests toolset=gcc-arm cxxflags=-fPIC -j 6
```
将生成的stage文件拷过去就ok了

-----------

编写 pi.cmake文件，[参考链接](https://www.cnblogs.com/rickyk/p/3875334.html)
```
SET(CMAKE_SYSTEM_NAME Linux)
SET(CMAKE_SYSTEM_VERSION 1)

# tool chain homedir

SET(CMAKE_C_COMPILER     /usr/bin/arm-linux-gnueabihf-gcc)
SET(CMAKE_CXX_COMPILER   /usr/bin/arm-linux-gnueabihf-g++)
SET(CMAKE_FIND_ROOT_PATH /usr/arm-linux-gnueabihf)
SET(CMAKE_FIND_ROOT_PATH_MODE_PROGRAM NEVER)
SET(CMAKE_FIND_ROOT_PATH_MODE_LIBRARY ONLY)
SET(CMAKE_FIND_ROOT_PATH_MODE_INCLUDE ONLY
```

执行
```
git clone https://github.com/jameskbride/cmake-hello-world.git 
cd cmake-hello-world
mkdir build
cd build
cmake -D CMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake ../
make
```
使用scp命令将程序拷贝到树莓派上，运行测试