[![CodeFactor](https://www.codefactor.io/repository/github/architector1324/easycl/badge)](https://www.codefactor.io/repository/github/architector1324/easycl)
[![Donate with Bitcoin](https://en.cryptobadges.io/badge/micro/1SrJvWx4MD1D7SPV236swT5jYJhug86tX)](https://en.cryptobadges.io/donate/1SrJvWx4MD1D7SPV236swT5jYJhug86tX)

# Easy Computing Library

## Overview
***EasyCL*** is an open heterogeneous computing library (*GPL v 3*) based on the [***OpenCL***](https://www.khronos.org/opencl/) specification. It's header-only library.

It is designed for easy and convenient use, at the same time optimized. It is not built on the OpenCL object wrapper from [**Khronos**](https://www.khronos.org/), but uses the original [***The OpenCL 1.2 C Specification***](https://www.khronos.org/registry/OpenCL/specs/opencl-1.2.pdf).

The library allows you to bypass some of the inconveniences of the original *OpenCL*, providing work with **abstractions** of different levels, allows you to bypass the **restriction of hard binding** of arguments to the kernel, kernels to the program, programs to the final device.

* At the moment, only OpenCL 1.2 is supported

## Installation
 1) Install OpenCL library on your system
 1) Clone the repo `$ git clone https://github.com/architector1324/EasyCL`
 2) Make the dynamic library `$ ./make.sh`
 3) Install `$ sudo ./install.sh`

## Abstractions
### Arguments
It is an abstraction of the arguments of the kernel of a program. Once created, the argument can be used in different kernels and in different OpenCL programs.
```c++
ecl::GPUArgument argument(void* pointer, size_t array_size, cl_mem_flags mem_type)
```

To change the pointer to the memory area of the host at any time during program execution:
```c++
argument.setPtr(void* pointer)
```
To change the buffer size at any time during program execution:
```c++
argument.setArrSize(size_t size)
```

To change the buffer memory access type:
```c++
argument.setMemType(cl_mem_flags mem_type)
```

### Kernels
This is an abstraction of functions in the OpenCL program. Once created, the function can be used in various programs.
```c++
ecl::GPUKernel myKern = "name_of_my_kernel"
```

### Programs
This is an abstraction of the program from the devices of the final execution. Once created, the program can be used on various devices.
```c++
ecl::GPUProgram myProg = "__kernel void name_of_my_kernel(){...}"
```

### GPU's
This is an abstraction of the execution device, which is a logical execution device. One physical execution device may have several logical ones.
```c++
ecl::GPU video(size_t platform_index, size_t device_index)
```

## Hello, World
 1) Copy 'EasyCL.hpp' to project folder
 2) Create `main.cpp`:

```c++

#include <iostream>
#include "EasyCL.hpp"

int main(){
    ecl::GPU video(0, 1);

    ecl::GPUProgram prog = ecl::GPUProgram::loadProgram("kernel.cl");
    ecl::GPUKernel kern = "test";

    size_t A[12] = {0};
    ecl::GPUArgument a(A, 12 * sizeof(size_t), CL_MEM_READ_WRITE);

    video.sendData({&a});
    video.compute(&prog, &kern, {&a}, {12}, {3});
    video.receiveData({&a});

    for(size_t i = 0 ; i < 12; i++) std::cout << A[i] << " ";
    std::cout << std::endl;

    ecl::Platform::free();

    return 0;
}
```
 3) Create `kernel.cl`:
```c
__kernel void test(__global size_t* a){
    size_t i = get_global_id(0);
    a[i] = get_group_id(0) + 1;
}
```

 4) Type in terminal:
```bash
$ g++ -lOpenCL -o test main.cpp
$ ./test
```

Output:
```
1 1 1 2 2 2 3 3 3 4 4 4
```
