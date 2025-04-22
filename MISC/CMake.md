
## 0.常用

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

