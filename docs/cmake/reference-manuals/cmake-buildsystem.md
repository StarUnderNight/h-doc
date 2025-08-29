# cmake构建系统

## 术语
**consuming target(消费目标)**


**target usage requirement(目标使用需求**
当目标A依赖于目标B时，目标B的**使用需求**就会自动传递给目标A,确保目标A在编译和链接的时候能够找到所需资源和配置。
这里的**使用需求**可以是一系列配置信息，包括头文件搜索路径、编译选项、链接选项等。


## 1. 介绍
该构建系统基于CMake,他由若干高级逻辑目标组成。 每个目标对应一个可执行文件、库，或者是一个包含自定义命令的自定义目标。
构建系统中定义了目标之间的依赖关系，这些依赖关系会决定构建顺序以及内容变更时的重新生成规则

## 2. 二进制目标
`add_executable()`和`add_library()`命令分别用于定义**可执行文件**和**库**。生成的二进制文件根据目标平台会自动带上合适的前缀、
后缀以及扩展名。`target_link_libraries()`命令用于表示二进制目标之间的依赖关系。
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

### 2.1 可执行文件
可执行文件是由链接对象一起创建的二进制文件，其中一个链接对象应该包含一个程序入口点（main函数)
`add_executable()`命令用于定义**可执行目标**
```cmake
add_executable(mytool mytool.cpp)
``` 
CMake生成构建规则并用于将源文件编译为目标文件，同时将目标文件链接为可执行文件。可执行文件的链接依赖可以通过`target_link_libraries()`命令来
指定。链接器从可执行文件自己源文件编译而成的目标文件开始，然后通过查找链接库来解析剩余符号依赖。
诸如`add_custom_command()`这类用于生成**构建时待执行规则**的命令，可直接将一个EXECUTABLE目标用作COMMAND对应的可执行文件。
构建系统的规则会确保：在尝试运行该命令之前，先完成该可执行文件的构建。

### 2.2 静态库
静态库其实是目标文件的归档，他们由归档器来创建，而不是链接器。可执行文件、共享库以及模块库在链接时能将静态库作为依赖项进行链接。链接器根据符号
解析需求从静态库中选择目标文件的子集，然后将其链接为待生成的二进制文件。每个链接了静态库的二进制文件都会有静态库中所需文件的拷贝，
静态库则在运行时不再需要。

使用`add_library()`命令并指定`STATIC`库类型来定义静态库
```cmake 
add_library(archive STATIC archive.cpp zip.cpp lzma.cpp)
```
或者`BUILD_SHARED_LIBS`变量为false，可以不用携带类型
```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
```
CMake 会生成构建规则，将源文件编译为目标文件（object files），并将这些目标文件归档（archive）为静态库。

静态库的链接依赖项可通过 target_link_libraries() 命令指定。由于静态库本质是归档文件而非已链接的二进制文件，其链接依赖项中的目标文件不会被
包含到该静态库本身（直接链接依赖项中指定的“目标文件库”除外）。相反，CMake 会记录静态库的链接依赖项，以便在链接“使用该静态库的二进制文件”
（如可执行文件、动态库）时，实现依赖的传递性使用。

#### 理解
静态库是一个小仓库，里面存储了若干目标文件(`.o`)，当我们要创建一个可执行文件、共享库时，如果需要静态库，那么链接器就会在链接阶段，根据符号解析
，从静态库中找到需要的目标文件并拷贝一份，最终的可执行文件、共享库中会包含这个拷贝文件。

当需要使用某个静态库的时候，使用`target_link_libraries()`命令来指定需要用到的静态库

### 共享库
共享库也可以叫做动态库，静态库是**目标文件**`.o`的归档，共享库则是链接的对象文件创建的二进制文件。可执行文件、其他动态库以及模块库能将动态库
作为依赖项进行链接。链接器会记录共享库在待生成文件中的引用。程序运行时，动态加载器会从磁盘搜索动态库并加载符号。

使用`add_library()`命令并指定SHARED参数来定义共享库
```cmake
add_library(archive SHARED archive.cpp zip.cpp lzma.cpp)
```
如果变量`BUILD_SHARED_LIBS`为true,则不需要携带类型
```cmake
add_library(archive archive.cpp zip.cpp lzma.cpp)
```
CMake生成构建规则并用于将源文件编译为目标文件，并将这些目标文件链接为一个共享库。

同静态库一样，`target_link_libraries()`命令用于指定需要使用的共享库依赖。链接器会从共享库编译的目标文件中开始，然后通过查询链接的库来解析
剩余的符号依赖

#### 理解
共享库是一种二进制文件，它由若干目标文件链接在一起。在链接阶段，链接器将共享库的**名称和符号信息**记录到可执行文件中，在可执行文件运行的过程中，
动态链接器就会根据**名称和符号信息**将所需要的库加载到内存中。这不同于静态库，需要将文件拷贝，最后都集中到可执行文件中。

### 2.3 苹果框架
和苹果开发相关，可自行官网查看

### 2.4 模块库
模块库也是链接的对象文件创建的二进制文件，但和共享库不同之处是，模块库不能作为依赖项供其他二进制文件链接：也就是说在`target_link_libraries()`
这个命令中，不要将模块库放到右侧位置（就是不能供其他库使用）。模块库是应用程序的插件，且能够在程序后台运行时动态加载。

`add_library`命令定义模块库，库类型为`MODULE`
```cmake
add_library(archivePlugin MODULE 7z.cpp)
```
CMake生成构建规则并用于将源文件编译为目标文件，并将这些目标文件链接为一个模块库。

模块库的链接依赖项使用`target_link_libraries()`命令来指定。链接器会从模块库编译的目标文件中开始，然后通过查询链接的库来解析
剩余的符号依赖。

### 2.5 对象库
对象库是源代码编译之后的目标文件的集合，对象文件可以在链接可执行文件、共享库、模块库以及归档静态库时被使用。
```cmake
add_library(archiveObjs OBJECT archive.cpp zip.cpp lzma.cpp)
```
其他目标可以通过**生成器表达式语法**`$<TARGET_OBJECTS:name>将对象文件指定为源输入
```cmake
add_library(archiveExtras STATIC $<TARGET_OBJECTS:archiveObjs> extras.cpp)

add_executable(test_exe $<TARGET_OBJECTS:archiveObjs> test.cpp)
```
消费目标在在链接时，可以使用来自它们自身的源文件，也可以使用对应的对象库。

对象库能够作为其他目标的链接依赖。
```cmake
add_library(archiveExtras STATIC extras.cpp)
target_link_libraries(archiveExtras PUBLIC archiveObjs)

add_executable(test_exe test.cpp)
target_link_libraries(test_exe archiveObjs)
```
消费目标在链接（生成可执行文件或共享库时）或归档（生成静态库时）过程中，所使用的目标文件来自两个方面：
* 编译其**自身源文件**所生成的目标文件
* 通过`target_link_libraries()`命令指定的**直接依赖项**的**对象库**所包含的目标文件

使用`add_custom_command(TARGET)`命令时，对象库不能作为`TARGET`。但对象的列表能够通过`$<TARGET_OBJECTS:objlib>`在
`add_custom_command(OUTPUT`或`file(GENERATE)`中使用`

## 3. 构建规范和使用要求
目标的构建是根据其自身的构建规范，结合从其链接依赖项传播而来的使用要求进行的。两者都可以通过特定于目标的命令来指定。

例子：
```cmake
add_library(archive SHARED archive.cpp zip.cpp)

if (LZMA_FOUND)
  # Add a source implementing support for lzma.
  target_sources(archive PRIVATE lzma.cpp)

  # Compile the 'archive' library sources with '-DBUILDING_WITH_LZMA'.
  target_compile_definitions(archive PRIVATE BUILDING_WITH_LZMA)
endif()

target_compile_definitions(archive INTERFACE USING_ARCHIVE_LIB)

add_executable(consumer consumer.cpp)

# Link 'consumer' to 'archive'.  This also consumes its usage requirements,
# so 'consumer.cpp' is compiled with '-DUSING_ARCHIVE_LIB'.
target_link_libraries(consumer archive)
```

### 3.1 目标命令
特定于目标的命令会填充二进制目标（Binary Targets）的构建规范，以及二进制目标、接口库（Interface Libraries）和导入目标（Imported Targets）
的使用要求。

调用这些命令时必须指定作用域关键字（scope keywords），每个关键字都会影响其后续参数的可见性。

`PUBLIC`

**理解**


### 3.2 目标构建规范



#### 3.2.1 目标编译属性

#### 3.2.2 目标链接属性

### 3.3 目标使用需求
#### 3.3.1 传递编译属性
#### 3.3.2 传递链接属性

### 3.4 自定义传输属性
### 3.5 兼容接口属性
### 3.6 属性来源调试

### 3.7生成式表达式的构建规范
#### 3.7.1 include目录和使用规范
### 3.8 链接库与生成式表达式
### 3.9 输出产物
#### 3.9.1 运行时输出产物
#### 3.9.2 链接库输出产物
#### 3.9.3 归档输出产物
## 4. 构建配置
### 4.1 大小写敏感
### 4.2 默认和自定义配置
## 5. 伪目标
### 5.1 导入的目标
### 5.2 别名目标
### 5.3 接口库
