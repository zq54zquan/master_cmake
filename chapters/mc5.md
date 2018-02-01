# 编写CMakeLists文件

这一章主要覆盖了为你的软件编写CMakeLists文件一些基础，我们将会学习到你在大多数项目中遇到的所有的基本命令和一些问题,也会学习下周末把现有的UNIX或者Windows项目转换到CMake上来，因为CMake能处理非常复杂的项目，所以大多数项目中你会发现这一章的内容完全够用啦，CMake是被CMakeLists文件驱动的，CMakeList文件决定一切，比如Cache中的值，需要被编译的文件。最后会讨论下怎么让CMakeLists文件健壮并且可维护，基础的语法和关键概念都已经在上边的章节说过了，这一章将张开这些概念并且引入一些新的东西

## 4.1 CMake 语法

CMakeLists文件遵守一个简单的语法规则(注释，命令，空白符)，注释是从一个`#`开始的知道行尾，命令是由命令名称，小括号，空白符分割的参数组成的，含有空白符的参数用双引号括起来，反斜线可以做转移符，后边的一些列子会帮你解决语法上困惑，你可能回想CMake为什么不用先用的语言的语法来实现，比如Python啥的，主要的母的是想让CMake不要依赖其他的东西来运行，要是依赖了这些语言的话，安装就会很大,也需要依赖特定版本的语言，我们自己干的话就没有性能和兼容问题啦

## 4.2 基本命令

上一章已经介绍了蛮多基础的基础命令了，这一章将会重新来看下并且要展开来讲下，第一个命令是顶层CMakeLists文件要包含的`PROJECT`命令，这个命令不仅可以给项目命名还能指定工程使用的语言他的语法如下:

```
project (projectname [CXX] [C] [Java] [NONE])

```

如果没有指定语言，CMake默认会支持C和C++，如果指定了NONE，CMake将会不包含任何语言支持，当CPP被支持的时候C的支持也会加载的

对于项目中每一个`project`命令,CMake会创建一个顶层的IDE的project文件，这个项目会包含CMakeLists文件中的所有Target，添加子目录需要使用`add_subdirectory`命令，如果`EXCLUDE_FROM_ALL`选项被用在`add_subdirectory`命令中的话，生产的项目将不会出现在顶层的Make文件或者IDE的项目文件中。这个功能在你想要让你的子目录的项目和你的主项目没啥关系的时候很有用,举个栗子，假设你的工程有很多例子项目,你可以使用这个功能来为你的例子生产编译文件，但是在正常的编译中不会包含例子目录

`set`命令可能是用的最多的命令，他可以用来定义和修改变量和列表，和`set`相对的，`remove`和`separate_arguments`命令，`remove`命令可以用来从变量列表里边删除值，`separate_arguments`可以用来把一个单一的变量值用空白分割成一个列表

`add_executable`和`add_library`命令是用来定义库和可执行文件的编译和源码的，加的源码可以包含头文件，反正基于makefile的generators是会简单的省略掉


## 4.3 流控制

在很多方面写一个CMakeList文件就和写一段程序是一毛一样的，所以和其他的编程语言一样，CMake为咱提供了控制流结构：

* 条件判断语句(例如:`if`等)
* 循环结构(例如:`foreach`和`while`等)
* 程序流程定义(例如`macro`和`function`等)

首先我们看下`if`命令，在很多方面和在其他编程语言是一样的，计算表达式的值，根据结果运行命令的体内的代码或者运行可选的`else`中的代码。举个栗子：

```cmake
if (FOO)
# do something here
else (FOO)
# do something else
endif(FOO)
```
一个你可能注意到的不一样的地方就是`if`中的条件在`else`和`endif`中都会重复写一次，这个是可选的，在这本书里边你将看到两种风格的代码，你可以自己选择：

```cmake
if (FOO)
# do something here 
else()
# do something else
endif()
```

`if`命令有一些使用的限制，它不支持一些像`${FOO} && ${BAR} || ${FUBAR}`这样的C类型的表达式，作为替代，它支持一些表达式的子集，大多数情况下，这些就够啦，`if`命令支持:

* **`if(variable)`**

	如果变量的值不是空，0，FALSE, OFF或者NOTFOUND
* **`if(NOT variable)`** 

	如果变量的值是空，0， FALSE, OFF或者NOTFOUND
* **`if(variabl1 AND variable2)`**	
	
	变量同时为真，表达式为真
* **`if(varibale1 OR variable2)`** 

	变量一个为真，表达式为真
* **`if(COMMAND command-name)`**

	如果给定的命令名称是一个可调用的命令，表达式为真
* **`if(DEFINED variable)`**
	
	如果变量被set过，不管这个值是多少，表达式为真

* **`if(EXISTS file-name)`**
* **`if(EXISTS directory-name)`**
	
	如果文件或者目录存在，表达式为真

* **`if(IS_DIRECTORY directory-name)`**
* **`if(IS_ABSOLUTE name)`**
	
	如果名称是一个目录或者一个绝对路径，表达式为真
* **`if(name1 IS\_NEWER_THAN name2)`**

	最后修改时间文件1比文件2更新，表达式为真

* **`if(variable MATCH regex)`**
* **`if(string MATCH regex)`**
	
	如果给定的字符串或者变量的值符合正则，表达式为真

`EQUAL`,`LESS`,和`GREATER`可以用作数值比较
	


	
	
