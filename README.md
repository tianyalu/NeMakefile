# NeMakefile MakeFile走读与语法

## 一、概念
### 1.1 makefile
> makefile 定义了一系列的规则来指定哪些文件需要先编译，哪些文件需要重新编译，如何进行链接等操作。  
> makefile 就是“自动化编译”，告诉make命令如何编译和链接。

### 1.2 makefile组成
> 显示规则：说明如何生成一个或多个目标文件；  
> 隐晦规则：makefile自动推导；  
> 变量定义：当makefile执行时，变量会被扩展到相应的引用位置上；   
> 文件指示: include；指定编译有效部分；编译多行命令  
> 注释：行注释-> #    \#

### 1.3 makefile的规则
> target: 目标文件，可以使Object File,也可以是执行文件，还可以是标签（Lable)；  
> prerequisites: 依赖文件，即要生成那个target所需要的文件或其它target；  
> command：make要执行的命令；
```c++
target ... : prerequisites ...
    command
或者：
target ... : prerequisites ... ; command
```
**示例：**  
```c++
#当前目录存在main.c, tool.c, tool.h 三个文件
#下面是makefile文件内容
main: main.o tool.o
    gcc main.o tool.o -o main
.PHONY: clean
clean:
    -rm main *.o
------------------------------
//执行 make 后输出如下
cc -c -o main.o main.c
cc -c -o tool.o tool.c
gcc main.o tool.o -o main
//并且生成了一个可执行文件main
```
说明：  
> -o指定可执行文件的名称。  
> clear: 标签，不会生成”clean"文件，这样的target称为“伪目标”，伪目标的名称不能喝文件名重复。clean一般放在文件最后。  
> .PHONY: 显示地指明clean是一个“伪目标”。    

实操：  
![image](https://github.com/tianyalu/NeMakefile/blob/master/show/make_file_command.png)  

### 1.4 makefile工作原理
默认情况下，输入make命令后：  
> make会在当前目录下找名字叫“Makefile” 或 “makefile” 的文件。  
> 如果找到，它会找文件中第一个目标文件（target），并把这个target作为最终的目标文件，如前面示例中的 “main”。  
> 如果 main 文件不存在，或 main 所依赖的 .o 文件的修改时间要比main文件要新，那么它会执行后面所定义的命令来生成main文件。  
> 如果 main 文件所依赖的 .o 文件也存在，那么make会在当前文件中找目标为 .o 文件的依赖性，若找到则根据规则生成 .o 文件。  
> make 再用 .o 文件声明make的终极任务，也就是执行文件 “main” 。  

### 1.5 make工作流程
GUN的make工作时的执行步骤如下：
> 1.读入所有的Makefile。  
> 2.读入被include的其它Makefile。  
> 3.初始化文件中的变量。  
> 4.推导隐晦规则，并分析所有规则。  
> 5.为所有的目标文件创建依赖关系链。  

> 6.根据依赖关系，决定哪些目标要重新生成。  
> 7.执行生成命令。  

## 二、语法
### 2.1 makefile中使用变量
> 为了 Makefile 的易维护，在Makefile中我们可以使用变量。Makefile中的变量就是一个字符串，理解为C语言中的宏可能会更好。
> 比如，声明一个变量：objects, 随后，就可以在Makefile中方便地以 “$(objects)” 的方式使用这个变量了。
```c++
objects = main.o tool.o

main: $(objects)
    gcc $(objects) -o main 

.PHONY: clean
clean:
    rm main $(objects)
--------------------------
//执行make 后输出如下
cc -c -o main.o main.c
cc -c -o tool.o tool.c
gcc main.o tool.o -o main
```
实操：  
![image](https://github.com/tianyalu/NeMakefile/blob/master/show/make_file_variable.png)  

### 2.2 引用其它的Makefile
> 使用include关键字可以把其它的Makefile包含进来，include语法格式： `include <filename>`
```c++
# 语法格式
include <filename>
# 举个例子，假如有这样几个Makefile： a.mk, b.mk, c.mk, 还有一个文件叫# foo.make, 以及一个变量$(bar), 其包含了 e.mk, f.mk 

include foo.make *.mk $(bar)
# 等价于
include foo.make a.mk b.mk c.mk e.mk f.mk 

# 如果文件找不到，而你希望make时不理会那些无法读取的文件而继续执行
# 可以在include前加一个减号 “-”， 如：
-include <filename>
```

### 2.3 环境变量MAKEFILES
> 如果当前环境中定义了环境变量 MAKEFILES，那么make会把这个变量中的值作为一个类似于include的动作。这个变量中的值是其它的Makefile，用空格分隔。但是它和include不同的是，从这个环境变量中引入的Makefile的“目标”不会起作用，如果环境变量中定义的文件发生错误，make也不会理会。但是建议不要使用这个环境变量，因为只要这个变量一被定义，那么当使用make时，所有的Makefile都会受到它的影响。当Makefile出现了奇怪的问题时，可以查看当前环境中有没有定义这个变量。

### 2.4 Makefile预定义变量
变量名   | 描述               | 默认值
------- | ------------------ | -- 
CC      |C语言编译器的名称     | cc
CPP     |C语言预处理器的名称   | $(CC) -E
CXX     |C++语言编译器的名称   | g++
RM      |删除文件程序的名称    | rm -f
CFLAGS  |C语言编译器的编译选项  | 无
CPPFLAGS|C语言预处理器的编译选项| 无
CXXFLAGS|C++语言编译器的编译选项| 无

### 2.5 Makefile自动变量
自动变量 | 描述
--      | --
$*      | 目标文件名称，不包含扩展名
$@      | 目标文件名称，包含扩展名
$+      | 所有的依赖文件，以空格隔开，可能包含有重复的文件
$^      | 所有的依赖文件，以空格隔开，不重复
$<      | 依赖项中第一个依赖文件的名称
$?      | 依赖项中所有比目标文件新的依赖文件

### 2.6 Makefile函数
* 不带参数
```c++
define FUNC
$(info echo "hello")
endef

$(call FUNC)
---------------------
输出：hello

```
实操：  
![image](https://github.com/tianyalu/NeMakefile/blob/master/show/make_file_fun_noparam.png)  

* 带参数
```c++
define FUNC1
$(info echo $(1) $(2))
endef

$(call FUNC1,hello,world)
-------------------------
输出： hello world
```
实操：  
![image](https://github.com/tianyalu/NeMakefile/blob/master/show/make_file_fun_param.png)  

