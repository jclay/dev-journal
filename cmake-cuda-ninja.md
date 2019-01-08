# Using CUDA with CMake, Ninja and Windows 10

**Note this is outdated since CUDA 10 and above now support VS2017 with latest updates**

Here, I document the setup process I'm using for my CUDA development process in Windows 10.

The first thing to note is that the CUDA toolkit is [constantly](https://devtalk.nvidia.com/default/topic/1027209/cuda-setup-and-installation/cuda-9-0-does-not-work-with-the-latest-vs-2017-update/) [breaking](https://devtalk.nvidia.com/default/topic/1028669/cuda-setup-and-installation/how-to-get-visual-studio-2017-to-use-cuda-9-1-/) [with Visual Studio 2017](https://devtalk.nvidia.com/default/topic/1027299/cuda-9-failed-to-support-the-latest-visual-studio-2017-version-15-5/?offset=51).
For this reason, I recommend using the Visual Studio 2015 SP3 which has proven to be very stable.

## Setup

* Install [Visual Studio 2015 SP3](https://my.visualstudio.com/Downloads?q=visual%20studio%202015&wt.mc_id=o~msft~vscom~older-downloads)
  * Ensure that you select C++ tools and ensure that you select Windows and Web Development -> Universal Windows App Development Tools -> Tools (1.4.1) and Windows 10 SDK (10.0.14393)
* Install Cuda 9.2
* Install latest CMake (3.11 at time of writing)
* Install Ninja

## A Simple Example
Here, I'm going to use the `basic_vector.cu` from the Thrust examples.

```cuda
#include <thrust/host_vector.h>
#include <thrust/device_vector.h>

#include <iostream>

int main(void)
{
    // H has storage for 4 integers
    thrust::host_vector<int> H(4);

    // initialize individual elements
    H[0] = 14;
    H[1] = 20;
    H[2] = 38;
    H[3] = 46;
    
    // H.size() returns the size of vector H
    std::cout << "H has size " << H.size() << std::endl;

    // print contents of H
    for(size_t i = 0; i < H.size(); i++)
        std::cout << "H[" << i << "] = " << H[i] << std::endl;

    // resize H
    H.resize(2);
    
    std::cout << "H now has size " << H.size() << std::endl;

    // Copy host_vector H to device_vector D
    thrust::device_vector<int> D = H;
    
    // elements of D can be modified
    D[0] = 99;
    D[1] = 88;
    
    // print contents of D
    for(size_t i = 0; i < D.size(); i++)
        std::cout << "D[" << i << "] = " << D[i] << std::endl;

    // H and D are automatically deleted when the function returns
    return 0;
}
```

## CMakeLists.txt

Now, we create a CMakeLists.txt file to tell CMake how to build our project. Note that we explicitly require CMake 3.8 and above as older versions do not have native support for CUDA and require additional workarounds.

```
cmake_minimum_required(VERSION 3.8 FATAL_ERROR)
project(basic_vector LANGUAGES CXX CUDA)

add_executable(basic_vector basic_vector.cu)
target_compile_features(basic_vector PUBLIC cxx_std_11)

# Required for CUDA builds
set_target_properties(basic_vector PROPERTIES CUDA_SEPARABLE_COMPILATION ON)
```

## Running CMake

Open the VS 2015 x64 Native Tools Command Prompt. From there, you may choose to enter a Powershell session if you wish. In the directory of our project, run the following:

```
cmake . -G Ninja -Bbuild `
    -DCMAKE_C_COMPILER:PATH="C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\cl.exe" `
    -DCMAKE_CXX_COMPILER:PATH="C:\Program Files (x86)\Microsoft Visual Studio 14.0\VC\bin\amd64\cl.exe" 
```

Here, I am setting the compiler paths explicitly, otherwise CMake will try to pick up my Visual Studio 2017 installation.

If that is successful, you can now go into the `build` directory and run `cmake --build .` 

Now, you can run `.\basic_vector.exe` and you should see that the program runs successfully.
