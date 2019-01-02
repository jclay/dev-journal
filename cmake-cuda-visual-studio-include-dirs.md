# Setting proper include directories with CMake, CUDA, and Visual Studio 2017

In an attempt to ease porting of a CPU based library, I have chosen to use the GLM library as a dependency. We can use 
the CMake `target_include_directories(my_target glm_include_path/)` which allows nvcc to find the files during compilation 
and linking. CMake passes the proper includes to the compiler, and we can see that reflected in the CUDA C/C++ properties
dialog within Visual Studio.

However, Visual Studio's IntelliSense will fail to find the included directories and will display errors reflecting this. 

I was not able to find a way to configure CMake to specify the include directories for both CUDA and the host compiler. 

Support was added in [CMake 3.12](https://gitlab.kitware.com/cmake/cmake/issues/17908) to override the default Visual Studio include directories using
the `CMAKE_VS_SDK_INCLUDE_DIRECTORIES` variable. We can use this to override the default Visual Studio include directories. This will allow for us
to set the **Configuration Properties > VC++ Directories > Include Directories** parameter, and will resolve the IntelliSense errors.

Here's a snippet of my CMake script with my modifications to the Include Directories:

```
option(USE_CUDA "Use CUDA" ON)
if (USE_CUDA)
  target_compile_definitions(myapp-core PUBLIC CUDA=1)
  target_compile_definitions(myapp-core PUBLIC GLM_FORCE_CUDA=1)

  enable_language(CUDA)

  # Only compile for newer Pascal cards (to save time)
  string(APPEND CMAKE_CUDA_FLAGS " -gencode=arch=compute_61,code=sm_61")

  # Hack to get intellisense working for CUDA includes
  if (MSVC)
    set(CMAKE_VS_SDK_INCLUDE_DIRECTORIES "$(VC_IncludePath);glm_include_path/")
  endif()

  set(CUDA_SOURCES cuda/GPUImplementation.cu cuda/GPUImplementation.h cuda/IntegrationUtilities.hpp)
  add_library(myapp_cuda STATIC ${CUDA_SOURCES})
  
  set_target_properties(myapp_cuda PROPERTIES
                                     CMAKE_CUDA_STANDARD 17
                                     POSITION_INDEPENDENT_CODE ON
                                     CUDA_SEPARABLE_COMPILATION ON
                                     CUDA_RESOLVE_DEVICE_SYMBOLS ON)
  

  target_include_directories(myapp_cuda PUBLIC 
        ${GLM_INCLUDE_DIRS}
        ${CMAKE_CURRENT_SOURCE_DIR}
        ${CMAKE_CUDA_TOOLKIT_INCLUDE_DIRECTORIES}
        ${CUDA_INCLUDE_DIRS}
        ${CMAKE_CURRENT_SOURCE_DIR}/cuda/)
        
  target_link_libraries(myapp_cuda PUBLIC glm cuda)

  target_link_libraries(myapp-core PUBLIC myapp_cuda)
endif()
```
