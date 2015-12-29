cmake
====

CMake是一个比make更高级的编译配置工具，它可以根据不同平台、不同的编译器，生成相应的Makefile或者vcproj项目。    
通过编写CMakeLists.txt，可以控制生成的Makefile，从而控制编译过程。     
CMake自动生成的Makefile不仅可以通过make命令构建项目生成目标文件，还支持安装（make install）、测试安装的程序是否能正确执行（make test，或者ctest）、生成当前平台的安装包（make package）、生成源码包（make package_source）、产生Dashboard显示数据并上传等高级功能，只要在CMakeLists.txt中简单配置，就可以完成很多复杂的功能，包括写测试用例。     
如果有嵌套目录，子目录下可以有自己的CMakeLists.txt。
总之，CMake是一个非常强大的编译自动配置工具，支持各种平台，KDE也是用它编译的，感兴趣的可以试用一下。    


[CMake Tutorial](http://www.cmake.org/cmake/help/cmake_tutorial.html)     
[CMake Tutorial - 中文](http://www.cnblogs.com/coderfenghc/archive/2013/01/20/2846621.html)

###A Basic Starting Point (Step 1)
大多数的源码都会编译为可执行文件，对于简单的工程只需要两行的CMakeLists.txt文件就行了，我们从这里开始我们的CMake之旅。

CMakeLists.txt文件如下：

	cmake_minimum_required (VERSION 2.6)
	project (Tutorial)
	add_executable(Tutorial tutorial.cxx)

add_executable()的第一个为编译生成的文件，后面为编译时需要的文件。    
注意到这个例子里面全部命令使用小写，CMake支持大写、小写或者混合大小写命令。    

tutorial.cxx文件中的源码会计算一个数的平方根，第一个版本的会非常简单，代码如下：

	// A simple program that computes the square root of a number
	#include <stdio.h>
	#include <stdlib.h>
	#include <math.h>
	int main (int argc, char *argv[])
	{
		if (argc < 2)
		{
			fprintf(stdout,"Usage: %s number\n",argv[0]);
			return 1;
		}
		double inputValue = atof(argv[1]);
		double outputValue = sqrt(inputValue);
		fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);
		return 0;
	}
	
**添加一个版本号和配置头文件**

我们要添加的第一个功能就是为我们的可运行程序和项目提供一个版本号。     
你可以在源码中专门做这件事，CMakeLists文件提供更灵活的方式来实现。
   
添加了版本号的CMakeLists文件如下：

	cmake_minimum_required (VERSION 2.6)
	project (Tutorial)
	# The version number.
	set (Tutorial_VERSION_MAJOR 1)
	set (Tutorial_VERSION_MINOR 0)
 
	# configure a header file to pass some of the CMake settings
	# to the source code
	configure_file (
		"${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
		"${PROJECT_BINARY_DIR}/TutorialConfig.h"
	)
 
	# add the binary tree to the search path for include files
	# so that we will find TutorialConfig.h
	include_directories("${PROJECT_BINARY_DIR}")
 
	# add the executable
	add_executable(Tutorial tutorial.cxx)

在我们的源码树中编写配置文件之后，我们还需要把路径添加到包含文件的搜索路径中。在我们的源码树中创建一个TutorialConfig.h.in文件，内容如下：

	// the configured options and settings for Tutorial
	#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
	#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@

当CMake配置这个头文件的时候，使用CMakeLists文件中的变量替换@Tutorial_VERSION_MAJOR@ 和 @Tutorial_VERSION_MINOR@ 。  
接下来修改tutorial.cxx文件包含配置好的头文件来使用版本号。修改后的源码如下：

	// A simple program that computes the square root of a number
	#include <stdio.h>
	#include <stdlib.h>
	#include <math.h>
	#include "TutorialConfig.h"
 
	int main (int argc, char *argv[])
	{
		if (argc < 2)
		{
			fprintf(stdout,"%s Version %d.%d\n",
			argv[0],
			Tutorial_VERSION_MAJOR,
			Tutorial_VERSION_MINOR);
			fprintf(stdout,"Usage: %s number\n",argv[0]);
			return 1;
		}
		double inputValue = atof(argv[1]);
		double outputValue = sqrt(inputValue);
		fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);
		return 0;
	}
主要的变化是包含了TutorialConfig.h头文件，在Usage信息中输出版本号。

**注**：修改CMakeLists.txt中的版本变量, 如
	
	set (Tutorial_VERSION_MAJOR 1)
会配置一个变量Tutorial_VERSION_MAJOR = 1。    
然后通过TutorialConfig.h.in文件中的定义，既可以在cmake配置时，替换对应的位置，这样，在生成的TutorialConfig.h中，就有：

	// the configured options and settings for Tutorial
	#define Tutorial_VERSION_MAJOR 1
	#define Tutorial_VERSION_MINOR 0

这样就实现了CMakeLists.txt中的变量到c语言程序中变量的转换！


-------

至此一个简单的项目就创建完成了，我们再源码中创建build目录，使用命令行进入build目录。

运行:     
`mkdir build`     
`cd build`     
`cmake ..`    
`make`     
即可在build文件夹中生成项目，通过为CMake添加-G参数来生成不同平台的makefile文件。

运行Tutorial会提示带版本号的Usage信息，输入Tutorial 25 可以计算25的平方根为5.

**为什么要新建build文件夹**     
这样生成的中间文件，都在build文件夹中，不会影响源代码。
`cmake ..`指令，将调用build外面(即源代码)的CMakeLists.txt，
执行相关配置(比如生成Makefile, 生成根据in文件生成头文件, 检查并配置相关路径等等)。


###Adding a Library (Step 2)

现在试着给项目加入一个库。这个库将包含我们用来实现计算给定数字平方根的相关代码。   
可执行文件将使用这个库中的函数，来代替使用标准库函数( sqrt() )。    
在这里，我们把库文件放在一个子目录中(MathFunctions)。这里需要在CMakeLists.txt文件中加入一行：

	add_library(MathFunctions mysqrt.cxx)

ADD_LIBRARY将添加一个库到目标之中。如果不进行设置，使用ADD_LIBRARY且没有指定库类型，默认编译生成的库是静态库。
这里即表示，添加一个MathFunctions库到编译目标中，其中这个库是由mysqrt.cxx源文件编译生成的。

 
源代码mysqrt.cxx文件有一个mysqrt()函数，提供和标准库函数sqrt()同样的功能。    
为了使用我们自己的库函数，这里在顶层CMakeLists文件中添加add_subdirectory，这样就会编译这个子目录的内容。CMake会循环的查找从当前目录到SUBDIRS列出的任何子目录的文件。
同时在子目录中添加了MathFunctions/mysqrt.h文件，使得可以找到函数原型。
最后的改变，就是将新库文件添加到可执行文件目标Tutorial的链接库中去。
改变后的CMakeLists.txt文件的最后几行，如下所示：

	include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
	add_subdirectory (MathFunctions) 
 
	# add the executable
	add_executable (Tutorial tutorial.cxx)
	target_link_libraries (Tutorial MathFunctions)


INCLUDE_DIRECTORIES( "dir1" "dir2" ... ) ，头文件路径，相当于编译器参数 -Idir1 -Idir2。    
TARGET_LINK_LIBRARIES( target-name lib1 lib2 ...)，设置单个目标需要链接的库。    

现在，我们试着加一个是否使用MathFunctions库的选项。目前的代码其实没有必要配置这个选项，但是以后可能需要使用其他依赖第三方代码的库需要使用。因此使用选项来配置库是很有必要的。

第一步，在顶层CMakeLists.txt文件中添加一个选项。

	# should we use our own math functions?
	option (USE_MYMATH "Use tutorial provided math implementation" ON) 
	
这个选项将在CMkae GUI中显示，供用户配置。并且默认的值为ON。
这个选项的值将被存储在缓存cache中，使得用户没有必要每次都重新配置一遍选项才能编译项目。

下一步，就是使得编译和链接过程中，根据选项的值判断如何操作。
为了实现这个，我们把CMakeLists文件的最后改为：

	# add the MathFunctions library?
	#
	if (USE_MYMATH)
		include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
		add_subdirectory (MathFunctions)
		set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
	endif (USE_MYMATH)
 
	# add the executable
	add_executable (Tutorial tutorial.cxx)
	target_link_libraries (Tutorial  ${EXTRA_LIBS})
        
 即，如果USE_MYMATH配置为ON，则执行if内部的语句。
 将包含库头文件，添加子目录，将库添加到EXTRA_LIBS变量中，供后面的链接语句target_link_libraries使用。
 
 
　　这里用USE_MYMATH设置来决定是否MathFunctions应该被编译和执行。注意到，要用一个变量（在这里是EXTRA_LIBS）来收集所有以后会被连接到可执行文件中的可选的库。这是保持带有许多可选部件的较大型工程干净清爽的一种通用的方法。源代码对应的改变相当直白，如下所示：
   
	// A simple program that computes the square root of a number
	#include <stdio.h>
	#include <stdlib.h>
	#include <math.h>
	#include "TutorialConfig.h"
	#ifdef USE_MYMATH
	#include "MathFunctions.h"
	#endif
 
	int main (int argc, char *argv[])
	{
		if (argc < 2)
		{
			fprintf(stdout,"%s Version %d.%d\n", argv[0],
			Tutorial_VERSION_MAJOR,
			Tutorial_VERSION_MINOR);
			fprintf(stdout,"Usage: %s number\n",argv[0]);
			return 1;
		}
 
		double inputValue = atof(argv[1]);
 
		#ifdef USE_MYMATH
		double outputValue = mysqrt(inputValue);
		#else
		double outputValue = sqrt(inputValue);
		#endif
 
		fprintf(stdout,"The square root of %g is %g\n", inputValue, outputValue);
		return 0;
	}
	
在源代码中，我们也使用了USE_MYMATH。这个宏是由CMake通过TutorialConfig.h.in配置文件中的下述语句行提供给源代码的：

	#cmakedefine USE_MYMATH

这样，如果USE_MYMATH在CMake中被配置为ON，则在生成的TutorialConfig.h头文件中，将有USE_MYMATH的宏定义。


----

###Installing and Testing (Step 3)

下一步我们会为我们的工程引入安装规则以及测试支持。    

安装规则相当直白，对于MathFunctions库，我们通过向MathFunctions的CMakeLists文件添加如下两条语句来设置要安装的库以及头文件：

	install (TARGETS MathFunctions DESTINATION bin)
	install (FILES MathFunctions.h DESTINATION include)

　对于应用程序，在顶层CMakeLists文件中添加下面几行，它们用来安装可执行文件以及配置头文件：	
	
	#添加安装目标
	install (TARGETS Tutorial DESTINATION bin)
	install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)
	
这就是要做的全部； 现在你应该可以构建tutorial工程了。      
然后，敲入命令`make install`（或者从IDE中构建INSTALL目标），然后它就会安装需要的头文件，库以及可执行文件到指定的bin/, include/目录。     
其中CMake的变量CMAKE_INSTALL_PREFIX用来确定这些文件被安装的根目录，可能为/usr/local/bin, 也可能是/usr/bin等。

添加测试同样也只需要相当浅显的过程。在顶层CMakeLists文件的的尾部补充许多基本的测试代码来确认应用程序可以正确工作。

	# 应用程序是否运行?
	add_test (TutorialRuns Tutorial 25)
 
	# 它是否对25做了开平方运算
	add_test (TutorialComp25 Tutorial 25)
  
	set_tests_properties (TutorialComp25 
	PROPERTIES PASS_REGULAR_EXPRESSION "25 is 5")
 
	# 它是否能处理是负数作为输入的情况
	add_test (TutorialNegative Tutorial -25)
	set_tests_properties (TutorialNegative
	PROPERTIES PASS_REGULAR_EXPRESSION "-25 is 0")
 
	# 它是否可以处理较小的数字。
	add_test (TutorialSmall Tutorial 0.0001)
	set_tests_properties (TutorialSmall
	PROPERTIES PASS_REGULAR_EXPRESSION "0.0001 is 0.01")
 
	# 用法信息是否可用？
	add_test (TutorialUsage Tutorial)
	set_tests_properties (TutorialUsage
	PROPERTIES 
	PASS_REGULAR_EXPRESSION "Usage:.*number")

第一个测试用例仅仅用来验证程序可以运行，没有出现段错误或其他的崩溃，并且返回值必须是0。这是CTest所做测试的基本格式。     
余下的几个测试都是用PASS_REGULAR_EXPRESSION 测试属性来验证测试代码的输出是否包含有特定的字符串。    
因为特殊境况的输入，会导致程序异常，输出一些特定的错误信息，这里就是要检查这些情况。    
在本例中，测试样例用来验证计算得出的平方根与预定值一样；    
 当指定错误的输入数据时，要打印用法信息。     
 如果你想要添加许多测试不同输入值的样例，你应该考虑创建如下所示的宏：

	#定义一个宏来简化添加测试的过程，然后使用它
	macro (do_test arg result)
		add_test (TutorialComp${arg} Tutorial ${arg})
		set_tests_properties (TutorialComp${arg}
			PROPERTIES PASS_REGULAR_EXPRESSION ${result})
	endmacro (do_test)<br># 做一系列基于结果的测试
	do_test (25 "25 is 5")
	do_test (-25 "-25 is 0")
	
对于每个do_test宏调用，都会向工程中添加一个新的测试用例；宏参数是测试名、函数的输入以及期望结果。

------

###Adding System Introspection (Step 4)

下一步，让我们考虑向我们的工程中引入一些依赖于目标平台上可能不具备的特性的代码。在本例中，我们会增加一些依赖于目标平台是否有log或exp函数的代码。当然，几乎每个平台都有这些函数；但是对于tutorial工程，我们假设它们并非如此普遍。如果该平台有log函数，那么我们会在mysqrt函数中使用它去计算平方根。我们首先在顶层CMakeLists文件中使用宏CheckFunctionExists.cmake测试这些函数的可用性：

	# does this system provide the log and exp functions?
	include (CheckFunctionExists.cmake)
	check_function_exists (log HAVE_LOG)
	check_function_exists (exp HAVE_EXP)

如果CMake在对应平台上找到了它们，我们修改TutorialConfig.h.in来定义这些值；如下：

	// does the platform provide exp and log functions?
	#cmakedefine HAVE_LOG
	#cmakedefine HAVE_EXP

这些log和exp函数的测试要在TutorialConfig.h的configure_file命令之前被处理，这一点很重要。最后，在mysqrt函数中，如果log和exp在当前系统上可用的话，我们可以提供一个基于它们的可选的实现：

	// if we have both log and exp then use them
	#if defined (HAVE_LOG) && defined (HAVE_EXP)
		result = exp(log(x)*0.5);
	#else // otherwise use an iterative approach
		. . .
	
###Adding a Generated File and Generator (Step 5)

在本节，我们会展示你应该怎样向一个应用程序的构建过程中添加一个生成的源文件。     
在本范例中，我们会创建一个预先计算出的平方根表作为构建过程的一部分。MathFunctions子路径下，一个新的MakeTable.cxx源文件来做这件事。

	// A simple program that builds a sqrt table 
	#include <stdio.h>
	#include <stdlib.h>
	#include <math.h>
 
	int main (int argc, char *argv[])
	{
		int i;
		double result;
 
		// make sure we have enough arguments
		if (argc < 2)
		{
			return 1;
		}
  
		// open the output file
		FILE *fout = fopen(argv[1],"w");
		if (!fout)
		{
			return 1;
		}
  
		// create a source file with a table of square roots
		fprintf(fout,"double sqrtTable[] = {\n");
		for (i = 0; i < 10; ++i)
		{
			result = sqrt(static_cast<double>(i));
			fprintf(fout,"%g,\n",result);
		}
 
		// close the table with a zero
		fprintf(fout,"0};\n");
		fclose(fout);
		return 0;
	}

注意到这个表是由合法的C++代码生成的，并且被写入的输出文件的名字是作为一个参数输入的。下一步是将合适的命令添加到MathFunction的CMakeLists文件中，来构建MakeTable可执行文件，然后运行它，作为构建过程的一部分。完成这几步，需要少数的几个命令，如下所示：

	# first we add the executable that generates the table
	add_executable(MakeTable MakeTable.cxx)
 
	# add the command to generate the source code
	add_custom_command (
		OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
		COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
		DEPENDS MakeTable
	)
 
	# add the binary tree directory to the search path for 
	# include files
	include_directories( ${CMAKE_CURRENT_BINARY_DIR} )
 
	# add the main library
	add_library(MathFunctions mysqrt.cxx ${CMAKE_CURRENT_BINARY_DIR}/Table.h  )

首先，MakeTable的可执行文件也和其他被加入的文件一样被加入。     
然后，我们添加一个自定义命令来指定如何通过运行MakeTable来生成Table.h。     
这是通过将生成Table.h增加到MathFunctions库的源文件列表中来实现的。     
我们还必须增加当前的二进制路径到包含路径的清单中，这样Table.h可以被找到并且可以被mysqrt.cxx所包含。当该工程被构建后，它首先会构建MakeTable可执行文件。然后它会运行MakeTable来生成Table.h文件。     
最后，它会编译mysqrt.cxx（其中包含Table.h）来生成MathFunctions库。    
到目前为止，拥有我们添加的完整特性的顶层CMakeLists文件看起来像是这样：
	
	cmake_minimum_required (VERSION 2.6)
	project (Tutorial)
 
	# The version number.
	set (Tutorial_VERSION_MAJOR 1)
	set (Tutorial_VERSION_MINOR 0)
 
	# does this system provide the log and exp functions?
	include (${CMAKE_ROOT}/Modules/CheckFunctionExists.cmake)
 
	check_function_exists (log HAVE_LOG)
	check_function_exists (exp HAVE_EXP)
 
	# should we use our own math functions
	option(USE_MYMATH "Use tutorial provided math implementation" ON)
 
	# configure a header file to pass some of the CMake settings
	# to the source code
	configure_file (
		"${PROJECT_SOURCE_DIR}/TutorialConfig.h.in"
		"${PROJECT_BINARY_DIR}/TutorialConfig.h"
	)
 
	# add the binary tree to the search path for include files
	# so that we will find TutorialConfig.h
	include_directories ("${PROJECT_BINARY_DIR}")
 
	# add the MathFunctions library?
	if (USE_MYMATH)
		include_directories ("${PROJECT_SOURCE_DIR}/MathFunctions")
		add_subdirectory (MathFunctions)
		set (EXTRA_LIBS ${EXTRA_LIBS} MathFunctions)
	endif (USE_MYMATH)
 
	# add the executable
	add_executable (Tutorial tutorial.cxx)
	target_link_libraries (Tutorial  ${EXTRA_LIBS})
 
	# add the install targets
	install (TARGETS Tutorial DESTINATION bin)
	install (FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h" DESTINATION include)
 
	# does the application run
	add_test (TutorialRuns Tutorial 25)
 
	# does the usage message work?
	add_test (TutorialUsage Tutorial)
	set_tests_properties (TutorialUsage
		PROPERTIES
		PASS_REGULAR_EXPRESSION "Usage:.*number"
	)
 
 
	#define a macro to simplify adding tests
	macro (do_test arg result)
		add_test (TutorialComp${arg} Tutorial ${arg})
		set_tests_properties (TutorialComp${arg}
		PROPERTIES PASS_REGULAR_EXPRESSION ${result}
	)
	endmacro (do_test)
 
	# do a bunch of result based tests
	do_test (4 "4 is 2")
	do_test (9 "9 is 3")
	do_test (5 "5 is 2.236")
	do_test (7 "7 is 2.645")
	do_test (25 "25 is 5")
	do_test (-25 "-25 is 0")
	do_test (0.0001 "0.0001 is 0.01")
TutorialConfig.h文件看起来像是这样：

	// the configured options and settings for Tutorial
	#define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
	#define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
	#cmakedefine USE_MYMATH
 
	// does the platform provide exp and log functions?
	#cmakedefine HAVE_LOG
	#cmakedefine HAVE_EXP
	
###Building an Installer (Step 6)

###Adding Support for a Dashboard (Step 7)


