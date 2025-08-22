# cmake构建系统

## 介绍
一个基于CMake的构建系统被组织为一组高级逻辑目标。每个目标对应一个可执行文件、库，或者是一个包含自定义命令的自定义目标。
构建系统中定义了目标之间的依赖关系，这些依赖关系会决定构建顺序以及内容变更时的重新生成规则

## 二进制目标
add_executable()和add_library()命令分别用于定义可执行文件和库。生成的二进制文件根据目标平台会自动带上合适的前缀、后缀以及扩展名。
target_link_libraries()命令用于展示二进制目标之间的依赖关系。
```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
add_executable(zipapp zipapp.cpp)
target_link_libraries(zipapp archive)
```

### 可执行文件

### 静态库

### 共享库

### 苹果框架

### 模块库

### 对象库

## 构建规范和使用要求

### 目标命令

### 目标构建规范


#### 目标编译属性

#### 目标链接属性