---
title: NVIDIA Driver and CUDA Installation on Ubuntu
date: 2025-02-23 20:14:00 +0800
categories: [Tools]
tags: [nvidia, cuda, gpu]
image: /assets/img/posts/nvidia-cuda-toolkit/cover.jpg
---
<div style="text-align: justify">
If you are interested in AI, you must have heard about CUDA toolkit. It provides the tools, libraries, and compiler needed to develop and run programs that use NVIDIA GPUs for general-purpose computing. Many AI frameworks (eg TensorFlow, PyTorch, OpenCV, ...) use CUDA to accelerate training and inference.
</div>

![](/assets/img/posts/nvidia-cuda-toolkit/gpu.jpg)

This blog show you how to install 2 main components for CUDA computing on Ubuntu:
* NVIDIA driver
* CUDA toolkit

Let's get started!

## Environment

* Ubuntu 20.04
* GPU NVIDIA GeForce RTX 3060

### Target package version

* NVIDIA driver version **525**
* CUDA toolkit version **12.0**

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> CUDA toolkit and NVIDIA driver should satisfy the conditions described in the reference below for optimal performance.
{: .prompt-tip }
<!-- markdownlint-restore -->

Reference: <https://docs.nvidia.com/deploy/cuda-compatibility/index.html#id1>

## Install NVIDIA driver

### 1. Remove all existing NVIDIA driver​s

```
sudo apt-get remove --purge nvidia\*​
sudo apt-get autoremove​
```

### 2. Install gcc and g++

```
sudo apt install gcc
```
My `gcc --version`

![](/assets/img/posts/nvidia-cuda-toolkit/gcc.png)

My `g++ --version`

![](/assets/img/posts/nvidia-cuda-toolkit/g++.png)

### 3. Install driver

Check available NVIDIA drivers by command:
```
sudo ubuntu-drivers devices​
```

Install NVIDIA driver
```
sudo ubuntu-drivers install nvidia:525​
```

**Reboot to apply the changes**: `sudo reboot`

### 4. Verify

```
nvidia-smi
```
![](/assets/img/posts/nvidia-cuda-toolkit/nvidia-smi.png)

## Install CUDA toolkit

```
sudo apt update​
sudo apt install cuda-toolkit-12-0​
```
<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
If above comments fail, add the NVIDIA CUDA repo to your system by the following commands
{: .prompt-tip }
<!-- markdownlint-restore -->

```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-keyring_1.0-1_all.deb​

sudo dpkg -i cuda-keyring_1.0-1_all.deb​
```

## Add CUDA to path

### 1. cuda.sh
```
sudo nano /etc/profile.d/cuda.sh
```
Add the following content to `cuda.sh`
```
export PATH=/usr/local/cuda-12.0/bin${PATH:+:${PATH}} ​
export LD_LIBRARY_PATH=/usr/local/cuda-12.0/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}} ​
```

```
sudo chmod +x /etc/profile.d/cuda.sh
```

### 2. cuda.conf
```
sudo nano /etc/ld.so.conf.d/cuda.conf​
```
Write the following path to `cuda.conf`
```
/usr/local/cuda-12.0/lib64​
```

Save and reboot to apply the changes
```
sudo ldconfig​
sudo reboot
```

### 3. Verify

```
nvcc --version
```
![](/assets/img/posts/nvidia-cuda-toolkit/cuda.png)

## Simple CUDA application

Now let's run a simple CUDA app. Create a file named `hello.cu` with the following contents:
```
#include <iostream>

__global__ void helloFromGPU() {
    printf("Hello from GPU thread %d!\n", threadIdx.x);
}

int main() {
    std::cout << "Hello from CPU!" << std::endl;

    // Launch with 1 block of 5 threads
    helloFromGPU<<<1, 5>>>();

    // Wait for GPU to finish
    cudaDeviceSynchronize();

    return 0;
}
```

Build and run the app
```
nvcc hello.cu -o hello
```
```
$ ./hello
Hello from CPU!
Hello from GPU thread 0!
Hello from GPU thread 1!
Hello from GPU thread 2!
Hello from GPU thread 3!
Hello from GPU thread 4!
```

> To get started with CUDA computing, check out <https://cuda-tutorial.readthedocs.io/en/latest/tutorials/tutorial01/>

Okay that's all. Hope you get something useful from my blog!
