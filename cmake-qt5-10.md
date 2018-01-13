# Using CMake to build Qt 5.10 projects

Let's take the Simple OpenGl example from `C:\Qt\Examples\$(QTVERSION)\qt3d\simple-cpp`. I took this file and copied it into another folder.

Now, create a CMakeLists.txt file with the following:

```cmake
cmake_minimum_required(VERSION 3.2 FATAL_ERROR)
project (SimpleCPP VERSION 0.1 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

if(MSVC)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /W4")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra -Wpedantic -std=c++14")
endif()

set(CMAKE_INCLUDE_CURRENT_DIR ON)
set(CMAKE_AUTOMOC ON)
find_package(Qt5 COMPONENTS Core Widgets 3DCore 3DRender 3DInput 3DExtras REQUIRED)

# build source code
file(GLOB_RECURSE SOURCES "*.cpp")
file(GLOB_RECURSE HEADERS "*.h")
add_executable(simple_cpp orbittransformcontroller.cpp main.cpp orbittransformcontroller.h)

# Copy Qt5 DLLs
# we use windeployqt to copy our dll dependencies into the same folder
# that the executable is built into
if (MSVC)
    get_target_property(QT5_WIDGETS_LIB Qt5::Widgets LOCATION)
    get_filename_component(QT5_BIN_DIR ${QT5_WIDGETS_LIB} PATH)
    set(QT5_WINDEPLOYQT_EXECUTABLE "${QT5_BIN_DIR}/windeployqt.exe")

    add_custom_command(TARGET simple_cpp POST_BUILD
       COMMAND ${QT5_WINDEPLOYQT_EXECUTABLE} $<TARGET_FILE_DIR:simple_cpp>)
endif()

target_link_libraries(simple_cpp Qt5::Core
                      Qt5::Widgets
                      Qt5::3DCore
                      Qt5::3DRender
                      Qt5::3DInput
                      Qt5::3DExtras)
                        
```

This requires the Qt_DIR to be set. I use a Powershell script to do the following:
```powershell
# scripts/run_cmake.ps1
New-Item -ItemType Directory -Force -Path ./build/ | Out-Null
pushd build
cmake .. -G "Visual Studio 14 2015 Win64" `
    -DQt5_DIR="C:\Qt\5.10.0\msvc2015_64\lib\cmake\Qt5"
popd
```

After running Cmake, you will have your Visual Studio Solution files in a `build` directory. From there you can compile and run with Visual Studio as normal.

## My Environment
(At time of writing)
- MSVC 2015 SP3
- Qt 5.10.0
- Cmake 3.10

## Further reading:
- https://doc.qt.io/qt-5/cmake-manual.html
- https://gist.github.com/Rod-Persky/e6b93e9ee31f9516261b
- https://www.kdab.com/using-cmake-with-qt-5/
