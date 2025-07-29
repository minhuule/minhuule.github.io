---
title: OpenCV CUDA Installation for C++ Application on Ubuntu
date: 2025-02-25 6:45:12 +0800
categories: [Tools]
tags: [opencv, cuda, gpu]
image: /assets/img/posts/opencv-gpu/cover.jpg
---
This guide walks you through installing OpenCV with CUDA support for C++ application on Ubuntu. OpenCV is a powerful computer vision tool. By combining it with GPU acceleration via CUDA, you can significantly speed up tasks like image processing, object detection, and real-time video analysis.

## Environment

* Ubuntu 20.04
* GPU NVIDIA GeForce RTX 3060

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Must have NVIDIA driver and CUDA toolkit installed, check my previous post below on how to install them
{: .prompt-tip }
<!-- markdownlint-restore -->

> 🔗 **Post:** [NVIDIA Driver and CUDA Installation on Ubuntu](https://www.duclee.com/posts/nvidia-cuda-toolkit-installation)

## Install necessary libraries​

```
$ sudo apt-get install -y build-essential cmake pkg-config unzip yasm git checkinstall libjpeg-dev libpng-dev libtiff-dev libavcodec-dev libavformat-dev libswscale-dev libavresample-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev libxvidcore-dev x264 libx264-dev libfaac-dev libmp3lame-dev libtheora-dev libfaac-dev libmp3lame-dev libvorbis-dev libopencore-amrnb-dev libopencore-amrwb-dev libdc1394-22 libdc1394-22-dev libxine2-dev libv4l-dev v4l-utils​
```
```
$ cd /usr/include/linux
$ sudo ln -s -f ../libv4l1-videodev.h videodev.h​
$ cd​
```
```
$ sudo apt-get install -y libgtk-3-dev python3-dev python3-pip​
$ sudo -H pip3 install -U pip numpy​
$ sudo apt install -y python3-testresources​
```
```
$ sudo apt-get install -y libtbb-dev libatlas-base-dev gfortran libprotobuf-dev protobuf-compiler libgoogle-glog-dev libgflags-dev libgphoto2-dev libeigen3-dev libhdf5-dev doxygen​
```

## Get OpenCV packages

```
$ wget -O opencv.zip https://github.com/opencv/opencv/archive/4.9.0.zip
$ wget -O opencv_contrib.zip https://github.com/opencv/opencv_contrib/archive/4.9.0.zip

$ unzip opencv.zip
$ unzip opencv_contrib.zip​
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> I used OpenCV 4.9.0, but its version can be changed according to your needs
{: .prompt-tip }
<!-- markdownlint-restore -->

## Build OpenCV for C++ GPU

```
$ cd opencv-4.9.0
$ mkdir build
$ cd build
```

Setup build folder with the following command:
```
cmake -D CMAKE_BUILD_TYPE=RELEASE \
-D CMAKE_C_COMPILER=/usr/bin/gcc-9 \
-D CMAKE_INSTALL_PREFIX=/usr/local \
-D INSTALL_PYTHON_EXAMPLES=ON \
-D INSTALL_C_EXAMPLES=OFF \
-D WITH_TBB=ON \
-D WITH_CUDA=ON \
-D OPENCV_DNN_CUDA=OFF \
-D CUDA_ARCH_BIN=8.6 \
-D BUILD_opencv_cudacodec=ON \
-D ENABLE_FAST_MATH=1 \
-D CUDA_FAST_MATH=1 \
-D WITH_CUBLAS=1 \
-D WITH_V4L=ON \
-D WITH_QT=ON \
-D WITH_OPENGL=ON \
-D WITH_GSTREAMER=ON \
-D OPENCV_GENERATE_PKGCONFIG=ON \
-D OPENCV_PC_FILE_NAME=opencv.pc \
-D OPENCV_ENABLE_NONFREE=ON \
-D OPENCV_EXTRA_MODULES_PATH=~/opencv_contrib-4.9.0/modules \
-D PYTHON_EXECUTABLE=~/usr/bin/python3 \
-D BUILD_EXAMPLES=ON .. \
-D OPENCV_PYTHON_SKIP_DETECTION=ON​
```

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Check the paths in the cmake command, taking note of the following information:
{: .prompt-tip }
<!-- markdownlint-restore -->

* `CMAKE_C_COMPILER`: depending on your gcc version
* `OPENCV_EXTRA_MODULES_PATH`: correct your opencv modules path
* `CUDA_ARCH_BIN`: This part needs to be correct for the CUDA library to work. This number is the Compute capability of the GPU, which can be checked at 2 sources:​
  * <https://en.wikipedia.org/wiki/CUDA#GPUs_supported​>
  ![](/assets/img/posts/opencv-gpu/GPUs_supported​.png)

  * <https://developer.nvidia.com/cuda-gpus>
  ![](/assets/img/posts/opencv-gpu/cuda-gpus.png)

Then build with command
```
$ make
```

## Install OpenCV to system

```
$ sudo make install
```
```
$ sudo /bin/bash -c 'echo "/usr/local/lib" >> /etc/ld.so.conf.d/opencv.conf'
$ sudo ldconfig
```

## Verify with a sample application

Clone the application
```
$ wget https://github.com/niconielsen32/ComputerVision/blob/master/OpenCVcuda/gpuTest.cpp
```

Build
```
$ g++ gpuTest.cpp $(pkg-config --cflags --libs opencv) -o gpuTest
```

Run
```
./gpuTest
```

Okay it's the end. Hope you enjoyed the process and learned something new!
