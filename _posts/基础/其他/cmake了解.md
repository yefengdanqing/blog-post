---
title: cmake了解
toc: true
date: 2023-03-08 22:26:59
tags:
- 编译
categories:
- 工具
typora-root-url: ..\img
---

## 简介 

CMake 有两个主要阶段。第一个是“配置”步骤，其中 CMake 处理所有提供给它的输入并创建要执行的构建的内部表示。第二个阶段是“生成”步骤。在这个阶段创建实际的构建文件。第一步是 cmake -B build，称为配置阶段（configure），配置阶段可以通过 -D 设置缓存变量；第二步是cmake --build build，称为构建阶段，调用编译器来编译代码，-G 选项：指定要用的生成器

<!-- more -->

### cmake过程

#### 环境变量

在项目构建过程中都会使用 shell 级环境变量。项目通常有一个 PROJECT_ROOT 环境变量，它指向源代码树的根位置。环境变量也用于指向可选包或外部包。这种方法的问题在于，要使构建工作，每次执行构建时都需要设置所有这些外部变量。为了解决这个问题，CMake 有一个缓存文件，将构建所需的所有变量存储在一个地方。这些不是 shell 或环境变量，而是 CMake 变量。第一次为特定构建树运行 CMake 时，它会创建一个`CMakeCache.txt`存储该构建的所有持久变量的文件。由于该文件是构建树的一部分，因此在每次运行期间变量将始终对 CMake 可用。

#### 配置步骤

在配置步骤中，如果CMakeCache.txt在之前的运行中存在，CMake首先读取它。然后它读取CMakeLists.txt，在给CMake的源树的根中找到。在配置步骤中，CMakeLists.txt文件被CMake语言解析器解析。文件中的每个CMake命令都由命令模式对象执行。在此步骤中，可以使用include和add_subdirectory CMake命令解析其他CMakeLists.txt文件。对于CMake语言中可以使用的每个命令，CMake都有一个c++对象。命令示例包括add_library、if、add_executable、add_subdirectory和include。实际上，整个CMake语言都是通过调用命令来实现的。解析器只是将CMake输入文件转换为命令调用和作为命令参数的字符串列表。 

<!-- more -->

配置步骤本质上是“运行”用户提供的CMake代码。在执行了所有代码并计算了所有缓存变量值之后，CMake在内存中就有了要构建的项目的表示。这将包括为所选生成器创建最终构建文件所需的所有库、可执行文件、自定义命令和所有其他信息。此时，CMakeCache.txt文件将被保存到磁盘，以供将来运行CMake时使用。 

项目在内存中的表示是一组目标，这些目标只是一些可以构建的东西，比如库和可执行文件。CMake还支持自定义目标:用户可以定义他们的输入和输出，并提供在构建时运行的自定义可执行文件或脚本。CMake将每个目标存储在一个cmTarget对象中。这些对象依次存储在cmMakefile对象中，该对象基本上是源树给定目录中所有目标的存储位置。最终结果是包含cmTarget对象映射的cmMakefile对象树。

#### 生成步骤

一旦完成了配置步骤，就可以进行生成步骤。生成步骤是当CMake为用户选择的目标构建工具创建构建文件时。此时，目标(库、可执行文件、自定义目标)的内部表示形式被转换为IDE构建工具(如Visual Studio)的输入，或者由make执行的一组makefile。在配置步骤之后，CMake的内部表示是尽可能通用的，这样就可以在不同构建的工具之间共享尽可能多的代码和数据结构。

![【CMake流程概述】](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/process.png)

#### 构建级别

CMake具有许多内置的构建配置，可用于编译工程。 这些配置指定了代码优化的级别，以及调试信息是否包含在二进制文件中。

CMAKE_BUILD_TYPE

这些优化级别，主要有：

- Release —— 不可以打断点调试，程序开发完成后发行使用的版本，占的体积小。 它对代码做了优化，因此速度会非常快，

  在编译器中使用命令： `-O3 -DNDEBUG` 可选择此版本。

- Debug ——调试的版本，体积大。

  在编译器中使用命令： `-g` 可选择此版本。

- MinSizeRel—— 最小体积版本

  在编译器中使用命令：`-Os -DNDEBUG`可选择此版本。

- RelWithDebInfo—— 既优化又能调试。

  在编译器中使用命令：`-O2 -g -DNDEBUG`可选择此版本。

## 语法

### 变量

#### c++相关的cmake变量

******

```cmake
CMAKE_CXX_STANDARD
CMAKE_CXX_STANDARD_REQUIRED
CMAKE_CXX_FLAGS
CMAKE_CXX_FLAGS_RELEASE
CMAKE_CXX_FLAGS_DEBUG
```

CMAKE_CXX_STANDARD 变量

CMAKE_CXX_STANDARD_REQUIRED 是 BOOL 类型，可以为 ON 或 OFF，默认 OFF。他表示是否一定要支持你指定的 C++ 标准：如果为 OFF 则 CMake 检测到编译器不支持 C++17 时不报错，而是默默调低到 C++14 给你用；

> 最好是在 project 指令前设置 CMAKE_CXX_STANDARD 这一系列变量，这样 CMake 可以在 project 函数里对编译器进行一些检测，看看他能不能支持 C++17 的特性。请勿直接修改 CMAKE_CXX_FLAGS 来添加 -std=c++17

###### c++新标准相关选项

CMake支持传递一个变量给函数CMAKE_CXX_COMPILER_FLAG来编译程序。 然后将结果存储在您传递的变量中。编译器选项

CMAKE_C_COMPILER用于编译c代码的程序；CMAKE_CXX_COMPILER用于编译c++；CMAKE_LINKER用于链接二进制程序。

确定编译器是否支持标志后，即可使用标准cmake方法将此标志添加到目标。 在此示例中，我们使用CMAKE_CXX_FLAGS将标志（c++标准）传播给所有目标。

```cmake
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
```

###### 编译标志

这些编译设置都在CMAKE_CXX_FLAGS变量中。（C语言编译选项是CMAKE_C_FLAGS）

可以通过target_compile_definitions（）函数设置某个目标的编译标志。

```cmake
target_compile_definitions(cmake_examples_compile_flags
    PRIVATE EX3
)
```

编译器选项，还可以使用target_compile_options（）函数【不建议使用】

```cmake
target_compile_options(<target> [BEFORE]
  <INTERFACE|PUBLIC|PRIVATE> [items1...]
  [<INTERFACE|PUBLIC|PRIVATE> [items2...] ...])
```

是给 `target` 添加编译选项， `target` 指的是由 `add_executable()`产生的可执行文件或 `add_library()`添加进来的库。`<INTERFACE|PUBLIC|PRIVATE>`指的是`[items...]` 选项可以传播的范围， `PUBLIC and INTERFACE`会传播 `<target>`的 [INTERFACE_COMPILE_DEFINITIONS](https://cmake.org/cmake/help/v3.0/prop_tgt/INTERFACE_COMPILE_DEFINITIONS.html#prop_tgt:INTERFACE_COMPILE_DEFINITIONS) 属性， `PRIVATE and PUBLIC` 会传播 `target` 的 [COMPILE_DEFINITIONS ](https://cmake.org/cmake/help/v3.0/prop_tgt/COMPILE_DEFINITIONS.html#prop_tgt:COMPILE_DEFINITIONS)属性。

###### 默认变量

#### Cmake普通变量【可以把所有不确定的地方都套上一层引号】

***

```cmake
cmake [<options>] (<path-to-source> | <path-to-existing-build>)
cmake [(-D<var>=<value>)...] -P <cmake-script-file>
cmake --build <dir> [<options>] [-- <build-tool-options>...]
cmake -E <command> [<options>]
cmake --find-package <options>...
```

Cmakelists.txt

```cmake
cmake_minimum_required(VERSION 3.5) #设置CMake最小版本
project (hello_cmake) #设置工程名
add_executable(hello_cmake main.cpp) #生成可执行文件
```

```cmake
#PROJECT_SOURCE_DIR指工程顶层目录
#PROJECT_Binary_DIR指编译目录
#PRIVATE指定了库的范围，下一节讲
```

CMake语法指定了许多变量，可用于帮助您在项目或源代码树中找到有用的目录。 其中一些包括：

| Variable                         | InfoInfo--                                                   |
| -------------------------------- | ------------------------------------------------------------ |
| CMAKE_SOURCE_DIRCMAKE_SOURCE_DIR | 根源代码目录，工程顶层目录。暂认为就是PROJECT_SOURCE_DIR根源代码目录，工程顶层目录。暂认为就是PROJECT_SOURCE_DIR |
| CMAKE_CURRENT_SOURCE_DIR         | 当前处理的 CMakeLists.txt 所在的路径                         |
| PROJECT_SOURCE_DIR               | 工程顶层目录                                                 |
| CMAKE_BINARY_DIR                 | 运行cmake的目录。外部构建时就是build目录                     |
| CMAKE_CURRENT_BINARY_DIR         | The build directory you are currently in.当前所在build目录   |
| PROJECT_BINARY_DIR               | 暂认为就是CMAKE_BINARY_DIR                                   |
| PROJECT_NAME                     | 当前project（）设置的项目的名称。                            |
| CMAKE_PROJECT_NAME               | 由project（）命令设置的第一个项目的名称，即顶层项目。        |
| PROJECT_SOURCE_DIR               | 当前项目的源文件目录。                                       |
| PROJECT_BINARY_DIR               | 当前项目的构建目录。                                         |
| name_SOURCE_DIR                  | 在此示例中，创建的源目录为 `sublibrary1_SOURCE_DIR`, `sublibrary2_SOURCE_DIR`, and `subbinary_SOURCE_DIR` |
| name_BINARY_DIR                  | 本工程的二进制目录是`sublibrary1_BINARY_DIR`, `sublibrary2_BINARY_DIR`,和 `subbinary_BINARY_DIR` |



#### cmake cache变量

set(<variable> <value>... CACHE <type> <docstring> [FORCE])

- ariable：变量名称
- value：变量值列表
- CACHE：cache变量的标志
- type：变量类型，取决于变量的值。类型分为：BOOL、FILEPATH、PATH、STRING、INTERNAL
- docstring：必须是字符串，作为变量概要说明
- FORCE：强制选项，强制修改变量值，一般用来修改cache变量

使用：

1. 定义缓存变量时，可以不加FORCE选项：

   set(MY_GLOBAL_VAR_STRING_NOFORCE "abcdef" CACHE STRING "定义一个STRING缓存变量")

2. 修改缓存变量时，一定要加FORCE选项，否则修改无效：

   ```cmake
   # 修改定义时不加FORCE的缓存变量(不加FORCE选项方式修改)
   set(MY_GLOBAL_VAR_STRING_NOFORCE "modify_abcdef_without_force" CACHE STRING "修改一个STRING缓存变量")
   message("MY_GLOBAL_VAR_STRING_NOFORCE: ${MY_GLOBAL_VAR_STRING_NOFORCE}")
    
   # 修改定义时不加FORCE的缓存变量(加FORCE选项方式修改)
   set(MY_GLOBAL_VAR_STRING_NOFORCE "modify_abcdef_with_force" CACHE STRING "修改一个STRING缓存变量" FORCE)
   message("MY_GLOBAL_VAR_STRING_NOFORCE: ${MY_GLOBAL_VAR_STRING_NOFORCE}")
   # 参考：https://www.cnblogs.com/Braveliu/p/15614013.html
    
   ```

3. Cache变量都会保存在CMakeCache.txt文件中。

4. 不论定义或修改缓存变量时，建议都加上FORCE选项。结合1、2项所得。

### 函数

#### 日志

在前面的示例中，运行make命令时，输出仅显示构建状态。 要查看用于调试目的的完整输出，可以在运行make时添加make VERBOSE = 1或者set(CMAKE_VERBOSE_MAKEFILE ON)标志.在顶层`CMakeLists.txt`里set(CMAKE_VERBOSE_MAKEFILE ON)`或者 `cmake。再make VERBOSE=1 或 `cmake -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON。再  make`

最好：cmake -DCMAKE_RULE_MESSAGES:BOOL=OFF -DCMAKE_VERBOSE_MAKEFILE:BOOL=ON .   再   make --no-print-directory最后这种方式可以减少很多无用的信息，如：`make[1]: Entering directory` and `make[1]: Leaving directory`

注意，你可以使用 -DCMAKE_EXPORT_COMPILE_COMMANDS=ON 选项来查看最干净的编译选项，它会生成一个 compile_commands.json 文件。

想仔细体会一下，可以在CMakeLists中，利用message（）命令输出一下这些变量。

另外，这些变量不仅可以在CMakeLists中使用，同样可以在源代码.cpp中使用。

#### include_directories()

Include_directories (x/y)影响目录作用域。这个CMakeList中的所有目标，以及在调用点之后添加的所有子目录中的目标，都将把路径x/y添加到它们的包含路径中。 

Target_include_directories (t x/y)具有目标作用域—它将x/y添加到目标t的包含路径中。 

如果所有目标都使用包含目录，则需要前者。如果路径特定于目标，或者希望更好地控制路径的可见性，则需要后一种方法。后者来自target_include_directories()支持PRIVATE、PUBLIC和INTERFACE限定符

#### target_include_directories（）添加头文件

当您有其他需要包含的文件夹（文件夹里有头文件）时，可以使用以下命令使编译器知道它们： target_include_directories（）。 编译此目标时，这将使用-I标志将这些目录添加到编译器中，例如 -I /目录/路径

#### TARGET

在cmake中不管是生成library还是binary，都可以称作构建target

#### add_executable()

该命令通过给定的源文件来生成可执行Target；主要有三类target：

```cmake
add_executable (<name> [WIN32] [MACOSX_BUNDLE]
      [EXCLUDE_FROM_ALL]
      [source1] [source2 ...])
add_executable (<name> IMPORTED [GLOBAL])
add_executable (<name> ALIAS <target>)
```

##### **普通可执行目标文件**

```cmake
add_executable (<name> [WIN32] [MACOSX_BUNDLE]
      [EXCLUDE_FROM_ALL]
      [source1] [source2 ...])
通过指定的source列表构建出可执行目标文件。
```

* **`name`:**可执行目标文件的名字，在一个cmake工程中，这个名字必须全局唯一。
* **`[source1] [source2 ...]`:**构建可执行目标文件所需要的源文件。也可以通过[`target_sources()`](https://links.jianshu.com/go?to=https%3A%2F%2Fcmake.org%2Fcmake%2Fhelp%2Flatest%2Fcommand%2Ftarget_sources.html%23command%3Atarget_sources)继续为可执行目标文件添加源文件，要求是在调用`target_sources`之前，可执行目标文件必须已经通过`add_executable`或`add_library`定义了。

##### 导入可执行目标

```cmake
add_execuatable()
```



#### add_library（）

cmake变量BUILD_SHARED_LIB 是一个全局变量，主要是用于控制cmake是否可以生成动态so

默认情况下BUILD_SHARED_LIB变量打开状态为on，即默认使用add_library是创建的动态lib，值为on。此时除非是cmakelist文件中特别制定需要生成静态lib，否则默认就是生成的动态lib，如果此时想要强制生成静态库则需要使用,如果BUILD_SHARED_LIB 设置为off，则关闭生成动态so，默认为静态lib文件。

```CMAKE
ADD_LIBRARY(libname [SHARED|STATIC|MODULE] [EXCLUDE_FROM_ALL] source1 source2 ... sourceN) # 不需要写全lib.so, 只需要填写name,cmake系统会自动为你生成，libxxx.X
#EXCLUDE_FROM_ALL 参数的意思是这个库不会被默认构建，除非有其他的组件依赖或者手工构建。
```

###### 编译静态库/编译动态库

add_library（）函数用于从某些源文件创建一个库，默认生成在构建文件夹。 写法如下：

```cmake
add_library(hello_library STATIC
    src/Hello.cpp
)
```

在add_library调用中包含了源文件，用于创建名称为libhello_library.a的静态库。

> 如前面的示例所述，将源文件直接传递给add_library调用,这是modern CMake的建议,而不是先把Hello.cpp赋给一个变量）

add_library（）函数用于从某些源文件创建一个动态库，默认生成在构建文件夹。 写法如下：

```cmake
add_library(hello_library SHARED
    src/Hello.cpp
)
```

在add_library调用中包含了源文件，用于创建名称为libhello_library.so的动态库。

| NOTE | 如前面的示例所述，将源文件直接传递给add_library调用，这是modern CMake的建议。（而不是先把Hello.cpp赋给一个变量） |
| ---- | ------------------------------------------------------------ |
|      |                                                              |

###### 设置库的范围

```
ADD_LIBRARY(glog STATIC IMPORTED GLOBAL)
SET_PROPERTY(TARGET glog PROPERTY IMPORTED_LOCATION ${THIRD_PARTY_PATH}/lib/libglog.a)
ADD_DEPENDENCIES(extern_glog gflags)
ADD_DEPENDENCIES(glog extern_glog)
```

###### alais库

#### optional()

#### set()

set(<variable> <value>... CACHE <type> <docstring> [FORCE])

该命令可以为普通变量、缓存变量、环境变量赋值,通过unset()取消

环境变量

```cmake
set(ENV{<variable>} [<value>])
```

- `set(ENV{variable_name} value)` 设置环境变量
- `$ENV{variable_name}` 获取环境变量

#### 第三方库

CMake支持使用find_package（）函数查找这些工具的路径。这将从CMAKE_MODULE_PATH中的文件夹列表中搜索格式为“ FindXXX.cmake”的CMake模块。 在linux上，默认搜索路径将是/ usr / share / cmake / Modules。 在我的系统上，这包括对大约142个通用第三方库的支持。

#### 子项目

###### 引用子项目中的库

如果子项目创建了一个库，则其他项目可以通过在target_link_libraries（）命令中调用该项目的名称来引用该库。 这意味着您不必引用新库的完整路径，而是将其添加为依赖项。

```cmake
target_link_libraries(subbinary
    PUBLIC
        sublibrary1
)
```

或者，您可以创建一个别名目标，该目标允许您在上下文（其实就是某个目标的绰号）中引用该目标。

```cmake
add_library(sublibrary2)
add_library(sub::lib2 ALIAS sublibrary2)
target_link_libraries(subbinary
    sub::lib2
)
```

###### 包含子项目的目录

从cmake v3开始从子项目添加库时，无需将项目include目录添加到二进制文件的include目录中。

创建库时，这由target_include_directories（）命令中的作用域控制。 在此示例中，因为子二进制可执行文件链接了sublibrary1和sublibrary2库，所以当它们与库的PUBLIC和INTERFACE范围一起导出时，它将自动包含$ {sublibrary1_SOURCE_DIR} / inc和$ {sublibrary2_SOURCE_DIR} / inc文件夹。（这个地方设及到了PUBLIC和INTERFACE的使用，本电子书的CMake-scope是讲这个的）

#### protocol bufer

#### get_filename_component

cmake获取'文件名'的'特定'部分

#### ninja

#### string

字符串相关的操作，比如转换为小写、替换、查找

#### option()

#### configure_file()

将文件复制到另一个位置并修改其内容。

当然，这里的修改其内容也不是任意地修改，也是遵循一定的规则：将input文件复制到output文件，并在输入文件内容中的变量，替换引用为@VAR@或${VAR}的变量值。每个变量引用将替换为该变量的当前值，如果未定义该变量，则为空字符串。

可能有些绕头，再浅显一点：configure_file，复制一份输入文件到输出文件，替换输入文件中被@VAR@或者${VAR}引用的变量值。也就是说，让普通文件，也能使用CMake中的变量。

#### install()

```cmake
INSTALL(TARGETS targets...
[[ARCHIVE|LIBRARY|RUNTIME]
[DESTINATION <dir>]
[PERMISSIONS permissions...]
[CONFIGURATIONS
[Debug|Release|...]]
[COMPONENT <component>]
[OPTIONAL]
] [...])
```

DESTINATION定义了安装的路径，如果路径以/开头，那么指的是绝对路径，这时候CMAKE_INSTALL_PREFIX其实就无效了。如果你希望使用CMAKE_INSTALL_PREFIX来定义安装路径，就要写成相对路径，即不要以/开头，那么安装后的路径就是${CMAKE_INSTALL_PREFIX}/<DESTINATION定义的路径>

#### execute_process()

#### add_compile_options()

add_compile_options函数来设置编译器选项，不过它是针对所有编译器的（包括c和c++编译器）

add_compile_options是用来配置当前目录和子目录的所有目标文件的options。如果有一个库需要让所有的目标文件链接的时候，使用此命令非常方便。所有添加的options可以通过[`COMPILE_OPTIONS`](https://cmake.org/cmake/help/v3.17/prop_dir/COMPILE_OPTIONS.html#prop_dir:COMPILE_OPTIONS)属性查看。`add_compile_options`作用的范围太广，一般很少使用。

CMAKE_CXX_FLAGS是配置所有C++目标文件的flags。可以传递一些参数比如warnings的等级，使用的C++标准等。对C语言的目标文件没有效，因此用户可为他们两种文件设置不同的flags。



#### set_target_properties()

#### aux_source_directory(<dir> <variable>)

收集指定目录中所有源文件的名称，并将列表存储在提供的<variable>变量中，该命令旨在供使用显式模板实例化的项目使用。 模板实例化文件可以存储在Templates子目录中，并使用此命令自动收集，以避免手动列出所有实例化。

#### add_subdirectory(source_dir [binary_dir] [`EXCLUDE_FROM_ALL`]))

这个指令用于向当前工程添加存放源文件的子目录，并可以指定中间二进制和目标二进制存 放的位置。EXCLUDE_FROM_ALL 参数的含义是将这个目录从编译过程中排除，比如，工程 的 example，可能就需要工程构建完成后，再进入 example 目录单独进行构建(当然，你 也可以通过定义依赖来解决此类问题)。

**`source_dir`**
 **必选参数**。该参数指定一个子目录，子目录下应该包含`CMakeLists.txt`文件和代码文件。子目录可以是相对路径也可以是绝对路径，如果是相对路径，则是相对当前目录的一个相对路径。

**`binary_dir`**
 **可选参数**。该参数指定一个目录，用于存放输出文件。可以是相对路径也可以是绝对路径，如果是相对路径，则是相对当前输出目录的一个相对路径。如果该参数没有指定，则默认的输出目录使用`source_dir`。

**`EXCLUDE_FROM_ALL`**
 **可选参数**。当指定了该参数，则子目录下的目标不会被父目录下的目标文件包含进去，父目录的`CMakeLists.txt`不会构建子目录的目标文件，必须在子目录下显式去构建。`例外情况：当父目录的目标依赖于子目录的目标，则子目录的目标仍然会被构建出来以满足依赖关系（例如使用了target_link_libraries）`。

#### ExternalProject

*https://cmake.org/cmake/help/v3.5/module/ExternalProject.html* 

`ExternalProject_Add`有许多选项，可用于外部项目的配置和编译等所有方面

##### dir

If any of the above `..._DIR` options are not specified, their defaults are computed as follows. If the `PREFIX` option is given or the `EP_PREFIX` directory property is set, then an external project is built and installed under the specified prefix:

```
TMP_DIR      = <prefix>/tmp
STAMP_DIR    = <prefix>/src/<name>-stamp
DOWNLOAD_DIR = <prefix>/src
SOURCE_DIR   = <prefix>/src/<name>
BINARY_DIR   = <prefix>/src/<name>-build
INSTALL_DIR  = <prefix>
LOG_DIR      = <STAMP_DIR>
```

Otherwise, if the `EP_BASE` directory property is set then components of an external project are stored under the specified base:

```
TMP_DIR      = <base>/tmp/<name>
STAMP_DIR    = <base>/Stamp/<name>
DOWNLOAD_DIR = <base>/Download/<name>
SOURCE_DIR   = <base>/Source/<name>
BINARY_DIR   = <base>/Build/<name>
INSTALL_DIR  = <base>/Install/<name>
LOG_DIR      = <STAMP_DIR>
```

If no `PREFIX`, `EP_PREFIX`, or `EP_BASE` is specified, then the default is to set `PREFIX` to `<name>-prefix`. Relative paths are interpreted with respect to [`CMAKE_CURRENT_BINARY_DIR`](https://cmake.org/cmake/help/latest/variable/CMAKE_CURRENT_BINARY_DIR.html#variable:CMAKE_CURRENT_BINARY_DIR) at the point where `ExternalProject_Add()` is called.

##### include(ExternalProject)

##### ExternalProject_Add（）

函数创建一个自定义目标，以实现外部项目的下载、更新/修补、配置、构建、安装和测试步骤。它允许**在构建时**检索项目的依赖项。本文主要讲解常用的一些选项。其他选项看参考[官方文档](https://cmake.org/cmake/help/latest/module/ExternalProject.html)。

##### Download Step Options

 ###### DOWNLOAD_COMMAND <cmd>... 为<cmd>提供空字符串有效地禁用下载步骤。

多个url会轮询下载，直到有一个成功；

可以从git下载

Mercurial和cvs



##### Update Step Options

##### Patch Step Options

##### Configure Step Options

CONFIGURE_COMMAND < cmd >… 【该命令在该阶段优先级最高】

默认的configure命令运行CMake，并根据主项目提供一些选项。添加的选项通常只需要使用与主项目相同的生成器，但是可以使用CMAKE_GENERATOR选项来覆盖该选项。项目负责添加任何工具链细节，标志或其他它想从主项目中重用或指定的设置(参见下面的CMAKE_ARGS, CMAKE_CACHE_ARGS和CMAKE_CACHE_DEFAULT_ARGS)。



##### Build Step Options

##### Install Step Options

##### Test Step Options

##### Output Logging Options



##### 其他

*#为啥一定是大写呢*

ExternalProject_Add(SPDLOG 。。。）

#### FetchContent

*https://cmake.org/cmake/help/v3.11/module/FetchContent.html* 

[FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html) 在 configure 阶段下载文件或者项目

##### include(FetchContent)

##### FetchContent_Declare()

```cmake
FetchContent_Declare(
  <name>
  <contentOptions>...
  [SYSTEM]
  [OVERRIDE_FIND_PACKAGE |
   FIND_PACKAGE_ARGS args...]
)
```

<contentOptions>可以是ExternalProject_Add()命令理解的任何下载、更新或补丁选项。配置、构建、安装和测试步骤被显式禁用，因此与它们相关的选项将被忽略。SOURCE_SUBDIR选项是一个例外，参见c了解它如何影响行为的详细信息。

##### FetchContent_MakeAvailable

```cmake
FetchContent_MakeAvailable(<name1> [<name2>...])
```

##### 变量

FETCHCONTENT_BASE_DIR

在大多数情况下，所保存的详细信息没有指定与用于内部子构建、最终源代码和构建区域的目录相关的任何选项。通常最好将这些决定留给FetchContent模块来代表项目处理。FETCHCONTENT_BASE_DIR缓存变量控制收集所有内容填充目录的点，但在大多数情况下，开发人员不需要更改这一点。默认位置是${CMAKE_BINARY_DIR}/_deps，

#### find_package()

```cmake
find_package(<package> [version] [EXACT] [QUIET] [MODULE]
             [REQUIRED] [[COMPONENTS] [components...]]
             [OPTIONAL_COMPONENTS components...]
             [NO_POLICY_SCOPE])
```

>c++程序运行的时候会通过`LD_LIBRARY_PATH`这个环境变量寻找**除了默认路径之外的其他路径**的动态链接库，默认路径就是类似于`/usr/lib`这种的在系统库中的动态链接库文件。
>
>动态链接库的寻找顺序:
>
>- 1.编译目标代码时指定的动态库搜索路径；
>- 2.环境变量LD_LIBRARY_PATH指定的动态库搜索路径；
>- 3.配置文件/etc/ld.so.conf中指定的动态库搜索路径；
>- 4.默认的动态库搜索路径/lib和/usr/lib；

怎么find，find 的顺序是啥样的，find什么?得到什么信息

##### Module模式

查找路径有两个，在这两个路径下查找；具体是查找Find<package>.cmake

路径1：环境变量CMAKE_MODULE_PATH所代表的路径【CMAKE_MODULE_PATH】

路径2：/usr/local/share/cmake-3.23/Modules/【CMAKE_ROOT】

message(STATUS "CMAKE_MODULE_PATH = ${CMAKE_MODULE_PATH}")

 message(STATUS "CMAKE_ROOT = ${CMAKE_ROOT}")

##### Config模式

如果Module模式搜索失败，没有找到对应的`Find<LibraryName>.cmake`文件，则转入Config模式进行搜索。

只有在`find_package()`中指定**CONFIG**、**NO_MODULE**等关键字，或者**Module**模式查找失败后才会进入到**Config**模式。

CMake 对 Config file 的命名是有规定的，对于`find_package(ABC)`这样一条命令，CMake 只会去寻找`ABCConfig.cmake`或是`abc-config.cmake`。在 Linux 下寻找路径包括`/usr/lib/cmake`以及`/usr/lib/local/cmake。

![1746850-20201116091250447-2073530575](https://raw.githubusercontent.com/yefengdanqing/picture_bed/master/1746850-20201116091250447-2073530575.png)

#### target_link_libraries()

```
target_link_libraries(<target>
                      <PRIVATE|PUBLIC|INTERFACE> <item>...
                     [<PRIVATE|PUBLIC|INTERFACE> <item>...]...)
```

#### add_custom_command()

https://zhuanlan.zhihu.com/p/397394950

#### 属性

属性有点像变量，但它依附在某个 target 或者文件、目录上。许多属性的初始值来自于 CMAKE_ 开头的变量，例如设置 CMAKE_CXX_STANDARD，将会设置 target 的 CXX_STANDARD 属性初始值。
set_property 用于设置属性，get_property 用于获取属性。参看 cmake-properties 查阅有哪些属性



#### 流程控制

在 CMake 中可以配合其他工具

CCache，使用它来加快编译速度
Clang tidy，对你的代码做静态扫描
Include what you use, 检查冗余头文件
Clang-format，很不幸 CMake 没法直接使用 Clang-format，但是可以通过一些小技巧将它嵌入到 CMake 流程中来。参考这里还有这里

#### CMake Modules

###### CMAKE_BUILD_TYPE

\1. Debug\2. MinSizeRel\3. RelWithDebInfo\4. Release

```cmake
set (CMAKE_BUILD_TYPE "RelWithDebInfo")
```

#### 变量的作用域

* 函数外部定义的变量可以被函数内部多引用和调用，内层函数会从内到外依次搜索当前使用的变量
* 当前CMakeLists.txt中的变量的作用域仅在当前子目录有效；当前变量被值拷贝到子目录的CMakeLists.txt中，子目录中变量的作用域仅在子目录范围内，不会影响到父目录中变量的值
* 缓存变量在整个cmake工程的编译生命周期内都有效



\#define MACRO(r, data, i, elem)     info.BOOST_PP_TUPLE_ELEM(BOOST_PP_TUPLE_SIZE(elem), 0, elem) = std::move(std::get<i>(result)); BOOST_PP_SEQ_FOR_EACH_I(MACRO, _, SDK_DEVICE_DMP_ONEID_INFO)

#undef MACRO

#### 构建

##### cmake --build ./build --parallel 2

![img](https://img-blog.csdnimg.cn/d4d58035a46a4d41b29f75527603f627.png)

##### cmake --build ./build/ --verbose 调试构建

#### 单元测试

##### CTest

##### GTest

```cmake
gtest_discover_tests
```

### tips

* *cmake --help-command-list | grep find*【*cmake --help-command find_library*】
* cmake --help-variable-list  | grep CMAKE | grep HOST
* cmake --help-property-list | grep NAME【cmake --help-property OUTPUT_NAME】
* cmake --help-module FindBoost | head -40
* 升级了依赖库的版本后建议直接删除build目录，避免使用旧的cache变量

## 工作

## 问题

- [ ] ##### could not load cache【cmake --build ./build/】

  先用cmake -B ./build生成CMakeCache.txt等相关的问题，然后cmake --build ./build

- [ ] ##### PRIVATE和target_include_directories关系,INTERFACE_INCLUDE_DIRECTORIES

- [ ] ##### 怎么把动态库指定到build目录下，而不是lib下

- [ ] ##### generate_export_header

- [ ] ##### INTERFACE 、PUBLIC、PRIVATE

- [ ] ##### cmake 的生成器

- [ ] 创建可执行目标的CMake命令是什么？
  2.创建库目标的CMake命令是什么？
  \3. 如何指定库是静态链接还是动态链接？
  \4. 对象库有什么特别之处，它们在哪里派上用场？
  \5. 如何指定共享库的默认符号可见性？
  \6. 如何为目标指定编译器选项以及如何查看编译命令？

- [ ] ##### GoogleTest.cmake和FindGoogleTest.cmake的区别





