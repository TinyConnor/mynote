
## 0.常用
默认指定编译为linux下的makefile，这样就可以每次编译不需要-G指定编译类型
>添加环境变量：
>变量名：CMAKE_GENERATOR(任意)
>变量值：MinGW Makefiles

```cmake
#最小版本号
cmake_minimum_required (VERSION 2.8)

#工程名
project (learn_cmake)

#引入头文件路径
include_directories(./inc_dir1 ./inc_dir2)#包含头文件的两个文件夹

#获取目录下的所有源文件
aux_source_directory (src_dir1 SRC_LIST1)
aux_source_directory (src_dir2 SRC_LIST2)
aux_source_directory (main_dir MAIN_DIR)

#生成可以执行文件
add_executable (hello ${SRC_LIST1} ${SRC_LIST2} ${MAIN_DIR})
```



## 1.只有一个源文件编译程序

**CMakeLists.txt：**

```cmake
cmake_minimum_required (VERSION 3.10)#cmake最低版本要求

project (learn_cmake)#工程名

add_executable(hello hello.cpp)#要生成的可执行文件 以及所需的源文件
```

## 2.同一目录下多个源文件

**CMakeLists.txt：**

```cmake
cmake_minimum_required (VERSION 3.10)#cmake最低版本要求

project (learn_cmake)#工程名

add_executable(hello hello.cpp myadd.cpp)#要生成的可执行文件 以及所需的源文件
```

## 3.同一目录下很多源文件

```cmake
cmake_minimum_required (VERSION 2.8)

project(learn_cmake)

aux_source_directory (. SRC_LIST)#将当前文件夹下所有源文件存储在SRC_LIST变量中

add_executable (hello ${SRC_LIST})#生成hello可执行文件 源文件在SRC_LIST变量中
```

`aux_source_directory(dir var)`:将`dir`目录下源文件存储在`var`变量中

CMake变量使用为`${变量名}`

## 4.头文件在别的文件夹

```cmake
cmake_minimum_required (VERSION 2.8)

project(learn_cmake)

aux_source_directory (. SRC_LIST)

include_directories (./inc_dir)#头文件在当前目录下的inc_dir文件夹下

add_executable (hello ${SRC_LIST})
```

`include_directories(dir)`:自动去`dir`下寻找头文件

## 5.头文件源文件分离，并包含多个文件夹

```cmake
cmake_minimum_required (VERSION 2.8)

project (learn_cmake)

#引入头文件路径
include_directories(./inc_dir1 ./inc_dir2)#包含头文件的两个文件夹

#获取目录下的所有源文件
aux_source_directory (src_dir1 SRC_LIST1)
aux_source_directory (src_dir2 SRC_LIST2)
aux_source_directory (main_dir MAIN_DIR)

#生成可以执行文件
add_executable (hello ${SRC_LIST1} ${SRC_LIST2} ${MAIN_DIR})
```

## 6.生成动态库和静态库

```cmake
cmake_minimum_required (VERSION 2.8)

project (learn_cmake)

aux_source_directory(${PROJECT_BINARY_DIR}/../src SRC_LIST)

include_directories(${PROJECT_BINARE_DIR}/../inc)

#生成动态静态库，参数1：生成库的名称 参数2：静态或动态 参数3：生成所需要的源文件
add_library(func_shared SHARED ${SRC_LIST})
add_library(func_static STATIC ${SRC_LIST})

#修改生成库的名字(如果需要改的话)
set_target_properties(func_shared PROPERTIES OUTPUT_NAME "myfunc")
set_target_properties(func_shared PROPERTIES OUTPUT_NAME "mufunc")

#设置生成路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/../lib)
```

`PROJECT_BINARY_DIR`:是cmake系统变量，意思是执行cmake命令的目录

> add_library(lib_name STATIC/SHARED src) 
>
> 函数作用：生成库。
>
> 参数lib_name：是要生成的库名称，
>
> 参数`STATIC/SHARED`：指定生成静态库或动态库，
>
> 参数src：指明库的生成所需要的源文件

`set_target_properties`:重新定义了库的输出名称，如果不使用set_target_properties也可以，那么库的名称就是add_library里定义的名称。
`LIBRARY_OUTPUT_PATH `:是cmake系统变量，项目生成的库文件都放在这个目录下。

## 7.连接库文件

```cmake
cmake_minimum_required (VERSION 2.8)

project(learn_cmake)

aux_source_directory(${PROJECT_BINARE_DIR}/../main_src MAIN_SRC)

include_directories(${PROJECT_BINARY_DIR}/../inc)

#查找库文件
#参数1：一个变量，存储查找到的库文件
#参数2：要查找的库文件
#参数3：查找的目录
find_library(FUNC_LIB mufunc ${PROJECT_BINARY_DIR}/../lib)

#设置可执行文件路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/../bin)

add_executable(hello ${MAIN_SRC})

#把库连接到可执行文件
target_link_libraries(hello ${FUNC_LIB})
```

`EXECUTABLE_OUTPUT_PATH`:是cmake系统变量，意思是生成的可执行文件的的目录。

> find_library(var lib_name lib_path1 lib_path2)
>
> 函数作用：查找库，并把库的绝对路径和名称存储到第一个参数里
>
> 参数var：用于存储查找到的库
>
> 参数lib_name：想要查找的库的名称，`默认是查找动态库`，想要指定查找动态库或静态库
>
> 可以加后缀，例如 funcname.so 或 funcname.a 
>
> 参数lib_path：想要从哪个路径下查找库，可以指定多个路径

> target_link_libraries(target lib_name)
>
> 函数作用：把库lib_name链接到target可执行文件中

## 8.添加编译选项

```cmake
#在CMake中添加编译选项
add_compile_options(-std=c++11 -Wall -02)
```

---
---
# 1.可执行文件到库
### 1.1将单个源文件编译成可执行文件
```cmake
# 设置最低版本
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# 工程名和语言
project(recipe-01 LANGUAGES C)
# 可执行文件和源文件
add_executable(hello-world hello-world.c)
```

```shell
mkdir build #创建配置文件夹
cd build
cmake .. #指定CMakeLists.txt路径
cmake --build . #在当前路径下编译
./hello-world #执行
```

构建完后build目录有如下文件
>build
>    |-----CMakeFiles/
>    |-----cmake_install.camke
>    |-----CMakeCache.txt
>    |-----Makefile


GNU/Linux上，CMake默认生成Unix Makefile来构建项目：
- *Makefile*  : make 将运行指令来构建项目。
- *CMakefile*  ：包含临时文件的目录，CMake用于检测操作系统、编译器等。此外，根据所选的生成器，它还包含特定的文件。
- *cmake_install.cmake* ：处理安装规则的CMake脚本，在项目安装时使用。
- *CMakeCache.txt* ：如文件名所示，CMake缓存。CMake在重新运行配置时使用这个文件。

### 1.2切换生成器
```shell
camke .. -G Ninja #构建时显式指定
```
或者
在Windows环境变量中设置一个值显式指定，在构建时就可以不加了
![[cmake环境变量.png]]

### 1.3构建并链接动态、静态库
```cmake
# 设置最低版本
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

# project name and language
project(recipe-03 LANGUAGES CXX)

# 为源文件生成一个静态库
add_library(message
  STATIC
    Message.hpp
    Message.cpp
  )

# 添加可执行文件及源文件
add_executable(hello-world hello-world.cpp)

# 为可执行文件链接库
target_link_libraries(hello-world message)
```

*add_library(库名  <参数> 源文件)* 中参数可为：
- **STATIC**：用于创建静态库，即编译文件的打包存档，以便在链接其他目标时使用，例如：可执行文件。
- **SHARED**：用于创建动态库，即可以动态链接，并在运行时加载的库。可以在 CMakeLists.txt 中使用 add_library(message SHARED Message.hpp Message.cpp) 从静态库切换到动态共享对象(DSO)。
- **OBJECT**：可将给定 add_library 的列表中的源码编译到目标文件，不将它们归档到静态库中，也不能将它们链接到共享对象中。如果需要一次性创建静态库和动态库，那么使用对象库尤其有用。

```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# project name and language
project(recipe-03 LANGUAGES CXX)

# 构建目标库
add_library(message-objs
  OBJECT
    Message.hpp
    Message.cpp
  )

# 开启PIC(位置无关代码),动态库所必须，默认开启，一些就版本可能不会开
set_target_properties(message-objs
  PROPERTIES
    POSITION_INDEPENDENT_CODE 1
  )
 
# 从目标库构建动态库
add_library(message-shared
  SHARED
    $<TARGET_OBJECTS:message-objs>
  )
#设置输出文件名
set_target_properties(message-shared
  PROPERTIES
    OUTPUT_NAME "message"
  )
  
# 从目标库构建静态库
add_library(message-static
  STATIC
    $<TARGET_OBJECTS:message-objs>
  )
#设置输出文件名
set_target_properties(message-static
  PROPERTIES
    OUTPUT_NAME "message"
  )
  
# 生成可执行文件
add_executable(hello-world hello-world.cpp)
# 链接静态库
target_link_libraries(hello-world message-static)
```

### 1.4条件语句控制编译
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# project name and language
project(recipe-04 LANGUAGES CXX)

# 添加切换按钮,控制是否启用库
set(USE_LIBRARY OFF)
# 打印切换按钮值
message(STATUS "Compile sources into a library? ${USE_LIBRARY}")

# 关闭动态库，构建库时可省略第二个参数
set(BUILD_SHARED_LIBS OFF)

# _sources变量 追加源文件
list(APPEND _sources Message.hpp Message.cpp)

  
#如果启用库
if(USE_LIBRARY)
  # 创建库 生成可执行文件并链接
  add_library(message ${_sources})
  add_executable(hello-world hello-world.cpp)
  target_link_libraries(hello-world message)
else()
# 不启用库  直接生成可执行文件
  add_executable(hello-world hello-world.cpp ${_sources})
endif()
```

- 如果将逻辑变量设置为以下任意一种： 1 、 ON 、 YES 、 true 、 Y 或非零数，则逻辑变量为 true 。
- 如果将逻辑变量设置为以下任意一种： 0 、 OFF 、 NO 、 false 、 N 、 IGNORE、NOTFOUND 、空字符串，或者以 -NOTFOUND 为后缀，则逻辑变量为 false 。

### 1.5向用户显示选项
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# project name and language
project(recipe-05 LANGUAGES CXX)

  

# USE_LIBRARY变量，默认为关
option(USE_LIBRARY "Compile sources into a library" OFF)
# 显示$(USE_LIBRARY)变量值
message(STATUS "Compile sources into a library? ${USE_LIBRARY}")

  
# 添加后面使用cmake_dependent_option选项依赖
include(CMakeDependentOption)

  

# 如果$(USE_LIBRARY)为开，设置MAKE_STATIC_LIBRARY变量，默认为关
cmake_dependent_option(
  MAKE_STATIC_LIBRARY "Compile sources into a static library" OFF
  "USE_LIBRARY" ON
  )

# 如果$(USE_LIBRARY)为开，设置MAKE_SHARED_LIBRARY变量，默认为关
cmake_dependent_option(
  MAKE_SHARED_LIBRARY "Compile sources into a shared library" ON
  "USE_LIBRARY" ON
  )

# 自动导出动态库链所有符号，Windows专属
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# 追加源文件
list(APPEND _sources Message.hpp Message.cpp)

#如果使用库
if(USE_LIBRARY)
# 打印信息使用静态库or动态库
  message(STATUS "Compile sources into a STATIC library? ${MAKE_STATIC_LIBRARY}")
  message(STATUS "Compile sources into a SHARED library? ${MAKE_SHARED_LIBRARY}")
# 使用动态库
  if(MAKE_SHARED_LIBRARY)
    add_library(message SHARED ${_sources})
    add_executable(hello-world hello-world.cpp)
    target_link_libraries(hello-world message)
  endif()
# 使用静态库
  if(MAKE_STATIC_LIBRARY)
    add_library(message STATIC ${_sources})
    add_executable(hello-world hello-world.cpp)
    target_link_libraries(hello-world message)
  endif()
else()
# 不使用库
  add_executable(hello-world hello-world.cpp ${_sources})
endif()
```

构建时可使用 *-D* 选项控制变量值
```shell
cmake -D USE_LIBRARY=OFF -D MAKE_SHARED_LIBRARY=ON ..
```

### 1.6指定编译器
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# project name and language
project(recipe-06 LANGUAGES C CXX)

# 打印编译器属性
message(STATUS "Is the C++ compiler loaded? ${CMAKE_CXX_COMPILER_LOADED}")

if(CMAKE_CXX_COMPILER_LOADED)
  message(STATUS "The C++ compiler ID is: ${CMAKE_CXX_COMPILER_ID}")
  message(STATUS "Is the C++ from GNU? ${CMAKE_COMPILER_IS_GNUCXX}")
  message(STATUS "The C++ compiler version is: ${CMAKE_CXX_COMPILER_VERSION}")
endif()

message(STATUS "Is the C compiler loaded? ${CMAKE_C_COMPILER_LOADED}")
if(CMAKE_C_COMPILER_LOADED)
  message(STATUS "The C compiler ID is: ${CMAKE_C_COMPILER_ID}")
  message(STATUS "Is the C from GNU? ${CMAKE_COMPILER_IS_GNUCC}")
  message(STATUS "The C compiler version is: ${CMAKE_C_COMPILER_VERSION}")
endif()
```

可以使用 *cmake --system-information information.txt* 命令导出所有cmake全局变量

### 1.7切换构建类型
1. Debug：用于在没有优化的情况下，使用带有调试符号构建库或可执行文件。
2. Release：用于构建的优化的库或可执行文件，不包含调试符号。
3. RelWithDebInfo：用于构建较少的优化库或可执行文件，包含调试符号。
4. MinSizeRel：用于不增加目标代码大小的优化方式，来构建库或可执行文件。

```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)

# project name and language
project(recipe-07 LANGUAGES C CXX)

  
# 默认设置Release构建类型
if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type" FORCE)
endif()
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

  
# 打印编译器信息
message(STATUS "C flags, Debug configuration: ${CMAKE_C_FLAGS_DEBUG}")
message(STATUS "C flags, Release configuration: ${CMAKE_C_FLAGS_RELEASE}")
message(STATUS "C flags, Release configuration with Debug info: ${CMAKE_C_FLAGS_RELWITHDEBINFO}")
message(STATUS "C flags, minimal Release configuration: ${CMAKE_C_FLAGS_MINSIZEREL}")

message(STATUS "C++ flags, Debug configuration: ${CMAKE_CXX_FLAGS_DEBUG}")
message(STATUS "C++ flags, Release configuration: ${CMAKE_CXX_FLAGS_RELEASE}")
message(STATUS "C++ flags, Release configuration with Debug info: ${CMAKE_CXX_FLAGS_RELWITHDEBINFO}")
message(STATUS "C++ flags, minimal Release configuration: ${CMAKE_CXX_FLAGS_MINSIZEREL}")
```

### 1.8设置编译器选项
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)
# project name and language
project(recipe-08 LANGUAGES CXX)

message("C++ compiler flags: ${CMAKE_CXX_FLAGS}")

#  追加$(flags)编译选项
list(APPEND flags "-fPIC" "-Wall")
# win没有这些选项
if(NOT WIN32)
  list(APPEND flags "-Wextra" "-Wpedantic")
endif()

  
# 构建静态库
add_library(geometry
  STATIC
    geometry_circle.cpp
    geometry_circle.hpp
    geometry_polygon.cpp
    geometry_polygon.hpp
    geometry_rhombus.cpp
    geometry_rhombus.hpp
    geometry_square.cpp
    geometry_square.hpp
  )
# 为库添加编译选项
target_compile_options(geometry
  PRIVATE
    ${flags}
  )

# 添加可执行文件
add_executable(compute-areas compute-areas.cpp)
# 为可执行文件添加编译选项
target_compile_options(compute-areas
  PRIVATE
    "-fPIC"
  )
# 链接库
target_link_libraries(compute-areas geometry)
```

*target_compile_options* 添加编译选项第二个参数：
- **PRIVATE**，编译选项会应用于给定的目标，不会传递给与目标相关的目标。我们的示例中， 即使 compute-areas 将链接到 geometry 库， compute areas 也不会继承 geometry 目标上设置的编译器选项。
- **INTERFACE**，给定的编译选项将只应用于指定目标，并传递给与目标相关的目标。
- **PUBLIC**，编译选项将应用于指定目标和使用它的目标。

### 1.9为语言设定标准
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR) 
# project name and language
project(recipe-09 LANGUAGES CXX)
# **自动导出动态链接库（DLL）中的所有符号**
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

  
# 构建库
add_library(animals
  SHARED
    Animal.cpp
    Animal.hpp
    Cat.cpp
    Cat.hpp
    Dog.cpp
    Dog.hpp
    Factory.hpp
  )
# 为库设定语言标准
set_target_properties(animals
  PROPERTIES
    CXX_STANDARD 14
    CXX_EXTENSIONS OFF
    CXX_STANDARD_REQUIRED ON
    POSITION_INDEPENDENT_CODE 1
  )

# 构建可执行文件
add_executable(animal-farm animal-farm.cpp)
# 为可执行文件设定语言标准
set_target_properties(animal-farm
  PROPERTIES
    CXX_STANDARD 14
    CXX_EXTENSIONS OFF
    CXX_STANDARD_REQUIRED ON
  )
# 连接库
target_link_libraries(animal-farm animals)
```

- *CXX_STANDARD* 会设置我们想要的标准。
- *CXX_EXTENSIONS* 告诉CMake，只启用 ISO C++ 标准的编译器标志，而不使用特定编译器的扩展。
- *CXX_STANDARD_REQUIRED* 指定所选标准的版本。如果这个版本不可用，CMake将停止配置并出现错误。当这个属性被设置为 OFF 时，CMake将寻找下一个标准的最新版本，直到一个合适的标志。这意味着，首先查找 C++14 ，然后是 C++11 ，然后是 C++98 。

### 1.10使用控制流
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)  
# project name and language
project(recipe-10 LANGUAGES CXX)

# 构建静态库
add_library(geometry
  STATIC
    geometry_circle.cpp
    geometry_circle.hpp
    geometry_polygon.cpp
    geometry_polygon.hpp
    geometry_rhombus.cpp
    geometry_rhombus.hpp
    geometry_square.cpp
    geometry_square.hpp
  )
# 设置库编译选项
target_compile_options(geometry
  PRIVATE
    -O3
  )

# 构建$(sources_with_lower_optimization),后面循环控制里面参数
list(
  APPEND sources_with_lower_optimization
    geometry_circle.cpp
    geometry_rhombus.cpp
  )

# 使用 foreach 语法设置编译属性
message(STATUS "Setting source properties using IN LISTS syntax:")
foreach(_source IN LISTS sources_with_lower_optimization)
  set_source_files_properties(${_source} PROPERTIES COMPILE_FLAGS -O2)
  message(STATUS "Appending -O2 flag for ${_source}")
endforeach()

# 使用 foreach 语法获取编译属性
message(STATUS "Querying sources properties using plain syntax:")
foreach(_source ${sources_with_lower_optimization})
  get_source_file_property(_flags ${_source} COMPILE_FLAGS)
  message(STATUS "Source ${_source} has the following extra COMPILE_FLAGS: ${_flags}")
endforeach()

# 构建可执行文件
add_executable(compute-areas compute-areas.cpp)
# 链接库
target_link_libraries(compute-areas geometry)
```

# 2.检测环境
### 2.1检测操作系统
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)  
# project name, in this case no language required
project(recipe-01 LANGUAGES NONE)

  
# $(CMAKE_SYSTEM_NAME)存放系统名
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  message(STATUS "Configuring on/for Linux")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  message(STATUS "Configuring on/for macOS")
elseif(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  message(STATUS "Configuring on/for Windows")
elseif(CMAKE_SYSTEM_NAME STREQUAL "AIX")
  message(STATUS "Configuring on/for IBM AIX")
else()
  message(STATUS "Configuring on/for ${CMAKE_SYSTEM_NAME}")
endif()
```

### 2.2处理与平台相关源代码
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR) 
# project name and language
project(recipe-02 LANGUAGES CXX)
  
# 构建可执行文件
add_executable(hello-world hello-world.cpp)

# 定义IS_<系统名>变量
if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
  target_compile_definitions(hello-world PUBLIC "IS_LINUX")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Darwin")
  target_compile_definitions(hello-world PUBLIC "IS_MACOS")
endif()
if(CMAKE_SYSTEM_NAME STREQUAL "Windows")
  target_compile_definitions(hello-world PUBLIC "IS_WINDOWS")
endif()
```

```c
/* hello-world.cpp */
#include <cstdlib>
#include <iostream>
#include <string>

std::string say_hello() {
#ifdef IS_WINDOWS
  return std::string("Hello from Windows!");
#elif IS_LINUX
  return std::string("Hello from Linux!");
#elif IS_MACOS
  return std::string("Hello from macOS!");
#else
  return std::string("Hello from an unknown system!");
#endif
}
int main() {
  std::cout << say_hello() << std::endl;
  return EXIT_SUCCESS;
}
```

### 2.3处理与编译器相关的源代码
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR)  
# project name and language
project(recipe-03 LANGUAGES CXX)

  

# 构建可执行文件
add_executable(hello-world hello-world.cpp)
# 为执行文件定义COMPILER_NAME宏，为编译器厂商名
target_compile_definitions(hello-world PUBLIC "COMPILER_NAME=\"${CMAKE_CXX_COMPILER_ID}\"")

# 根据编译器厂商名定义各自相关宏
if(CMAKE_CXX_COMPILER_ID MATCHES Intel)
  target_compile_definitions(hello-world PUBLIC "IS_INTEL_CXX_COMPILER")
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES GNU)
  target_compile_definitions(hello-world PUBLIC "IS_GNU_CXX_COMPILER")
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES PGI)
  target_compile_definitions(hello-world PUBLIC "IS_PGI_CXX_COMPILER")
endif()
if(CMAKE_CXX_COMPILER_ID MATCHES XL)
  target_compile_definitions(hello-world PUBLIC "IS_XL_CXX_COMPILER")
endif()
# etc ...
```

### 2.4检测处理器体系结构
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.5 FATAL_ERROR) 
# project name and language
project(recipe-04 LANGUAGES CXX)

# define executable and its source file
add_executable(arch-dependent arch-dependent.cpp)

# 检查空指针类型的大小 确定系统位数
if(CMAKE_SIZEOF_VOID_P EQUAL 8)
  target_compile_definitions(arch-dependent PUBLIC "IS_64_BIT_ARCH")
  message(STATUS "Target is 64 bits")
else()
  target_compile_definitions(arch-dependent PUBLIC "IS_32_BIT_ARCH")
  message(STATUS "Target is 32 bits")
endif()

# $(CMAKE_HOST_SYSTEM_PROCESSOR)主机架构
if(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "i386")
  message(STATUS "i386 architecture detected")
elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "i686")
  message(STATUS "i686 architecture detected")
elseif(CMAKE_HOST_SYSTEM_PROCESSOR MATCHES "x86_64")
  message(STATUS "x86_64 architecture detected")
else()
  message(STATUS "host processor architecture is unknown")
endif()


target_compile_definitions(arch-dependent
  PUBLIC "ARCHITECTURE=${CMAKE_HOST_SYSTEM_PROCESSOR}"
  )
```

### 2.5检测处理器指令集
```cmake
# set minimum cmake version
cmake_minimum_required(VERSION 3.10 FATAL_ERROR)  
# project name and language
project(recipe-05 LANGUAGES CXX)

# 定义可执行文件
add_executable(processor-info "")
# 为可执行文件添加源文件
target_sources(processor-info
  PRIVATE
    processor-info.cpp
  )  
# 为可执行文件添加头文件
target_include_directories(processor-info
  PRIVATE
    ${PROJECT_BINARY_DIR}
  )

# 继续查询主机系统的信息，将结果存储在以`_`为前缀的变量中
foreach(key
  IN ITEMS
    NUMBER_OF_LOGICAL_CORES
    NUMBER_OF_PHYSICAL_CORES
    TOTAL_VIRTUAL_MEMORY
    AVAILABLE_VIRTUAL_MEMORY
    TOTAL_PHYSICAL_MEMORY
    AVAILABLE_PHYSICAL_MEMORY
    IS_64BIT
    HAS_FPU
    HAS_MMX
    HAS_MMX_PLUS
    HAS_SSE
    HAS_SSE2
    HAS_SSE_FP
    HAS_SSE_MMX
    HAS_AMD_3DNOW
    HAS_AMD_3DNOW_PLUS
    HAS_IA64
    OS_NAME
    OS_RELEASE
    OS_VERSION
    OS_PLATFORM
  )
  cmake_host_system_information(RESULT _${key} QUERY ${key})
endforeach()

# 生成配置头文件
configure_file(config.h.in config.h @ONLY)
```

*@变量名@* 为占位符，cmake根据变量生成头文件是将相应变量替换占位符

```c
/* config.h.in */
#pragma once
#define NUMBER_OF_LOGICAL_CORES   @_NUMBER_OF_LOGICAL_CORES@
#define NUMBER_OF_PHYSICAL_CORES  @_NUMBER_OF_PHYSICAL_CORES@
#define TOTAL_VIRTUAL_MEMORY      @_TOTAL_VIRTUAL_MEMORY@
#define AVAILABLE_VIRTUAL_MEMORY  @_AVAILABLE_VIRTUAL_MEMORY@
#define TOTAL_PHYSICAL_MEMORY     @_TOTAL_PHYSICAL_MEMORY@
#define AVAILABLE_PHYSICAL_MEMORY @_AVAILABLE_PHYSICAL_MEMORY@
#define IS_64BIT                  @_IS_64BIT@
#define HAS_FPU                   @_HAS_FPU@
#define HAS_MMX                   @_HAS_MMX@
#define HAS_MMX_PLUS              @_HAS_MMX_PLUS@
#define HAS_SSE                   @_HAS_SSE@
#define HAS_SSE2                  @_HAS_SSE2@
#define HAS_SSE_FP                @_HAS_SSE_FP@
#define HAS_SSE_MMX               @_HAS_SSE_MMX@
#define HAS_AMD_3DNOW             @_HAS_AMD_3DNOW@
#define HAS_AMD_3DNOW_PLUS        @_HAS_AMD_3DNOW_PLUS@
#define HAS_IA64                  @_HAS_IA64@
#define OS_NAME                  "@_OS_NAME@"
#define OS_RELEASE               "@_OS_RELEASE@"
#define OS_VERSION               "@_OS_VERSION@"
#define OS_PLATFORM              "@_OS_PLATFORM@"
```

### 2.6为Eigen库使能向量化
```cmake
# 调用包
find_package(Eigen3 3.3 REQUIRED CONFIG)
```

