# master cmake 1

## 2.3 cmkae基本语法和语法 
使用cmake很简单,build过程被创建的CMakeLists.txt文件所控制,包含在子目录中的CMakelists.txt文件，CMakeLists文件应该描述了项目的结构,基本的命令的形式如下: 

`
command (args ...)
`

args 是用空白符分开的参数列表(参数如果包含空格，就用双引号包含下),CMake是大小写不敏感的(2.2之后)
所以你可以使用command或者COMMAND

CMake支持简单的变量比如字符串或者字符串列表，使用${变量}来引用, 多个参数可以使用`set`命令打包到列表中去，所有的其他命令会展开用空白符分隔来展开列表

举个栗子 ![举个栗子](http://img.mp.itc.cn/upload/20170320/81a3c11642104d87b0a2e33b2eaafdd3.jpeg) 

`set (Foo a b)` 命令的结果是将a,b,c包进Foo变量，所以如果传递`Foo`到另外一个命令`COMMAND(${Foo})`和`COMMAND(a b c)`是一样的

如果你想要将列表传递给命令作为一个单独的参数，只需要简单的用双引号包括以下就行

再举个栗子 ![再举个栗子](http://img.mp.itc.cn/upload/20170320/81a3c11642104d87b0a2e33b2eaafdd3.jpeg) 

`COMMAND("${Foo}")` 和 `COMMAND("a b c")`是一样的效果

系统环境变量和Windows的注册变量(注册表啥的吧，不懂:<)在CMake中都可以访问，要访问系统环境变量使用`$ENV{变量}`语法，CMake还可以在很多不同的命令中使用`[HKEY_CURRENT_URSER\\Software\\path1\\path2;key]`引用注册变量,这些路径是注册树和key

## 2.4 CMake Hello World

咱们来看个简单的吧:), 为了从源码中编译出一个可执行文件，CMake需要一个CMakeLists.txt文件，文件中需要包含两行:

`
project (Hello)
add_executable (Hello Hello.c)
`

为了编译出Hello可执行文件，咱们运行CMake来生成Makefiles或者巨硬的项目文件。
* `project`命令标识了结果的工作空间应该叫什么
* `add_executable`命令向编译程序添加了一个可执行的目标

如果你的项目需要不是特别多的源文件，只需要简单的修改下add_executable就好啦:

`
add_executable(Hello Hello.c File2.c File3.c File4.c)
`

`add_executable`只是CMake众多命令中的一件。咱们来看看一个更不清真的栗子:

```cmake
cmake_minimum_required(2.6)
project (HELLO)

set (HELLO_SRCS Hello.c File2.c File3.c)
if (WIN32)
	set (HELLO_SRCS ${HELLO_SRCS} WinSupport.c) 
else ()
	set (HELLO_SRCS ${HELLO_SRCS} UnixSupport.c) 
endif ()

add_executable (HELLO ${HELLO_SRCS})
#look for thr Tcl library
find_library (TCL_LIBRARY NAMES tcl tcl84 tcl83 tcl82 tcl80 PATHS /usr/lib /usr/local/lib)

if (TCL_LIBRARY) 
	target_link_library(Hello ${TCL_LIBRARY}) 
endif ()

```

这个栗子里边使用`set`命令来打包参数列表，使用`if`命令来添加各自平台的支持文件，最后使用了`add_executable`命令来编译可执行文件，`find_library`命令搜索TCL\_LIBRARY使用了不同的名称和不同的路径，使用一个`if`命令来判断TCL\_LIBRARY是否找到，如果找到了就把他link到可执行文件的target中，注意`#`作为一个注释来使用，从`#`到行尾都是注释部分


## 2.5 如何运行CMake

只要你系统里边装了CMake，使用它来编译项目很简单，CMake用两个主要的目录来编译项目:源码目录和二进制目录。源码目录里边包含源码和CMakeLists.txt文件。二进制目录就是放可执行文件，库，目标文件这些东西的，一般来说，CMake不会在源码目录里边写东西，只在二进制目录里边干活，如果你想让CMake在源码目录里边写东西的话:<(一般不要这么做吧，不明白什么时候需要这么干)，就把源码目录和二进制目录设置成一样的就妥了这个就是--in-source-build。相对的就是out-source-build。

CMake在所有平台上两种方式都是资瓷的。就是说你用out-source-build的时候你可以简单地删除掉所有编译期间生产的文件。拥有不同的编译路径和源码路径让资瓷对同一份代码的多个编译很简单，这个功能在你想要用不同的编译选项编译同一份代码的时候很有用。CMake还有一个QT的GUI(蛤，我为什么要用这个，反正我不用，我就不翻了)

#### 在控制台运行CMake

在命令行上，CMake可以当成一个交互的问答式的会话或者是一个无交互的程序。加上`-i`参数就可以启动交互模式，交互模式下Cmake将会为每一个参数询问你。CMake将会提供有效的默认值。CMake将问完所有的问题然后停止


使用无交互模式编译项目在不需要参数或者很少参数的时候很简单。对于大型的项目像VTK,使用gui或者cmake -i比较推荐。

使用CMake的无交互模式，首先cd到你想要放二进制的目录,in-source-build的话就直接运行cmake,加上-D来传递编译选项。out-source-build的话同样的操作，只是需要你添加上源文件的目录作为cmake的参数。然后使用`make`命令来编译你的项目，有些项目会包含 install 目标，你可以使用`make install`来安装他们

#### 指定编译器 

有时候在你的操作系统里边可能包好不止一个编译器或者编译器不在标准的路径下边，这个时候你告诉CMake你想要用的编译器在什么地方。有三种方法可以实现这个功能: 

* generator可以指定编译器
* 设置环境变量参数
* 设置一个缓存实例

默写generator可以绑定在特定的编译器，例如宇宙第一IDE的generator就是绑定到VS的编译器的。基于Makefile的generator会尝试一个常用的编译器列表。一般这个列表可以在这些文件中找到:

> Modules/CMakeDeterminCCompiler.cmake 
> Modules/CMakeDeterminCxxCompiler.cmake

这个列表可以被环境变量覆盖掉。

* CC 环境变量用来指定C编译器
* CXX 用来指定c++编译器

你可以使用——DCMake\_CXX\_COMPILER=cl直接在命令行上指定编译器。如果不设置这个的话，CMake会依次选择下边的这些编译器:

> c++ g++ CC aCC cl bcc xlC

CMake开始运行并且选好编译器之后,你可以修改缓存选项CMAKE\_CXX\_COMPILER和CMAKE\_C\_COMPILER修改选择的编译器，不过这种做法是不推荐的，这么做的问题主要是项目可能是你的配置可能已经运行了一些基于编译器的测试来判断一些支持特性。修改了编译器之后可能会导致一些不正确的结果。如果你必须修改编译器。重在一个新的空的二进制目录中进行。编译器的flags和链接器同意可以使用环境变量来修改。
设置LDFLAFGS将会为link flag初始化一个缓存值,同样的CXXFLAGS和CFLAGS将会初始化CMAKE\_CXX\_FLAGS和CMAKE\_C\_FLAGS

#### 依赖分析


