# CMake文档
本文档从官网翻译，用于自己学习和使用
官网地址: https://cmake.org/cmake/help/latest/

## C++ Build
C++源文件到最终的可执行文件分为4个阶段：预处理(Preprocessing)、编译(Compilation)、汇编(Assembly)以及链接(Linking)。

### 1. 预处理(Preprocessing)
**输入**
* C++源文件(`.cpp`,`.cc`,`.cxx`)
* C++头文件(`.h`,`.hpp`,`.hxx`)

**输出**
* 预处理后的文件(`.i`，`.ii`)

**执行工具** 
* 预处理器(一般是`cpp`)，通常由编译器驱动(`g++`或`clang++`)调用`cpp`这个命令。`cpp`是一个工具程序，名字就叫做`cpp`，
他的功能单一，只做预处理。

**命令**
```bash 
# 预处理，生成单个文件
g++ -E -o main.i main.cpp
```
**过程解析**

预处理器会对源代码进行**文本处理**，如下：
1. 展开头文件：将`#include`指令替换为相应文件的内容，递归过程，引入的头文件可能还包含其他头文件
2. 宏展开：将所有宏定义展开
3. 条件编译：根据条件(`#if`，`#ifdef`)判断，保留或删除部分代码
4. 删除注释：去掉注释
5. 处理预处理指令：处理`#pragma`、`#line`等特殊指令

### 2. 编译(Compilation)
**输入**
* 预处理后的文件`.i`

**输出**
* 汇编代码文件`.s`或`.asm`

**执行工具**
* 编译器，`g++`(GCC)、`clang++`(Clang)、`c1`(MSVC)等

**命令**
```bash
# 可以源文件作为命令输入，命令执行过程中会先生成.i文件
g++ -S main.cpp -o main.s
# .i文件作为命令输入
g++ -S main.i -o main.s
```
**过程分析**

编译器将预处理后的文件翻译成汇编语言，这个过程包括：
1. 词法分析(Lexical Analysis)：将字符序列分解成一系列的词法单元(Token)，如关键字、标识符、运算符等；
2. 语法分析(Syntax Analysis)：根据语法规则，将Token流组合成**抽象语法树(Abstract Syntax Tree)**，检查语法错误，比如缺少分号等；
3. 语义分析(Semantic Analysis)：在AST基础上进一步分析，检查语义错误，如类型不匹配、未声明的变量等；
4. 中间代码生成和优化：生成与机器无关的中间表示，并对齐进行优化（删除无用代码、常量折叠等）；
5. 代码生成：将优化后的中间代码转换为目标平台的汇编代码；

### 3. 汇编(Assembly)
**输入**
* 汇编文件`.s`

**输出**
* 目标文件(Linux上为`.o`， window上`.obj`)

**执行工具**
* 汇编器(Assembler)，如`as`(GNU Assembler)

**命令**
```bash
g++ -c main.cpp -o main.o

# 使用objdump或者nm工具可以查看目标文件内容
objdump -d main.o  # 反汇编
nm main.o # 查看符号表
```

**过程分析**

将汇编代码`.s`文件一对一翻译为**机器指令**，生成目标文件`.o`。目标文件几乎包含了所有你写的代码和数据的机器指令，但它还不是一个完整的可执行程序，
主要在于：
* 重定位信息：目标文件的代码和数据的地址都是0开始，需要链接器在最终合并的时候替换为正确的地址
* 符号表：该表记录了目标文件中定义的函数和变量，以及需要从其他地方引用的函数和变量

### 4. 链接(Linking)
**输入**
* 一个或多个目标文件`.o`
* 静态库(linux上为`.a`，window为`.lib`)

**输出**
* 可执行文件(linux无后缀，window后缀为.exe)
* 动态库/共享库(linux为`.so`，window为`.dll`)

**命令**
```bash
# 将main.o和helper.o链接为可执行程序
g++ main.o helper.o -o main
```

**过程分析**

链接器将多个目标文件和库文件"拼接"为一个完整的可执行程序，主要过程如下：
* 符号解析(Symbol Resolution)
  * 链接器查看所有目标文件，找到每个符号(函数名、变量名)的定义
  * 确保所有被引用的符号有且仅有一个定义，如果找不到定义，会报`undefined reference`错误，如果找到多个定义，会报`multiple definition`错误
* 重定位(Relocation)
  * 链接器合并所有目标文件的相同**段**，比如代码段`.text`、数据段`.data`
  * 为每个段分配最终的**内存地址**
  * 最后，修改之前未确定的引用，将其指向正确的内存地址

**库的处理**
* 静态链接：链接器将静态库`.a`中**被用到的目标文件**直接复制到最终的可执行文件中
* 动态链接：链接器只会在可执行文件中记录动态库的名称和符号信息。在程序运行时，**动态链接器**将所需要的库加载到内存并绑定地址。







## g++运行过程中调用的工具
使用如下命令
```bash 
g++ -v -c main.cpp -o main.o
```
输出内容如下：
```bash 
Using built-in specs.
COLLECT_GCC=g++
OFFLOAD_TARGET_NAMES=nvptx-none:amdgcn-amdhsa
OFFLOAD_TARGET_DEFAULT=1
Target: x86_64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 11.4.0-1ubuntu1~22.04.2' --with-bugurl=file:///usr/share/doc/gcc-11/README.Bugs --enable-languages=c,ada,c++,go,brig,d,fortran,objc,obj-c++,m2 --prefix=/usr --with-gcc-major-version-only --program-suffix=-11 --program-prefix=x86_64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/lib --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-gnu-unique-object --disable-vtable-verify --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --disable-werror --enable-cet --with-arch-32=i686 --with-abi=m64 --with-multilib-list=m32,m64,mx32 --enable-multilib --with-tune=generic --enable-offload-targets=nvptx-none=/build/gcc-11-2Y5pKs/gcc-11-11.4.0/debian/tmp-nvptx/usr,amdgcn-amdhsa=/build/gcc-11-2Y5pKs/gcc-11-11.4.0/debian/tmp-gcn/usr --without-cuda-driver --enable-checking=release --build=x86_64-linux-gnu --host=x86_64-linux-gnu --target=x86_64-linux-gnu --with-build-config=bootstrap-lto-lean --enable-link-serialization=2
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 11.4.0 (Ubuntu 11.4.0-1ubuntu1~22.04.2) 
COLLECT_GCC_OPTIONS='-v' '-c' '-o' 'main.o' '-shared-libgcc' '-mtune=generic' '-march=x86-64'
 /usr/lib/gcc/x86_64-linux-gnu/11/cc1plus -quiet -v -imultiarch x86_64-linux-gnu -D_GNU_SOURCE main.cpp -quiet -dumpbase main.cpp -dumpbase-ext .cpp -mtune=generic -march=x86-64 -version -fasynchronous-unwind-tables -fstack-protector-strong -Wformat -Wformat-security -fstack-clash-protection -fcf-protection -o /tmp/ccMdLJ5J.s
GNU C++17 (Ubuntu 11.4.0-1ubuntu1~22.04.2) version 11.4.0 (x86_64-linux-gnu)
        compiled by GNU C version 11.4.0, GMP version 6.2.1, MPFR version 4.1.0, MPC version 1.2.1, isl version isl-0.24-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
ignoring duplicate directory "/usr/include/x86_64-linux-gnu/c++/11"
ignoring nonexistent directory "/usr/local/include/x86_64-linux-gnu"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/11/include-fixed"
ignoring nonexistent directory "/usr/lib/gcc/x86_64-linux-gnu/11/../../../../x86_64-linux-gnu/include"
#include "..." search starts here:
#include <...> search starts here:
 /usr/include/c++/11
 /usr/include/x86_64-linux-gnu/c++/11
 /usr/include/c++/11/backward
 /usr/lib/gcc/x86_64-linux-gnu/11/include
 /usr/local/include
 /usr/include/x86_64-linux-gnu
 /usr/include
End of search list.
GNU C++17 (Ubuntu 11.4.0-1ubuntu1~22.04.2) version 11.4.0 (x86_64-linux-gnu)
        compiled by GNU C version 11.4.0, GMP version 6.2.1, MPFR version 4.1.0, MPC version 1.2.1, isl version isl-0.24-GMP

GGC heuristics: --param ggc-min-expand=100 --param ggc-min-heapsize=131072
Compiler executable checksum: 6c87588fc345655b93b8c25f48f88886
COLLECT_GCC_OPTIONS='-v' '-c' '-o' 'main.o' '-shared-libgcc' '-mtune=generic' '-march=x86-64'
 as -v --64 -o main.o /tmp/ccMdLJ5J.s
GNU汇编版本 2.38 (x86_64-linux-gnu) 使用BFD版本 (GNU Binutils for Ubuntu) 2.38
COMPILER_PATH=/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/
LIBRARY_PATH=/usr/lib/gcc/x86_64-linux-gnu/11/:/usr/lib/gcc/x86_64-linux-gnu/11/../../../x86_64-linux-gnu/:/usr/lib/gcc/x86_64-linux-gnu/11/../../../../lib/:/lib/x86_64-linux-gnu/:/lib/../lib/:/usr/lib/x86_64-linux-gnu/:/usr/lib/../lib/:/usr/lib/gcc/x86_64-linux-gnu/11/../../../:/lib/:/usr/lib/
COLLECT_GCC_OPTIONS='-v' '-c' '-o' 'main.o' '-shared-libgcc' '-mtune=generic' '-march=x86-64' '-dumpdir' 'main.'
```