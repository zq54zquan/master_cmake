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

CMake拥有一个强大的内置C/C++依赖分析能力,CMake还支持部分的Java和Fortran依赖分析，因为IDE支持和维护了依赖信息，CMake对这些编译系统放弃了这部分功能，但是，Makefiels是不了解怎么样去计算和维护依赖信息的，对于这些系统，CMake为C/C++/Fortran自动计算依赖信息，CMake自动生成和维护依赖信息，这些都是自动完成的，如果一个项目是用CMake配置的，用户主需要运行Make，CMake会帮你完成剩余的工作，对于多处理器系统CMake完全支持并行编译

尽管用户不需要知道CMake是如何工作的，不过看一下项目的依赖信息文件也没啥坏处:),为每一个target创建的依赖文件保存在这四个文件中：

* depend.make
* flags.make
* build.make 
* DependInfo.cmake

> depend.make保存了目录中的所有目标文件的所有的依赖信息
> flags.make包含了编译的flags

这两个文件如果改变了，源码就会被重新编译

> DependInfo.cmakes是保持更新信息更新，并且包含了项目中包括哪些文件和这些文件使用的语言
> 最后编译依赖的规则保存在build.make中，如果依赖不是过期了，所有的依赖需要重新计算来保持信息永远是最新的

## 2.6 编辑 CMakeLists 文件

CMakeLists文件可以用几乎任意的文本编辑器来编辑,有些编辑器比如Notepad++内置了语法高亮和缩进支持，神的编辑器和编辑器之神也内置了语法和缩进支持

这段没啥营养啦，反正我用vim...

## 2.7 为CMake设置初始值
尽管CMake在交互模式下运行的很好，有时你也可能在没有GUI的时候设置下缓存条目，这种情况是很常见的当设置nightly dashboard(不明白这边)<a href="unknow0"></a>或者是你将创建很多编译树使用相同的cahce值，在这些情况下，CMake缓存可以以两种方式被初始化，第一种方式是使用CMake命令行下`-DCACHE_VAR:TYPE=VALUE`传递cache值

再举个栗子 ![再举个栗子](http://img.mp.itc.cn/upload/20170320/81a3c11642104d87b0a2e33b2eaafdd3.jpeg) 

考虑下边的nightly dashboard脚本:

```
#!/bin/tcsh
cd ${HOME}
#wipe out the old binary tree and then create it again
rm -rf Foo-Linux
mkdir Foo-Linux
cd Foo-Linux
#run cmake to setup the cache
cmake -DBUILD_TESTING:BOOL=ON <etc...> ../Foo
#generate the dashboard
ctest -D Nightly
```

同样的方法可以用在windows的批处理文件上

第二种方式是创建一个文件用来被CMake -C选项来加载。这种情况下取代-D这种方式的是使用一个文件来使用CMake解析，这个文件的语法就是标准的CMakeLists语法，通常来说就是一系列的`set`命令像这样的：

```cmake
#Build the vtkHybrid kit always
set (VTK_USE_HYBRID ON CACHE BOOL "doc" FORCE)
```

还有一种方法是你想要设置一个变量然后隐藏他，用户不会改变这个值，用下边的命令可以实现

```cmake
#Build the vtkHybrid kit always
set (VTK_USE_HYBRID ON CACHE BOOL "doc" FORCE)
mark_as_advanced (VTK_USE_HYBRID)
```
你可能会想直接编辑cache文件，或者去“初始化”一个项目通过给他一个初始的cache文件的方式，这有可能跑不出来，或者在以后出现问题，首先，CMake cahe的语法是修改的东西，第二，cache 文件包含了完整路径。

## 2.8 编译你的项目

运行完了CMake之后就可以准备编译了。如果你的编译系统是基于make的，你就cd到你的二进制目录然后运行make就行啦，如果是IDE的话就加载生产的项目文件，然后你按你平时的操作来就行了

另外一种选择就是在命令行中使用CMake的编译选项，这个选项就是一个便利的工具让你在命令行编译你的项目
运行下`cmake -build`你就知道怎么用了。

#### [不清楚的地方]

* [nightly dashboard](#unknow0)

