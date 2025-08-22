# cmake构建系统

## 介绍
该构建系统基于CMake,他由若干高级逻辑目标组成。 每个目标对应一个可执行文件、库，或者是一个包含自定义命令的自定义目标。
构建系统中定义了目标之间的依赖关系，这些依赖关系会决定构建顺序以及内容变更时的重新生成规则

## 二进制目标
`add_executable()`和`add_library()`命令分别用于定义**可执行文件**和**库**。生成的二进制文件根据目标平台会自动带上合适的前缀、后缀以及扩展名。
`target_link_libraries()`命令用于表示二进制目标之间的依赖关系。
```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
add_executable(zipapp zipapp.cpp)
target_link_libraries(zipapp archive)
```
**更像是规则的集合**
上面的构建系统中：
- 第一行，定义了一个静态库`archive`，这个静态库中有由`archive.cpp`,`zip.cpp`以及`lzma.cpp`等源文件编译得到的对象.
- 第二行，定义了一个可执行目标，该目标(.o/.obj)依赖于`zipapp.cpp`。编译时不涉及到外部库。
- 第三行，告诉链接器，链接`zipapp`时需要加入`archive`这个静态库。从而链接器才会在链接阶段将`archive`和目标文件链接，最后生成可执行文件

### 可执行文件
可执行文件是有链接对象一起创建的二进制文件，其中一个链接对象应该包含一个程序入口点（main函数)
`add_executable()`命令用于定义**可执行目标**
```cmake
add_executable(mytool mytool.cpp)
```


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