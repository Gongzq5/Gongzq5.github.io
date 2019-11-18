---
title: CMake 从0开始
date: 2019-09-06 10:03:12
tags: CMake
category: CMake
---

从CMake是什么讲到如何写一个简单的CMake工程，再到CMake的一些常用的变量和函数介绍。

希望可以教会大家入门CMake。

<!-- more -->

[TOC]

## CMake是什么

CMake的全称是：Cross Platform Make，即跨平台Make；CMake到底是什么呢？WikiPedia的解释是这样的：

> **CMake**是个一个[开源](https://zh.wikipedia.org/wiki/开源)的[跨平台](https://zh.wikipedia.org/wiki/跨平台)[自动化建构](https://zh.wikipedia.org/wiki/Build_automation)系统，用来管理软件建置的程序，并不依赖于某特定编译器，并可支持多层目录、多个应用程序与多个库。 
>
> :link: 链接：https://zh.wikipedia.org/wiki/CMake

比较难以理解吼，使用过命令行编译或者Unix的Makefile进行构建的同学应该大概可以类比一下，CMake就是一个写脚本然后可以生成Makefile一类的文件的程序。

> 只用IDE进行过工程开发的同学可能不太能理解，但是可以注意一下：大部分IDE需要新建工程，新建工程后会多出来一些工程文件（比如Eclipse，Visual Studio……都会自己建），里边描述了依赖的头文件路径啊，依赖的库文件的路径啊，文件之间的依赖关系啊什么的，这其实就是CMake会生成的东西。

所以简单地说， CMake是用来进行构建的一个东西（随便你叫它语言也好、程序也好，反正就是这么一个东西），写CMake的脚本，就可以生成Makefile，就可以用Unix的Make进行编译了。



听起来还挺酷的！那么问题来了，怎么写这些东西呢？、

首先我们来介绍一下CMake的文件编排方式和结构。



## CMake工程的文件和结构

CMake的文件大概可以分为几类：

* **CMakeLists.txt**：描述构建信息。CMake中负责构建的文件都命名为 **CMakeLists.txt**。通常来说，每个模块处于一个单独的文件夹中，这样的每个文件夹都有一个专职的CMakeLists.txt负责描述它的构建需求。
* **.cmake文件**：通常会放一些函数啊，或者专门负责定义一类变量什么的，可以在CMakeLists.txt中用`include`命令调用这一类文件，会顺序执行其中的语句。

也就是说，一个项目的工程目录可能是这样的：

```bash
RootDir
├─CMakeLists.txt		# 一个根目录下的顶层CMakeLists.txt
├─cmake/				# cmake文件夹，存放一些构建需要的脚本什么的
|  ├─utils.cmake		# 一个.cmake文件，utils表示存放了一些工具性的函数
├─build/				# build文件夹，用作存放生成构建中间+最终文件的目录
├─server/				# 一个子模块文件夹，比如：服务器程序，Server
│  ├─CMakeLists.txt		# 子模块通常也要一个CMakeLists.txt
|  ├─include/
│  |   ├─a.hpp
│  |   └─b.hpp
│  └─src/
│      ├─a.cpp
│      └─b.cpp
└─client/				# 另一个子模块，比如：客户端程序，Client
   ├─CMakeLists.txt		# 另一个子模块的CMakeLists.txt
   ├─include/
   |   └─c.hpp
   └─src/
       └─c.cpp
```

根目录下的CMakeLists.txt是整个CMake程序的入口，而子目录下的CMakeLists.txt会编译出两个可执行文件来（当然

那么CMakeLists.txt怎么写呢？



## 写一个顶层的CMakeLists.txt文件

顶层CMakeLists.txt文件是整个CMake构建的入口，因此通常来说会设置一些全局通用的变量啊，还要将所有的子目录包含进来。

顶层文件比子目录下的CMakeLists.txt多出来的（也是必不可少的）是如下的两句话：

```cmake
cmake_minimum_required(VERSION 3.12)
# cmake_minimum_required(VERSION major[.minor[.patch[.tweak]]] [FATAL_ERROR])
project(M_PROJECT	C)
# project(<PROJECT-NAME>
#        [VERSION <major>[.<minor>[.<patch>[.<tweak>]]]]
#        [LANGUAGES <language-name>...])
```

然后我们可以使用`add_subdirectory`来添加我们的子目录，比如：

```cmake
add_subdirectory(./client)
add_subdirectory(./server)
```

这样，CMake就会自动的读取这两个子目录下的CMakeLists.txt文件，然后进行解析。



## 使用CMakeLists.txt描述一个构建目标



在子目录下的每个CMakeLists.txt文件一般会用来描述一个具体的构建目标（当然，如果工程比较简单的话，直接放在顶层的CMakeLists.txt里也是完全OK的）。

我们来举个简单的栗子，比如如下的这条编译命令：

```bash
$ g++ main.cpp lib.cpp -o demo -I../include -lpthread -DA_PREDEF_MACRO=STH
```

> 解释一下，是将两个**cpp**源文件一起编译成一个可执行文件，叫做**demo**；我们还给这个编译命令添加了一个包含目录，是**../include**；还有一个预定义的宏，即`A_PREDEF_MACRO`，值为`STH`；同时还指定了一个链接库，即**pthread**；

我们把这个改编成一个CMakeLists.txt来看看。

首先，我们需要明确，我们的构建目标是一个可执行文件，叫做**demo**。所以，我们需要添加一个target。

> 这里的**目标（target）**是一个很重要的概念，不管是CMake还是Makefile，整个构建都是围绕着target来的。这个target可以是可执行文件，也可以是一个.a的静态库文件，或者.so的动态库，都是可以的。

这个`target`肯定需要指定源文件，因此CMake将添加源文件的操作与添加target放在一起，如下：

```cmake
add_executable(demo		main.cpp lib.cpp) 
# <=>
# g++ main.cpp lib.cpp -o demo
```

然后我们处理我们加入的其他的选项，比如添加包含目录`-I../include`，如下：

```cmake
target_include_directories(demo	PRIVATE ../include)
# <=>
# g++ ... -I../include
```

我们还给他添加了链接库，链接到`pthread`库`-lpthread`，如下：

```cmake
target_link_libraries(demo	PRIVATE pthread)
# <=>
# g++ ... -lpthread
```

我们还添加了编译期定义的宏变量`-DA_PREDEF_MACRO=STH`，如下：

```cmake
target_compile_options(demo	PRIVATE -DA_PREDEF_MACRO=STH)
# OR
# target_compile_definitions(demo PRIVATE A_PREDEF_MACRO=STH)
# <=>
# g++ ... -DA_PREDEF_MACRO=STH
```

把上边这几句话组合到一个CMakeLists.txt文件中，就完成了对如何编译`demo`这个可执行程序的完整描述。



## CMake的一些基础语法



### 显示帮助信息 `message()`

当需要打印信息时，我们会使用这个函数，举个栗子：

```cmake
message("Hello,world")
message(STATUS "A status code") 		  # STATUS 会在打印消息前加上两个-，比如 "-- ..."
message(AUTHOR_WARNING "A user warning")  # AUTHOR WARNING 会把这条消息按照警告的格式输出
message(FATAL_ERROR	"An error")           # FATAL_ERROR 会发送一个error然后退出程序
```

> 其实还有很多其他的标记，去看看文档吧：https://cmake.org/cmake/help/v3.3/command/message.html?highlight=message



### 变量

变量一般会用`set`来设置，比如

```cmake
set(target	demo)
```

这样，我们就定义了一个变量，叫做`target`，其值为**demo**。

那么如何访问这个变量的值呢？这一点上cmake和shell是非常相似的，我们可以通过`${}`来访问。比如：

```cmake
message(STATUS ${target}) # 打印：-- demo
# OR
message(STATUS "${target}") # 加双引号不影响解析，还是打印：-- demo
```

> 给`${target}`外边加个""是没有影响的，只是将其变成了一个真·字符串。

如果需要使用列表的话，有两种不同的写法，比如：

```cmake
set(aList  "demo1;demo2;demo3")
# or
set(bList  demo1 demo2 demo3) # 完全等价
```



### 条件语句

就是`if`啦，这是一个简单又不简单的语句，举个栗子：

```cmake
if (A EQUAL B)
	message("A equal to B")
elseif(A EQUAL C)
	message("A equal to C")
endif()
```

`if`里的那个判断语句只是一个简单的栗子，CMake还提供了好多的判断方式，多给几个栗子尝一尝：

```cmake
if(IS_DIRECTORY ../include) ...
if(${aString} MATCHES regex) ...
if(${aString} IN_LIST ${a_list}) ...
...
```

> 好多好多，具体去看官网的说明吧：https://cmake.org/cmake/help/v3.3/command/if.html?highlight=#command:if



### `foreach`循环

循环语句也好理解吼，举个栗子：

```cmake
foreach(element IN LISTS a_list)
    message("${arg}")
endforeach()
```

简单的循环了一个列表，`foreach`还有不少其他的遍历方式，比如：

```cmake
foreach(loop_var arg1 arg2 ...) ...
foreach(loop_var RANGE total) ...
foreach(loop_var IN ITEMS item1 item2) ...
```

还是蛮好理解的，也就不多做解释了

> 老规矩，看官网：https://cmake.org/cmake/help/v3.0/command/foreach.html?highlight=foreach



### 函数

函数有两种写法，一种是`macro`，另一种则是`function`。其实效果上是差不多的，我们都来举几个栗子。

```cmake
macro(m_macro cpu_type linux_name)
  # 假设就按照 m_macro(x86, ubuntu, a_unknown_parameter) 这样来调用
  message("${cpu_type}, ${linux_name}")  # x86, ubuntu
  # ARGC 保存了传入的参数数量
  message("${ARGC}")                     # 3
  # 可以使用 ARGV#n 来访问第n个参数
  message("${ARGV0}")                    # x86
  # ARGN 保存了我们在声明中没有声明的变量，就是最后的几个
  message("${ARGN}")                     # a_unknown_parameter
  # ARGV 保存了所有的参数列表
  message("${ARGV}")                     # x86;ubuntu;a_unknown_parameter
endmacro(m_macro)

function(m_function cpu_type linux_name)
  # 假设就按照 m_function(x86, ubuntu, a_unknown_parameter) 这样来调用
  message("${cpu_type}, ${linux_name}")  # x86, ubuntu
  # 和macro一样，也有那几个参数的控制
  message("${ARGC} & ${ARGV0} & ${ARGN} & ${ARGV}")
  # 3 & x86 & a_unknown_parameter & x86;ubuntu;a_unknown_parameter
endfunction(m_function)
```

> macro的官网说明：https://cmake.org/cmake/help/v3.0/command/macro.html
>
> function的官网说明：https://cmake.org/cmake/help/v3.0/command/function.html

根据官网的说法，在我们调用函数时，会首先将函数里定义的命令里的变量替换为我们传入的值，然后顺序执行所有的命令。

当然，函数和宏还是有一定的区别的，从官网找到的差别大概是函数会打开一个新的命名作用域， 而宏的话则是一个纯粹的替换，和上下文使用的作用域是完全相同的。这也是和我们平常使用的宏和函数是可以类比的。应该还是可以理解的。

这里有一篇文章讲的非常好，可以看一哈：https://juejin.im/post/5a8ab0e4f265da4e9d223972



## `find_package`函数

> 这是一个感觉需要讲的东西，用来寻找依赖的库

这个应用的场景通常是在我们需要自己指定依赖的第三方库时，比如我们需要依赖哪个公司提供给我们的一个so文件，我们会把这个so存放在我们的一个目录下（比如，**openGL的libGL.so**，在windows下叫做**opengl.lib**)，那么如何让我们的CMake程序找到这个依赖的so文件呢？这时候就要用到这个命令了。

我们先来回顾一下之前是如何写依赖库的，我们使用的是`target_link_libraries()`命令。

```cmake
add_executable(demo		main.cpp lib.cpp) 
target_link_libraries(demo  PRIVATE pthread)
```

这个pthread是系统库，不需要进行进一步的描述，如果不是系统自带的库的话，理所当然的，我们需要告诉CMake这个库所在的位置。比如使用libGL.so时，我们直接写

```cmake
target_link_libraries(demo  PRIVATE GL)
```

CMake十有八九是很懵逼的，什么GL，哪里来的GL？因此我们可以使用一个如下这样的命令告诉CMake这个库是什么，在哪里存放着。

### `find_libraries`函数

我们之前说过了一个很重要的概念叫`target`，我们可以编译出一个可执行文件或者一个库（lib或者so什么的）作为target。CMake也提供了一组函数，供我们导入现有的库作为CMake里的target。下面我们来举栗子：

```cmake
find_libraries(lib_gl GL ../libs) # 会去../libs目录下寻找名为libGL.so的文件，
                                  # 然后放在lib_gl这个变量里
add_library(opengl SHARED IMPORTED GLOBAL
            IMPORTED_LOCATION ${lib_gl}) # 表明是导入的库，会直接导入位置指定的文件作为目标
```

重点就是上面的哪个`IMPORTED`，表明是导入的库，无需编译构建。

这样我们就可以使用opengl这个名字作为依赖库了，举个栗子尝一尝：

```cmake
find_libraries(lib_gl GL ../libs)
add_library(opengl SHARED IMPORTED GLOBAL
            IMPORTED_LOCATION ${lib_gl})
# 然后就可以使用了
target_link_libraries(demo  PRIVATE opengl)
```

就可以顺畅的将这个添加到我们的工程里。

### 一种更通用的写法

通用的写法就是使用`find_package`命令了。

当我们使用的库比较多的时候，比如一个产品会提供好多个so文件，那么此时我们通常会使用一个类似命名空间的方法来控制，比如：

```cmake
target_link_libraries(demo  PRIVATE opengl::GL)
```

即将Opengl这个大命名空间下的GL库作为依赖。

> 随便怎么理解啦，理解为一个叫做**"opengl::GL"**的库大概也是可以的

`find_package(XXX ...)`命令会去一些目录里寻找一个命名为**findXXX.cmake**的脚本文件并运行它，我们通常会在这样的一个文件里写我们的寻找库文件的具体过程。

> 这些目录被定义在`CMAKE_MODULE_PATH`里，我们可以自己编辑这个变量，添加我们自己的目录进去

这个文件是可以完全由我们自己定义的，里边写什么都是很随意的了，CMake没有对这个做什么很严格的要求。但是通常来说，我们会定义以下的一些变量：

* `XXX_FOUND`：表明包已经找到了，完全OK，在下文中可以正常使用；
* `XXX_LIBRARIES`：一个列表，写明了所有导入的库在cmake中的名称；
* `XXX::lib_name`：每个都是一个单独的库文件，比如我们上面提到的`opengl::GL`；

还有一个常用的函数是：`FindPackageHandleStandardArgs`，方法如下：

```cmake
FIND_PACKAGE_HANDLE_STANDARD_ARGS(NAME [REQUIRED_VARS <var1>...<varN>])
# 举个栗子
find_package_handle_standard_args(OPENGL REQUIRED_VARS lib_GL)
```

> 当然这个方法的参数还有很多，请看官网链接：https://cmake.org/cmake/help/v3.0/module/FindPackageHandleStandardArgs.html

这个函数做了什么呢？其实就是验证和`lib_GL`放一起的这些变量有没有都找到，如果都找到的话，就会把`OPENGL_FOUND`设置为真，否则会报错表示缺少了一些需要的库。

具体找包的过程和上述的`find_libraries`是一样的，也就不太赘述具体的过程了，直接给个栗子尝尝（请详细的看下注释）：

```cmake
# findOPENGL.cmake
find_libraries(lib_gl GL ../libs)
find_package_handle_standard_args(OPENGL REQUIRED_VARS lib_GL)
add_library(OPENGL::GL SHARED IMPORTED GLOBAL
            IMPORTED_LOCATION ${lib_gl})
```

到了我们真正的`CMakeLists.txt`文件中，栗子如下：

```cmake
...
find_package(OPENGL REQUIRED GLOBAL)
...
if (NOT OPENGL_FOUND)
	...
endif()
...
target_link_libraries(demo  PRIVATE opengl::GL)
```

> 实际中，我们通常会在顶层文件中把所有依赖的包都找出来，会比较好统一管理



## CMake常用变量

CMake内置了很多变量，比如我们上一节提到的`CMAKE_MODULE_PATH`就是一个，还有一些常用的，我们直接列下来：

### 全局编译选项

* **CMAKE_C_FLAGS**：编译C程序时加入的编译器选项；
* **CMAKE_CXX_FLAGS**：同理，编译C++程序时的编译选项；

### 工作目录信息

* **CMAKE_CURRENT_SOURCE_DIR**：当前CMakeLists.txt所在的目录
* **EXECUTABLE_OUTPUT_PATH**：输出可执行文件的目录
* **LIBRARY_OUTPUT_PATH**：输出库文件的目录
* **CMAKE_CURRRENT_BINARY_DIR**：当前正在编译的目标要输出的目录

> 以上的这些变量通常都可以自己设置，比如可以自己指定可执行文件输出到哪里，自己指定要使用哪些全局的编译选项……

### CMake版本

* **CMAKE_MAJOR_VERSION**，CMAKE 主版本号,比如 2.4.6 中的 2
* **CMAKE_MINOR_VERSION**，CMAKE 次版本号,比如 2.4.6 中的 4
* **CMAKE_PATCH_VERSION**，CMAKE 补丁等级,比如 2.4.6 中的 6

### 系统信息

* **CMAKE_SYSTEM**，系统名称,比如 Linux-2.6.22
* **CMAKE_SYSTEM_NAME**，不包含版本的系统名,比如 Linux
* **CMAKE_SYSTEM_VERSION**，系统版本,比如 2.6.22
* **CMAKE_SYSTEM_PROCESSOR**，处理器名称,比如 i686.



## CMake常用方法

### `message`函数

最常用的当然就是我们提过的`message`啦，不在此处赘述了

### `FILE`函数

`file`方法可以对文件系统进行很多操作，比如读写，新建，重命名，哈希校验……具体的还是要看文档

> https://cmake.org/cmake/help/v3.0/command/file.html?highlight=file

有一个方法`GLOB`是我曾经踩过的坑，在这里提一嘴

```cmake
file(GLOB variable [RELATIVE path] [globbing expressions]...)
```

这里的`globbing expressions`和正则很像，但是真的不是正则，可以到维基上看一下什么叫`globbing`，千万不要简单的当作正则来用了。

### `INSTALL`方法

这个东西一说就很多了，但是也不是很难，大家直接看官网就好了



## The end but may be not the end

感觉想到的东西都已经说的差不多了，那就先到这里吧，想起来了会继续补充的，希望大家栗子吃的开心，也学到了一些CMake的小技能