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
* **`if(name1 IS_NEWER_THAN name2)`**

	最后修改时间文件1比文件2更新，表达式为真

* **`if(variable MATCH regex)`**
* **`if(string MATCH regex)`**
	
	如果给定的字符串或者变量的值符合正则，表达式为真

`EQUAL`,`LESS`,和`GREATER`可以用作数值比较

`STRLESS`,`STREQUAL`,和`STRGREATER` 可以用作字符字面值的比较

`VERSION_LESS`,`VERSION_EQUAL`和`VERSION_GREATER`可以用来做像`major.[.minor[.patch.[.tweak]]]`这样的版本比较，像C/C++一样，这些表达式可以组合起来使用来创建更多复杂的比较，举个栗子，考虑下边的这些情况:

```cmake
if ((1 LESS 2) AND (3 LESS 4))
	message("sequence of numbers")
endif()

if (1 AND 3 AND 4)
	message("series of true values")
endif(1 AND 3 AND 4)

if (nor 0 AND 3 AND 4)
	message ("a flase value")
endif(NOT 0 AND 4 AND 4)

if (0 OR 3 AND 4)
	message ("or statements")
endif (0 OR 3 AND 4)


if (EXISTS ${PROJECT_SOURCE_DIR}/help.txt AND COMMAND IF)
	message("help text")
endif （EXISTS ${PROJECT_SOURCE_DIR/help.txt AND COMMAND IF}）

set (foobar 0)
if (NOT DEFINED foobar)
	message ("foobar is not defined")
endif (NOT DEFINED foobar)

if (NOT DEFINED fooba)
	message ("fooba not defined")
endif
```

在复合的`if`语句中，有一个指定运算符的执行的顺序，在下边的表达式中，`NOT`会在`AND`之前运行，而不是相反的顺序，因此表达式的结果是`false`，打印的语句也不会执行，如果`AND`先运行的话，结果就是`true`了：

```cmake
if (NOT 0 AND 0)
	message ("this line is nerver executed")
endif (NOT 0 AND 0)
```

CMake 定义了操作符的执行顺序，比如括号内的表达式先支持性，`EXISTS`,`COMMAND`,`DEFINED`和相似的前缀运算符先执行，然后是想`EQUAL`,`LESS`,`GREATER`,`STREQUAL`,`STRLESS`,`STRGREATER`和`MATCHS`运算符，之后运行`NOT`运算符，最后是`AND`和`OR`,哪些具有相同执行级别的运算符比如`AND`和`OR`将会从左到右执行，一旦所有的表达式都执行并得到整个表达式的结果，CMake将`ON`,`1`,`YES`,`TRUE`,`Y`当做`true`,将`OFF`,`0`,`NO`,`FALSE`,`N`,`NOTFOUND`,`*-NOTFOUND`,`IGNORE`当做false，计算的值的大小写不敏感的，所有`true`,`True`和`TRUE`是相同的

咱们现在再来看下另外一种控制流命令，`foreach`,`while`,`macro`和`function`命令是减少你CMakeLists文件大小并保证可维护性的的最好的命令，`foreach`命令可以让你在一个列表中重复运行一组CMake命令，我们看看下边VTK中的栗子：

```cmake
foreach (tfile 
		TestAnisotropicDiffusion2D
		TestButterworthLowPass
		TestButterworthHighPass
		TestCityBlockDistance
		TestConvolve
		)

add_test (${tfile}-image ${VTK_EXECUTABLE}
		${VTK_SOURCE_DIR}/Tests/rtImageTest.tcl
		${VTK_SOURCE_DIR}/Tests/${tfile}.tcl
		-D ${VTK_DATA_ROOT}
		-V Baseline/Imaging/${tfile}.png
		-A ${VTK_SOURCE_DIR}/Wrapping/Tcl
		)
endforeach (tfile)
```

`foreach`的第一个参数是在循环中使用的变量名。剩余的参数是用来做循环的列表的值，在栗子中的`foreach`循环只是一个CMake命令`add_test`,在`foreach`的循环体内部任何时候都可以使用循环变量(`tfile`)来指代列表中在当前迭代中的元素，在第一次迭代中，${tfile}将会被`TestAnisotropicDiffusion2D`替换，下一个迭代`${tfile}`将会被`TestButterworthLowPass`替代,`foreach`将会继续循环知道所有参数都被处理完为止

值得一提的是，`foreach`可以嵌套使用，循环变量在任意表达式之前被替换成列表中的值，这意味着，在`foreach`的循环体中可以使用循环变量来创建新的变量名，下边的代码循环变量`tfile`被展开，然后和`_TEST_RESULT`组合起新的变量，新变量的名字被展开和并当场`if`的条件表达式来运算:

```cmake
if (${${tfile}_TEST_RESULT} MATCHES FAILED)
	message ("Test ${tfile} failed")
endif()
```

** 原文中`if`表达式的参数是`${${tfile}}_TEST_RESULT} MATCHES FAILED`,应该是有问题的 **


`while`命令提供了一个基于条件判断的循环，条件表达式的格式和`if`是一样的，考虑下边CTest中使用的栗子，注意CTest在内部更新了`CTEST_ELAPSED_TIME`的值：

```cmake
################################################################################
#  run paraview and ctest dashboards for 6 hours
# 
while (${CTEST_ELAPSED_TIME} LESS 36000)
	set (START_TIME ${CTEST_ELAPSED_TIME})
	ctest_run_script ("dash1_ParaView_vs71continuous.cmake")
	ctest_run_script ("dash1_cmake_vs71continuous.cmake")
endwhile()
```

`foreach`和`while`命令可以让你处理重复的任务，`macro`和`function`命令支持把散落在CMakeLists文件中的重复的任务打包。一旦他们被定义，任何在定义之后CMakeLists文件都可以使用他们

CMake中的function和C/C++中的function很相似，你可以传递参数，在方法内部参数变成了变量，就像其他的标准变量`ARGC`,`ARGV`,`ARGN`和`ARGV0`,`ARGV1`等等，在function内部，你就在一个新的变量作用域中了，有点像你使用`add_subdirectorty`命令进入了一个新的子目录使用新的变量作用域一样，所有定义的变量在方法调用中还是被定义的，但是任何在对这些变量的改变或者新的变量都只存在于方法内部，当方法返回之后，这些都会消失不见，更简单点说，当你调用一个方法是，一个新的变量作用域被push到栈顶，当你从方法返回时，作用域被pop掉

 function 的第一参数是方法名称，剩下的参数都是function的正式参数啦：

```cmake
function (DetermineTime _time) 
# pass the result up to whatever invoked this
	set (${_time} "1:23:45" PARENT_SCOPE)
endfunction()

DetermineTime( current_time ) 
if (DEFINED current_time)
	message(STATUS "the time is now: ${current_time}")
endif
```

注意这个栗子中的`_time`被用作返回值的名称，`set`命令使用`_time`的**值**。这个栗子中就是`current_time`,最后`set`命令使用`PARENT_SCOPE`选项来设置变量的值到父作用域

`macro`的定义和调用在默写方面是和`function`是一样的，主要的不同点在于，`macro`没有push和pop新的变量作用域，并且参数不是作为变量，而是作为字符串在运行前被替换的，这些很像C/C++里边的方法和宏的区别。

第一个参数作为`macro`的名字，后边的作为参数

```cmake
#define a simple macro

macro (assert TEST COMMENT)
	if (NOT ${TEST})
		message ("Assertion failed: ${COMMENT}")
	endif(NOT ${TEST})
endmacro(assert)

#use the macro
find_library (FOO_LIB foo /usr/local/lib)
assert (${FOO_LIB} "Unable to find library foo")
```
上边的简单例子中创建了一个叫做`assert`的宏，这个宏使用了两个参数，第一个是用于测试的值，第二个是如果测试失败打印的字符串，`macro`的宏体内是一个简单的`if`命令和一个`message`命令。`macro`用一个`endmacro`来作为结束，`macro`可以像一个普通的命令一样被调用，在上边的例子中，如果`FOO_LIB`没找到，就会显示一条信息来表示错误情况

`macro`命令同样支持定义使用参数列表的宏，当你想要用可变参数或者多签名(???)时会很有用。可变参数可以使用`ARGC`,`ARGV0`,`ARGV1`等来引用，`ARGV0`表示宏的第一个参数，`ARGV1`代表下一个，以此类推，你甚至可以使用最少的正式参数和可变参数.下边举个栗子：

```cmake
# define a macro that takes at least two arguments
# (thr formal arguments) plus an optional third argument

macro (assert TEST COMMENT)
	if (NOT ${TEST})
		message ("Assertion failed :${COMMENT}")
		# if called with three arguments then also write the 
		# message to a file specified as the third argument
		if (${ARGC} MATCHES 3) 
			file (APPEND ${ARGV2} "Assertion failed: ${COMMENT}")
		endif (${ARGC} MATCHES 3)
	endif (NOT ${TEST})
endmacro (assert)

#use the macro
find_library (FOO_LIB foo /usr/local/lib)
assert (${FOO_LIB} "Unable to find library foo")
```

CMake有连个命令用于打断执行流，`break`命令可以打断`foreach`和`while`循环的正常执行，`return`命令将从方法或者listfile的正常结束之前返回


#
在这个栗子中，`TEST`和`COMMENT`是两个必备的参数，像这个例子中一样这些必备参数可以使用名称来引用,也可以使用`ARGV0`,`ARGV1`,如果你想要把参数当成列表来使用，你可以使用`ARGV`和`ARGN`变量，`ARGV`(相对于`ARGV0`,`ARGV1`等等)是一个宏的全参数列表，`ARGN`是正式参数之后的参数列表，在宏体内，你可以使用`foreach`命令来遍历`ARGV`或者`ARGN`


## 4.4 正则表达式

有些CMake命令比如`if`和`string`会使用正则或者使用正则作为一个参数，在简单的模式下，一个正则是一系列的字符用于匹配搜索确切的字符，不过，很多时候我们不知道确切的字符串是什么，或者只知道字符串的开头和结尾，因为有好几种不同的方式来指定正则，CMake的标准制动如下:

正则可以使用标准的字母数字组合来指定，并使用下边的这些元字符：

`^` 匹配以一行或者一个字符串的开始

`$` 匹配一行或一个字符串的结尾

`.` 匹配一个处理换行外的任意字符

`[]` 匹配中间的任意字符

`[^ ]` 匹配不在符号中间的任意字符

`[ - ]` 匹配范围

`*` 匹配前一个模式0次或多次

`+` 匹配前一个模式1次货多次

`?` 匹配前一个模式0次或1次

`()` 保存匹配的表达式，后边可以再用

`(|)` 匹配|左边或者右边的模式

注意上边的这些元字符可以在单一的模式中使用不止一次用来创建一个复杂的搜索模式，举个栗子，`[^ab1-9]`匹配任意的不是以a,b或者数字开头的字符序列,下边的栗子可以帮助我们了解正则的用法:

- `^hello`只匹配哪些以`hello`开头的字符串，他可以匹配`"hello there"`, 但是不匹配`"hi,\n hello there"`.

- `long$` 匹配 `long`结尾的字符串，他匹配`so long`,不匹配`long ago`

- `t..t..g` 匹配一个`t`然后两个字符，另一个`t`和连个字符，最后是一个`g`,可以匹配`testing`或者`test again`, 但是不能匹配`toasting`.

- `[1-9ab]`匹配1-9数字和a,b, 他可以匹配`hello 1`或者`begin` 但是不匹配`no-match`

- `[^1-9ab]`匹配任意的非1-9的数字和非a,b的字母，他**不**匹配`1ab2`和`b2345a`,但是可以匹配`brrh`和`b`


