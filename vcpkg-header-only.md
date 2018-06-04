# Finding header only libraries with VCPKG and CMake

I'm using the args header only library for this example from [Taywee/args](https://github.com/Taywee/args)

Since most ports in VCPKG don't export a target for CMake to pick up the library, we need to use the following:

```
# CMakeLists.txt
...
find_path(ARGS_INCLUDE_DIR args.hxx REQUIRED)
include_directories(${ARGS_INCLUDE_DIR})
```
