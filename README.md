# 服务器搭建说明
功能：1. 目标探测SSD训练
      2. 实例分割训练
      3. 标注数据质检

## 硬件要求
英伟达显卡NVIDIA 1080性能以上

建议硬件配置

硬件 | 型号
---|---
处理器 | Intel Core i7-6950X @ 3.00GHzX10
内存 | 12X10G
硬盘 | 1T
显卡 | NVIDIA GP102 TITAN XP X 2~4


## 基础工具安装
系统运行在Linux Ubuntu 16.04 64-bit
或Linux Mint 18.3 Cinnamon 64-bit

更新系统，注意：以下操作不作说明，则都是在root用户权限下进行操作
```
apt update
apt upgrade
```
安装通用软件包
```
apt install 
apt install cmake cmake-curses-gui
apt install autoconf libtool
apt install libleveldb-dev libsnappy-dev libhdf5-serial-dev liblmdb-dev
apt install libatlas-base-dev libgflags-dev libgoogle-glob-dev
apt install --no-install-recommends libboost-all-dev
apt install rapidjson-dev libproj-dev libssl-dev libfreetype6-dev
apt install screen colordiff
apt install imagemagick ffmpeg
```
安装python相关软件包
```
apt install python-pip python-dev python-tk
pip install --upgrade pip
```
## CUDA8.0 安装
下载CUDA 8.0的本地安装包
```
wget https://developer.nvidia.com/compute/cuda/8.0/Prod2/local_installers/cuda_8.0.61_375.26_linux-run
sh cuda_8.0.61_375.26_linux.run
```
本步骤安装时，在提示是否安装NVIDIA Accelarated Graphics Driver时，请选择yes；在提示是否安装opengl时，请选择no。其他都选择yes。

如果系统之前安装过NVIDIA显卡的驱动，在安装CUDA时，提示是否安装NVIDIA显卡驱动时，请选择否。

另外，先安装NVIDIA显卡驱动，后安装CUDA8.0的情况下，有时候会出现以下问题
1. 循环登陆，也就是登陆之后再退出来到登陆界面
2. 界面变得很大，分辨率降低
3. 登陆并显示正常，但是只有桌面背景和鼠标。

通常需要卸载原有驱动后从安装CUDA 8.0开始。

安装完成后，运行
```
nvidia-smi
deviceQuery
```
如果输出
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 384.111                Driver Version: 384.111                   |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  TITAN Xp            Off  | 00000000:05:00.0  On |                  N/A |
| 23%   39C    P0    62W / 250W |    350MiB / 12186MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
```
验证安装成功，有时需要重启系统后运行上述语句。

如果nvidia-smi运行缓慢，需要运行以下语句设置persistence模式
```
nvidia-persistenced --persistence-mode
```
请随时注意参考英伟达官方安装指南[NVIDIA](https://developer.nvidia.com/cuda-80-ga2-download-archive)，如果不是使用最新的显卡，使用系统自带的硬件驱动更新管理即可满足使用需求。

## cuDNN 6.0 安装
下载[cuDNN](https://developer.nvidia.com/rdp/cudnn-archive)
1. cuDNN v6.0 Runtime Library for Ubuntu16.04 (Deb)
2. cuDNN v6.0 Developer Library for Ubuntu16.04 (Deb)
3. cuDNN v6.0 Code Samples and User Guide for Ubuntu16.04 (Deb)

安装
```
dpkg -i libcudnn6_6.0.21-1+cuda8.0_amd64.deb
dpkg -i libcudnn6-dev_6.0.21-1+cuda8.0_amd64.deb
dpkg -i libcudnn6-doc_6.0.21-1+cuda8.0_amd64.deb
```

## （optional）TensorRT 2.1 安装
TensorRT 用于使用NVIDIA显卡设备的前向运算inference加速，如果不需要可以不安装

下载[tensorRT](https://developer.nvidia.com/nvidia-tensorrt-download)
注意，请下载TensorRT 2.1 for Ubuntu 16.94 and CUDA8.0 deb local 安装包

安装
```
dpkg -i nv-gie-repo-ubuntu1604-ga-cuda8.0-trt2.1-20170614_1-1_amd64.deb
apt update
apt install tensorrt-2.1.2
```

## NCCL 2.0 安装
下载[NCCL](https://developer.nvidia.com/nccl/nccl2-download-survey)正确版本， NCCL v2，for CUDA 8.0, August 30, 2017

安装
```
dpkg -i nccl-repo-ubuntu1604-2.0.5-ga-cuda8.0_2-1_amd64.deb
apt update
apt install libnccl2 libnccl-dev
```

## (optional) NVIDIA Docker 1.0.1 安装
如果不使用Docker版本的DIGITS 6.0,则可以不安装Docker

安装Docker CE
```
apt update
apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
apt update
apt install docker-ce
```
确认安装成功
```
docker run hello-world
```
下载[NVIDIA DOCKER](https://github.com/NVIDIA/nvidia-docker/releases)
安装
```
wget https://github.com/NVIDIA/nvidia-docker/releases/download/v1.0.1/nvidia-docker_1.0.1-1_amd64.deb
dpkg -i nvidia-docker_1.0.1-1_amd64.deb
```

在下载并安装Docker过程中，有时候会遇到Docker服务器下载速度过慢的情况，可以配置成阿里云服务器或清华、中科大的服务器进行下载，配置方法自行搜索。

## Protobuf /GRPC 安装
Protobuf
```
git clone https://github.com/google/protobuf.git
cd protobuf
git checkout v3.1.0
./autogen.sh
./configure
make -j4
make check -j4
make install
```
在/usr/local/lib路径确认libprotobuf*.so等正确安装

GRPC
```
git clone https://github.com/grpc/grpc.git
cd grpc
git checkout v1.1.0
git submodule update --init
make -j4
make install
ldconfig
```
在/usr/loca/lib路径确认libgrpc*.so等正确安装

## OpenCV
Version 3.1.0和Version 3.2.0都可以，3.2.0与CUDA8更融洽
```
git clone https://github.com/opencv/opencv_contrib.git
cd opencv_contrib
git checkout 3.2.0
git clone https://github.com/opencv/opencv.git
cd opencv
git checkout 3.2.0
```
CMake opencv
```
mkdir build
cd build
ccmake-gui ..
```
按照下表设置
```
CUDA_ARCH_BIN=6.1
CUDA_ARCH_PTX=6.1
OPENCV_EXTRA_MODULES=/path/to/opencv_contrib/modules/
WITH_CUBLAS=ON
WITH_CUDA=ON
WITH_CUFFT=ON
WITH_EIGEN=ON
WITH_GTK=ON
WITH_GTK_2_X=OFF
WITH_JPEG=ON
WITH_OPENCL=OFF
WITH_OPENCLAMDBLAS=OFF
WITH_OPENCLAMDFFT=OFF
WITH_OPENCL_SVM=OFF
WITH_TBB=ON
```
配置+生成（configure+generate)
```
cd build
cmake ..
make -j4
make install
```
在/usr/local/lib/路径下确认libopencv_*.so 等文件的存在，尤其需要检查libopencv_tracking.so

----------------------------------
## 分界线
下面的设置请根据实际情况针对某一个用户设置，尽量不要使用root用户安装

## SSD Caffe
SSD Caffe是用来进行目标探测的caffe版本
1. 到 C:\NavinfoSign\03详细设计和代码\01 源代码\项目预研\SSDsigndetect 下载evovision-ssd.tar.gz 到用户home路径下解压
2. 如果是为学习练习SSD使用，请下载SSD的标准版本
```
git clone https://github.com/weiliu89/caffe.git
```
以上2种方式下载的SSD，按照如下方式配置安装
```
cd ssd
cd caffe
mkdir build
cd build
cmake-gui ..
```
检查如下设置
```
BLAS=Atlas
BUILD_matlab=OFF
BUILD_python=ON
BUILD_python_layer=ON
CMAKE_INSTALL_PREFIX=/path/to/ssd/caffe/build/install
CPU_ONLY=OFF
OpenCV_DIR=/usr/local/share/OpenCV
USE_CUDNN=ON
USE_LMDB=ON
USE_OPENCV=ON
python_version=2
```
然后Configure + Generate
```
make -j4
make install
```
确保bins和libs在路径/path/to/ssd/caffe/build/install
设置PYTHONPATH的环境变量，指向/path/to/ssd/caffe/python,可以在.bashrc, .bash_aliases, .envrc 或着其他地方进行设置
例如在.bashrc下进行设置的时候
```
vim ~/.bashrc
```
在文档最后添加
```
export PYTHONPATH=/path/to/ssd_caffe/python:$PYTHONPATH
export CAFFE_ROOT=/path/to/ssd_caffe/caffe:$CAFFE_ROOT
```
保存后
```
source ~/.bashrc
```
使环境变量生效

## Dilation Caffe
Dilation caffe是用来进行语义分割的caffe版本
1. 到 C:\NavinfoSign\03详细设计和代码\01 源代码\项目预研\LaneSegment 下载semanticsegmentation.zip到用户路径解压
2. 学习目的，请下载标准版本Dilation caffe
```
git clone https://github.com/fyu/caffe-dilation.git
```
通过以上2种方式下载的caffe，按照如下说明进行安装
```
cd caffe-dilation
mkdir build
cd build
cmake-gui ..

```
检查如下设置
```
BLAS=Atlas
BUILD_matlab=OFF
BUILD_python=ON
BUILD_python_layer=ON
CMAKE_INSTALL_PREFIX=/path/to/semanticsegmentation/caffe-dilation/build/install
CPU_ONLY=OFF
OpenCV_DIR=/usr/local/share/OpenCV
USE_CUDNN=ON
USE_LMDB=ON
USE_OPENCV=ON
python_version=2
```
然后Congfigure + Generate
```
make -j4
make install
```
确保bins和libs在路径/path/to/ssd/caffe/build/install
设置PYTHONPATH的环境变量，指向/path/to/ssd/caffe/python,可以在.bashrc, .bash_aliases, .envrc 或着其他地方进行设置

注意：如果同时安装了SSD和Dilation Caffe，在使用对应的caffe版本时，需要更改PYTHONPATH

## （optional）标注及标注质检工具
到C:\NavinfoSign\03详细设计和代码\01 源代码\项目预研\相关工具 路径下载signsframework.zip并解压
```
mkdir build
cd build
ccmake-gui ..
```
根据下表进行设置
```
BUILD_ANNOTATIONTOOL=ON
BUILD_***=OFF
CAFFEDIR=/path/to/ssd/caffe/build/install
CAFFE_INCLUDE_DIR=/path/to/ssd/caffe/build/install/include
CAFFE_LIBS_DIR=/path/to/ssd/caffe/build/install/lib
JETSON_ARX=OFF
OpenCV_DIR=/usr/local/share/OpenCV
USE_PTGREY=OFF
WITH_CUDA=OFF
WITH_QT=OFF
```
然后Configure + Generate
```
make -j4
```
## DIGITS 6.0 安装
必要软件安装
```
apt-get install --no-install-recommends git graphviz python-dev python-flask python-flaskext.wtf python-gevent python-h5py python-numpy python-pil python-pip python-scipy python-tk
```

配置好的CAFFE

下载DIGITS
```
DIGITS_ROOT=~/digits
git clone https://github.com/NVIDIA/DIGITS.git $DIGITS_ROOT
```
安装PyPI软件包
```
sudo pip install -r $DIGITS_ROOT/requirements.txt
```

运行DIGITS
```
./digits-devserver
```
[DIGITS使用指导](https://github.com/NVIDIA/DIGITS/blob/master/docs/BuildDigits.md)
