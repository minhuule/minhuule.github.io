---
title: C++ Object Detection Yolov7 Tensorrt
date: 2025-07-28 10:28:25 +0800
categories: [Machine Learning]
tags: [C++, object detection, yolov7, machine learning, nvidia, cuda, tensorrt]
image: /assets/img/posts/yolov7-tensorrt-cpp/cover.png
---

<div style="text-align: justify">
If you’re working on real-time applications, detection speed is just as important as accuracy. Running YOLOv7 directly in C++ with TensorRT can significantly boost performance compared to traditional approaches, such as using OpenCV’s built-in object detection APIs or running PyTorch models without optimization. This setup allows you to achieve much higher FPS (frames per second) and smoother real-time performance, especially in embedded systems like NVIDIA Jetson devices or GPU-powered servers. Now let's begin!
</div>

## Environment

First, make sure you have NVIDIA GPU with CUDA supported. Here is my environment:
* Ubuntu 20.04
* NVIDIA RTX 3060

Second, install NVIDIA CUDA Toolkit and OpenCV C++. Check out my blog on how to install them.

* [🚀 NVIDIA Driver and CUDA Installation on Ubuntu](https://www.duclee.com/posts/nvidia-cuda-toolkit-installation/)
* [🚀 OpenCV CUDA Installation for C++ Application on Ubuntu](https://www.duclee.com/posts/opencv-c++-gpu-installation)
* TensorRT - Download from [nvidia-tensorrt-8x-download](https://developer.nvidia.com/nvidia-tensorrt-8x-download). Select tar packages depending on your environment. In my case I get Linux x86_64 and CUDA 12.0.
![](/assets/img/posts/yolov7-tensorrt-cpp/tensorrt.png)

## Source code
Clone the repository: [duclee1509/yolov7-tensorrt-cpp](https://github.com/duclee1509/yolov7-tensorrt-cpp.git)
```
git clone https://github.com/duclee1509/yolov7-tensorrt-cpp.git
```
```
export PROJECT_PATH="<path to yolov7-tensorrt-cpp folder>"
```

## Tensorrtx Yolov7
### 1. Build yolov7
In `CMakeLists.txt`, update your TensorRT links depending on the packages download location.

```
cd ${PROJECT_PATH}/yolov7
mkdir build && cd build
cmake ..
```
![](/assets/img/posts/yolov7-tensorrt-cpp/tensorrtx-config.png)

```
make
```
![](/assets/img/posts/yolov7-tensorrt-cpp/tensorrtx-build.png)

### 2. Detect using yolov7-tiny.pt

Generate wts weight
```
cd ${PROJECT_PATH}/yolov7/WongKinYiu_yolov7
python3 gen_wts.py -w ${PROJECT_PATH}/weights/yolov7-tiny.pt -o ${PROJECT_PATH}/weights/yolov7-tiny.wts
```

Generate engine weight
```
cd ${PROJECT_PATH}/yolov7/build
./yolov7 -s ${PROJECT_PATH}/weights/yolov7-tiny.wts ${PROJECT_PATH}/weights/yolov7-tiny.engine t
```
![](/assets/img/posts/yolov7-tensorrt-cpp/yolov7-engine.png)

**Note**: `t` stand for **tiny**, If you use different model architecture, change this argument.
```
cd ${PROJECT_PATH}/yolov7/build
./yolov7
```
![](/assets/img/posts/yolov7-tensorrt-cpp/yolov7-args.png)

Detect
```
cd ${PROJECT_PATH}/yolov7/build
./yolov7 -d ${PROJECT_PATH}/weights/yolov7-tiny.engine ${PROJECT_PATH}/yolov7/images/common/
```
![](/assets/img/posts/yolov7-tensorrt-cpp/yolov7-detect.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> The first inference time always high, it decreases from the second time onwards. This can be fixed in my APIs below.
{: .prompt-tip }
<!-- markdownlint-restore -->

Let's see the result `output/_bus.jpg`

![](/assets/img/posts/yolov7-tensorrt-cpp/bus.jpg)

### 3. Detect using custom yolov7 model
You can use my google colab example `colab/train-yolov7-object-detection-on-custom-data.ipynb` to build your own yolov7 custom model. In my case, I build 

Generate wts weight
```
cd ${PROJECT_PATH}/yolov7/WongKinYiu_yolov7
python3 gen_wts.py -w ${PROJECT_PATH}/weights/lp_character.pt -o ${PROJECT_PATH}/weights/lp_character.wts
```

**Important**: Remember to change `kNumClass` to number of classes of your model in `yolov7/include/config.h`, in my case it is 32.
![](/assets/img/posts/yolov7-tensorrt-cpp/config.png)

Rebuild yolov7
```
cd ${PROJECT_PATH}/yolov7/build
make
```

Generate engine weight
```
cd ${PROJECT_PATH}/yolov7/build
./yolov7 -s ${PROJECT_PATH}/weights/lp_character.wts ${PROJECT_PATH}/weights/lp_character.engine t
```
![](/assets/img/posts/yolov7-tensorrt-cpp/lp-engine.png)

Detect license plate images
```
cd ${PROJECT_PATH}/yolov7/build
./yolov7 -d ${PROJECT_PATH}/weights/lp_character.engine ${PROJECT_PATH}/yolov7/images/license_plate/
```
![](/assets/img/posts/yolov7-tensorrt-cpp/lp-detect.png)

Check the result `output/_1.jpg`

![](/assets/img/posts/yolov7-tensorrt-cpp/1.jpg)

<div style="text-align: justify">
You may understand that the characters on the image are detected with random numbers. But not, when build, yolov7 uses a file to define the order of classes. So that the numbers showed on detected characters is the order of that character in the classes file.
</div>

## API Yolov7 Controller

Based on **Tensorrtx Yolov7** source code, I developed APIs and CMake build system to simplify the use of this object detection model.

* controller/yolov7controller.cpp
* controller/yolov7controller.h
* yolov7/src/modelAPI.cpp
* yolov7/include/modelAPI.h
* CMakeLists.txt

And `main.cpp` implements a simple usage of my custom model: license plate character detection.

Now let's build it
```
cd ${PROJECT_PATH}
mkdir build
cd build
cmake ..
make
```

Run the executable
```
cd ${PROJECT_PATH}/build
./Yolov7_Object_Detection
```
![](/assets/img/posts/yolov7-tensorrt-cpp/api-run.png)

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> By running a dummy inference to initialize TensorRT buffers and CUDA stream, the inference time is low at the first time detection.
{: .prompt-tip }
<!-- markdownlint-restore -->

Based on `classes/character.names`, I was able to map the detected character to its correct name. **Output**: `output.jpg`

![](/assets/img/posts/yolov7-tensorrt-cpp/output.jpg)

## Conclusion

Running YOLOv7 with TensorRT in C++ is one of the best ways to speed up object detection for embedded devices. Hope you learn something from my blog. See ya!
