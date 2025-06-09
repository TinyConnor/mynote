## 1.简介
```shell
target ... : prerequisites ...
    command
    ...
    ...
```
- target        - 目标文件, 可以是 Object File, 也可以是可执行文件
- prerequisites - 生成 target 所需要的文件或者目标
- command       - make需要执行的命令 (任意的shell命令), Makefile中的命令必须以 \[tab\] 开头

1. 显示规则 :: 说明如何生成一个或多个目标文件(包括 生成的文件, 文件的依赖文件, 生成的命令)
2. 隐晦规则 :: make的自动推导功能所执行的规则
3. 变量定义 :: Makefile中定义的变量
4. 文件指示 :: Makefile中引用其他Makefile; 指定Makefile中有效部分; 定义一个多行命令
5. 注释     :: Makefile只有行注释 "#", 如果要使用或者输出"#"字符, 需要进行转义, "\#"

**工作流程：**
1. 读入主Makefile (主Makefile中可以引用其他Makefile)
2. 读入被include的其他Makefile
3. 初始化文件中的变量
4. 推导隐晦规则, 并分析所有规则
5. 为所有的目标文件创建依赖关系链
6. 根据依赖关系, 决定哪些目标要重新生成
7. 执行生成命令

---
## 2.初级语法
#### 2.1通配符
- *     :: 表示任意一个或多个字符
- ?     :: 表示任意一个字符
- \[...\] :: ex. \[abcd\] 表示a,b,c,d中任意一个字符, \[^abcd\]表示除a,b,c,d以外的字符, \[0-9\]表示 0~9中任意一个数字
- ~     :: 表示用户的home目录
#### 2.2路径搜索
指定了 VPATH 之后, 如果当前目录中没有找到相应文件或依赖的文件, Makefile 回到 VPATH 指定的路径中再去查找..

VPATH 使用方法:

- vpath \<directories\>            :: 当前目录中找不到文件时, 就从\<directories\>中搜索
- vpath \<pattern\> \<directories\>  :: 符合\<pattern\>格式的文件, 就从\<directories\>中搜索
- vpath \<pattern\>                :: 清除符合\<pattern\>格式的文件搜索路径
- vpath                          :: 清除所有已经设置好的文件路径

```shell
# 示例1 - 当前目录中找不到文件时, 按顺序从 src目录 ../parent-dir目录中查找文件
VPATH src:../parent-dir   

# 示例2 - .h结尾的文件都从 ./header 目录中查找
VPATH %.h ./header

# 示例3 - 清除示例2中设置的规则
VPATH %.h

# 示例4 - 清除所有VPATH的设置
VPATH
```

#### 2.3变量定义
*=* 和  *:=* 的区别在于, := 只能使用前面定义好的变量, = 可以使用后面定义的变量
```shell
OBJS = programA.o programB.o
OBJS-ADD = $(OBJS) programC.o
# 或者
OBJS := programA.o programB.o
OBJS-ADD := $(OBJS) programC.o
```

#### 2.4变量替换
```shell
# Makefile内容
SRCS := programA.c programB.c programC.c
OBJS := $(SRCS:%.c=%.o)

all:
    @echo "SRCS: " $(SRCS)
    @echo "OBJS: " $(OBJS)
---
# bash中运行make
$ make
SRCS:  programA.c programB.c programC.c
OBJS:  programA.o programB.o programC.o
```

#### 2.5变量追加+=
```shell
# Makefile内容
SRCS := programA.c programB.c programC.c
SRCS += programD.c

all:
    @echo "SRCS: " $(SRCS)

# bash中运行make
$ make
SRCS:  programA.c programB.c programC.c programD.c
```

#### 2.6变量覆盖override
作用是使 Makefile中定义的变量能够*覆盖 make 命令参数中指定*的变量

- override \<variable\> = \<value\>
- override \<variable\> := \<value\>
- override \<variable\> += \<value\>

```shell
# Makefile内容 (没有用override)
SRCS := programA.c programB.c programC.c

all:
    @echo "SRCS: " $(SRCS)

# bash中运行make
$ make SRCS=nothing
SRCS:  nothing

#################################################

# Makefile内容 (用override)
override SRCS := programA.c programB.c programC.c

all:
    @echo "SRCS: " $(SRCS)

# bash中运行make
$ make SRCS=nothing
SRCS:  programA.c programB.c programC.c
```

#### 2.7 目标变量
作用是使*变量的作用域仅限于这个目标(target)*, 而不像之前例子中定义的变量, 对整个Makefile都有效.

- \<target ...\> :: \<variable-assignment\>
- \<target ...\> :: override \<variable-assignment\> (override作用参见 变量覆盖的介绍)

```shell
# Makefile 内容
SRCS := programA.c programB.c programC.c

target1: TARGET1-SRCS := programD.c
target1:
    @echo "SRCS: " $(SRCS)
    @echo "SRCS: " $(TARGET1-SRCS)

target2:
    @echo "SRCS: " $(SRCS)
    @echo "SRCS: " $(TARGET1-SRCS)

# bash中执行make
$ make target1
SRCS:  programA.c programB.c programC.c
SRCS:  programD.c

$ make target2     <-- target2中显示不了 $(TARGET1-SRCS)
SRCS:  programA.c programB.c programC.c
SRCS:
```

#### 2.8命令前缀
- 不用前缀 :: 输出执行的命令以及命令执行的结果, 出错的话停止执行
- 前缀 @   :: 只输出命令执行的结果, 出错的话停止执行
- 前缀 -   :: 命令执行有错的话, 忽略错误, 继续执行

```shell
# Makefile 内容 (不用前缀)
all:
    echo "没有前缀"
    cat this_file_not_exist
    echo "错误之后的命令"       <-- 这条命令不会被执行

# bash中执行 make
$ make
echo "没有前缀"             <-- 命令本身显示出来
没有前缀                    <-- 命令执行结果显示出来
cat this_file_not_exist
cat: this_file_not_exist: No such file or directory
make: *** [all] Error 1

###########################################################

# Makefile 内容 (前缀 @)
all:
    @echo "没有前缀"
    @cat this_file_not_exist
    @echo "错误之后的命令"       <-- 这条命令不会被执行

# bash中执行 make
$ make
没有前缀                         <-- 只有命令执行的结果, 不显示命令本身
cat: this_file_not_exist: No such file or directory
make: *** [all] Error 1

###########################################################

# Makefile 内容 (前缀 -)
all:
    -echo "没有前缀"
    -cat this_file_not_exist
    -echo "错误之后的命令"       <-- 这条命令会被执行

# bash中执行 make
$ make
echo "没有前缀"             <-- 命令本身显示出来
没有前缀                    <-- 命令执行结果显示出来
cat this_file_not_exist
cat: this_file_not_exist: No such file or directory
make: [all] Error 1 (ignored)
echo "错误之后的命令"       <-- 出错之后的命令也会显示
错误之后的命令              <-- 出错之后的命令也会执行
```

#### 2.9为命令
伪目标并不是一个"目标(target)", 不像真正的目标那样会生成一个目标文件.
```shell
.PHONY: clean   <-- 这句没有也行, 但是最好加上
clean:
    -rm -f *.o
```

#### 2.10引用其他Makefile
include \<filename\>  (filename 可以包含通配符和路径)

#### 2.11查看C文件依赖关系
```shell
$ cd virt/kvm/
$ gcc -MM kvm_main.c 
kvm_main.o: kvm_main.c iodev.h coalesced_mmio.h async_pf.h   <-- 这句就可以加到 Makefile 中作为编译 kvm_main.o 的依赖关系
```

#### 2.12 退出码
- 0 :: 表示成功执行
- 1 :: 表示make命令出现了错误
- 2 :: 使用了 "-q" 选项, 并且make使得一些目标不需要更新

#### 2.13指定特定Makefile文件
```shell
# Makefile文件名改为 MyMake, 内容
target1:
    @echo "target [1]  begin"
    @echo "target [1]  end"

target2:
    @echo "target [2]  begin"
    @echo "target [2]  end"

# bash 中执行 make
$ ls
Makefile
$ mv Makefile MyMake
$ ls
MyMake
$ make                     <-- 找不到默认的 Makefile
make: *** No targets specified and no makefile found.  Stop.
$ make -f MyMake           <-- 指定特定的Makefile
target [1]  begin
target [1]  end
$ make -f MyMake target2   <-- 指定特定的目标(target)
target [2]  begin
target [2]  end
```

#### 2.14make参数
|   |   |
|---|---|
|**参数**|**含义**|
|--debug[=<options>]|输出make的调试信息, options 可以是 a, b, v|
|-j --jobs|同时运行的命令的个数, 也就是多线程执行 Makefile|
|-r --no-builtin-rules|禁止使用任何隐含规则|
|-R --no-builtin-variabes|禁止使用任何作用于变量上的隐含规则|
|-B --always-make|假设所有目标都有更新, 即强制重编译|

#### 2.15隐含规则
编译C时，\<n\>.o 的目标会自动推导为 \<n\>.c

```shell
# Makefile 中
main : main.o
    gcc -o main main.o

#会自动变为:
main : main.o
    gcc -o main main.o

main.o: main.c    <-- main.o 这个目标是隐含生成的
    gcc -c main.c
```

#### 2.16隐含规则中的命令变量、命令参数变量
|   |   |
|---|---|
|**变量名**|**含义**|
|RM|rm -f|
|AR|ar|
|CC|cc|
|CXX|g++|

| **变量名**  | **含义**      |
| -------- | ----------- |
| ARFLAGS  | AR命令的参数     |
| CFLAGS   | C语言编译器的参数   |
| CXXFLAGS | C++语言编译器的参数 |

#### 2.17自动变量

| **自动变量** | **含义**                       |
| -------- | ---------------------------- |
| $@       | 目标集合                         |
| $%       | 当目标是函数库文件时, 表示其中的目标文件名       |
| $<       | 第一个依赖目标. 如果依赖目标是多个, 逐个表示依赖目标 |
| $?       | 比目标新的依赖目标的集合                 |
| $^       | 所有依赖目标的集合, 会去除重复的依赖目标        |
| $+       | 所有依赖目标的集合, 不会去除重复的依赖目标       |
| $*       | 这个是GNU make特有的, 其它的make不一定支持 |

## 3.高级语法
#### 3.1嵌套Makefile
- export variable = value
- export variable := value
- export variable += value

```shell
# Makefile 内容
export VALUE1 := export.c    <-- 用了 export, 此变量能够传递到 ./other/Makefile 中
VALUE2 := no-export.c        <-- 此变量不能传递到 ./other/Makefile 中

all:
    @echo "主 Makefile begin"
    @cd ./other && make
    @echo "主 Makefile end"


# ./other/Makefile 内容
other-all:
    @echo "other makefile begin"
    @echo "VALUE1: " $(VALUE1)
    @echo "VALUE2: " $(VALUE2)
    @echo "other makefile end"

# bash中执行 make
$ make
主 Makefile begin
make[1]: Entering directory `/path/to/test/makefile/other'
other makefile begin
VALUE1:  export.c        <-- VALUE1 传递成功
VALUE2:                  <-- VALUE2 传递失败
other makefile end
make[1]: Leaving directory `/path/to/test/makefile/other'
主 Makefile end
```

#### 3.2定义命令包
命令包有点像是个函数, 将连续的相同的命令合成一条, 减少 Makefile 中的代码量, 便于以后维护.

```shell
define <command-name>
command
...
endef
```

```shell
# Makefile 内容
define run-hello-makefile
@echo -n "Hello"
@echo " Makefile!"
@echo "这里可以执行多条 Shell 命令!"
endef

all:
    $(run-hello-makefile)


# bash 中运行make
$ make
Hello Makefile!
这里可以执行多条 Shell 命令!
```

#### 3.3条件判断
条件判断的关键字主要有 *ifeq ifneq ifdef ifndef*

```shell
<conditional-directive>
<text-if-true>
endif

# 或者
<conditional-directive>
<text-if-true>
else
<text-if-false>
endif
```

```shell
# Makefile 内容
all:
ifeq ("aa", "bb")
    @echo "equal"
else
    @echo "not equal"
endif

# bash 中执行 make
$ make
not equal
```

#### 3.4Makefile中的函数
Makefile 中自带了一些函数, 利用这些函数可以简化 Makefile 的编写.

函数调用语法如下:
```shell
$(<function> <arguments>)
#或者
${<function> <arguments>}
```

- \<function\> 是函数名
- \<arguments\> 是函数参数

##### 3.4.1字符串函数
###### 字符串替换函数
字符串替换函数: $(subst \<from\>,\<to\>,\<text\>)
功能: 把字符串\<text\> 中的 \<from\> 替换为 \<to\>
返回: 替换过的字符串
```shell
# Makefile 内容
all:
    @echo $(subst t,e,maktfilt)  <-- 将t替换为e

# bash 中执行 make
$ make
makefile
```

###### 模式字符串替换函数
模式字符串替换函数: $(patsubst \<pattern\>,\<replacement\>,\<text\>)
功能: 查找\<text\>中的单词(单词以"空格", "tab", "换行"来分割) 是否符合 \<pattern\>, 符合的话, 用 \<replacement\> 替代.
返回: 替换过的字符串

```shell
# Makefile 内容
all:
    @echo $(patsubst %.c,%.o,programA.c programB.c)
# bash 中执行 make
$ make
programA.o programB.o
```

###### 去空格函数
去空格函数: $(strip \<string\>)
功能: 去掉 \<string\> 字符串中开头和结尾的空字符
返回: 被去掉空格的字符串值

```shell
# Makefile 内容
VAL := "       aa  bb  cc "

all:
    @echo "去除空格前: " $(VAL)
    @echo "去除空格后: " $(strip $(VAL))

# bash 中执行 make
$ make
去除空格前:         aa  bb  cc 
去除空格后:   aa bb cc
```

###### 查找字符串函数
查找字符串函数: $(findstring \<find\>,\<in\>)
功能: 在字符串 \<in\> 中查找 \<find\> 字符串
返回: 如果找到, 返回 \<find\> 字符串,  否则返回空字符串

```shell
# Makefile 内容
VAL := "       aa  bb  cc "

all:
    @echo $(findstring aa,$(VAL))
    @echo $(findstring ab,$(VAL))

# bash 中执行 make
$ make
aa
```

###### 过滤函数
过滤函数: $(filter \<pattern...\>,\<text\>)
功能: 以 \<pattern\> 模式过滤字符串 \<text\>, *保留* 符合模式 \<pattern\> 的单词, 可以有多个模式
返回: 符合模式 \<pattern\> 的字符串

```shell
# Makefile 内容
all:
    @echo $(filter %.o %.a,program.c program.o program.a)

# bash 中执行 make
$ make
program.o program.a
```

###### 反过滤函数
反过滤函数: $(filter-out \<pattern...\>,\<text\>)
功能: 以 \<pattern\> 模式过滤字符串 \<text\>, *去除* 符合模式 \<pattern\> 的单词, 可以有多个模式
返回: 不符合模式 \<pattern\> 的字符串

```shell
# Makefile 内容
all:
    @echo $(filter-out %.o %.a,program.c program.o program.a)

# bash 中执行 make
$ make
program.c
```

###### 排序函数
排序函数: $(sort \<list\>)
功能: 给字符串 \<list\> 中的单词排序 (升序)
返回: 排序后的字符串

```shell
# Makefile 内容
all:
    @echo $(sort bac abc acb cab)

# bash 中执行 make
$ make
abc acb bac cab
```

###### 取单词函数
取单词函数: $(word \<n\>,\<text\>)
功能: 取字符串 \<text\> 中的 第\<n\>个单词 (n从1开始)
返回: \<text\> 中的第\<n\>个单词, 如果\<n\> 比 \<text\> 中单词个数要大, 则返回空字符串

```shell
# Makefile 内容
all:
    @echo $(word 1,aa bb cc dd)
    @echo $(word 5,aa bb cc dd)
    @echo $(word 4,aa bb cc dd)

# bash 中执行 make
$ make
aa

dd
```

###### 取单词串函数
取单词串函数: $(wordlist \<s\>,\<e\>,\<text\>)
功能: 从字符串\<text\>中取从\<s\>开始到\<e\>的单词串. \<s\>和\<e\>是一个数字.
返回: 从\<s\>到\<e\>的字符串

```shell
# Makefile 内容
all:
    @echo $(wordlist 1,3,aa bb cc dd)
    @echo $(word 5,6,aa bb cc dd)
    @echo $(word 2,5,aa bb cc dd)


# bash 中执行 make
$ make
aa bb cc

bb
```

###### 单词个数统计函数
单词个数统计函数: $(words \<text\>)
功能: 统计字符串 \<text\> 中单词的个数
返回: 单词个数

```shell
# Makefile 内容

all:
    @echo $(words aa bb cc dd)
    @echo $(words aabbccdd)
    @echo $(words )

# bash 中执行 make
$ make
4
1
0
```

###### 首单词函数
首单词函数: $(firstword \<text\>)
功能: 取字符串 \<text\> 中的第一个单词
返回: 字符串 \<text\> 中的第一个单词

```shell
# Makefile 内容
all:
    @echo $(firstword aa bb cc dd)
    @echo $(firstword aabbccdd)
    @echo $(firstword )

# bash 中执行 make
$ make
aa
aabbccdd
```

##### 3.4.2文件名函数
###### 取目录函数
取目录函数: $(dir \<names...\>)
功能: 从文件名序列 \<names\> 中取出目录部分
返回: 文件名序列 \<names\> 中的目录部分

```shell
# Makefile 内容
all:
    @echo $(dir /home/a.c ./bb.c ../c.c d.c)


# bash 中执行 make
$ make
/home/ ./ ../ ./
```

###### 取文件函数
取文件函数: $(notdir \<names...\>)
功能: 从文件名序列 \<names\> 中取出非目录部分
返回: 文件名序列 \<names\> 中的非目录部分

```shell
# Makefile 内容
all:
    @echo $(notdir /home/a.c ./bb.c ../c.c d.c)

# bash 中执行 make
$ make
a.c bb.c c.c d.c
```

###### 取后缀函数
取后缀函数: $(suffix \<names...\>)
功能: 从文件名序列 \<names\> 中取出各个文件名的后缀
返回: 文件名序列 \<names\> 中各个文件名的后缀, 没有后缀则返回空字符串

```shell
# Makefile 内容
all:
    @echo $(suffix /home/a.c ./b.o ../c.a d)

# bash 中执行 make
$ make
.c .o .a
```

###### 取前缀函数
取前缀函数: $(basename \<names...\>)
功能: 从文件名序列 \<names\> 中取出各个文件名的前缀
返回: 文件名序列 \<names\> 中各个文件名的前缀, 没有前缀则返回空字符串

```shell
# Makefile 内容
all:
    @echo $(basename /home/a.c ./b.o ../c.a /home/.d .e)


# bash 中执行 make
$ make
/home/a ./b ../c /home/
```

###### 加后缀函数
加后缀函数: $(addsuffix \<suffix\>,\<names...\>)
功能: 把后缀 \<suffix\> 加到 \<names\> 中的每个单词后面
返回: 加过后缀的文件名序列

```shell
# Makefile 内容
all:
    @echo $(addsuffix .c,/home/a b ./c.o ../d.c)


# bash 中执行 make
$ make
/home/a.c b.c ./c.o.c ../d.c.c
```

###### 加前缀函数
加前缀函数: $(addprefix \<prefix\>,\<names...\>)
功能: 把前缀 \<prefix\> 加到 \<names\> 中的每个单词前面
返回: 加过前缀的文件名序列

```shell
# Makefile 内容
all:
    @echo $(addprefix test_,/home/a.c b.c ./d.c)

# bash 中执行 make
$ make
test_/home/a.c test_b.c test_./d.c
```

###### 连接函数
连接函数: $(join \<list1\>,\<list2\>)
功能: \<list2\> 中对应的单词加到 \<list1\> 后面
返回: 连接后的字符串

```shell
# Makefile 内容
all:
    @echo $(join a b c d,1 2 3 4)
    @echo $(join a b c d,1 2 3 4 5)
    @echo $(join a b c d e,1 2 3 4)

# bash 中执行 make
$ make
a1 b2 c3 d4
a1 b2 c3 d4 5
a1 b2 c3 d4 e
```

##### 3.4.3foreach
$(foreach \<var\>,\<list\>,\<text\>)

```shell
# Makefile 内容
targets := a b c d
objects := $(foreach i,$(targets),$(i).o)

all:
    @echo $(targets)
    @echo $(objects)

# bash 中执行 make
$ make
a b c d
a.o b.o c.o d.o
```

##### 3.4.4if
这里的if是个函数, 和前面的条件判断不一样, 前面的条件判断属于Makefile的关键字
**语法:**
$(if \<condition\>,\<then-part\>)
$(if \<condition\>,\<then-part\>,\<else-part\>)

```shell
# Makefile 内容
val := a
objects := $(if $(val),$(val).o,nothing)
no-objects := $(if $(no-val),$(val).o,nothing)

all:
    @echo $(objects)
    @echo $(no-objects)

# bash 中执行 make
$ make
a.o
nothing
```

##### 3.4.5call创建新参数化函数
**语法:**
$(call \<expression\>,\<parm1\>,\<parm2\>,\<parm3\>...)

```shell
# Makefile 内容
log = "====debug====" $(1) "====end===="

all:
    @echo $(call log,"正在 Make")

# bash 中执行 make
$ make
====debug==== 正在 Make ====end====
```

##### 3.4.5origin 判断变量来源
**语法:**
$(origin \<variable\>)

|    **类型**    |                **含义**                 |
| :----------: | :-----------------------------------: |
|  undefined   |           <variable> 没有定义过            |
|   default    |     <variable> 是个默认的定义, 比如 CC 变量      |
| environment  | <variable> 是个环境变量, 并且 make时没有使用 -e 参数 |
|     file     |        <variable> 定义在Makefile中        |
| command line |          <variable> 定义在命令行中           |
|   override   |      <variable> 被 override 重新定义过      |
|  automatic   |           <variable> 是自动化变量           |

```shell
# Makefile 内容
val-in-file := test-file
override val-override := test-override

all:
    @echo $(origin not-define)    # not-define 没有定义
    @echo $(origin CC)            # CC 是Makefile默认定义的变量
    @echo $(origin PATH)         # PATH 是 bash 环境变量
    @echo $(origin val-in-file)    # 此Makefile中定义的变量
    @echo $(origin val-in-cmd)    # 这个变量会加在 make 的参数中
    @echo $(origin val-override) # 此Makefile中定义的override变量
    @echo $(origin @)             # 自动变量, 具体前面的介绍

# bash 中执行 make
$ make val-in-cmd=val-cmd
undefined
default
environment
file
command line
override
automatic
```

##### 3.4.6 shell
**语法:**
$(shell \<shell command\>)
它的作用就是执行一个shell命令, 并将shell命令的结果作为函数的返回.
作用和 \`\<shell command\>\` 一样, **\`** 是反引号

##### 3.4.7make控制函数
###### 产生一个致命错误
产生一个致命错误: $(error \<text ...\>)
功能: 输出错误信息, 停止Makefile的运行

```shell
# Makefile 内容
all:
    $(error there is an error!)
    @echo "这里不会执行!"

# bash 中执行 make
$ make
Makefile:2: *** there is an error!.  Stop.
```

###### 输出警告
输出警告: $(warning \<text ...\>)
功能: 输出警告信息, Makefile继续运行

```shell
# Makefile 内容
all:
    $(warning there is an warning!)
    @echo "这里会执行!"

# bash 中执行 make
$ make
Makefile:2: there is an warning!
这里会执行!
```

#### 3.5Makefile中一些GNU约定俗成的伪目标

|   **伪目标**    |              **含义**              |
| :----------: | :------------------------------: |
|     all      |      所有目标的目标，其功能一般是编译所有的目标       |
|    clean     |          删除所有被make创建的文件          |
|   install    | 安装已编译好的程序，其实就是把目标可执行文件拷贝到指定的目录中去 |
|    print     |            列出改变过的源文件             |
|     tar      |       把源程序打包备份. 也就是一个tar文件       |
|     dist     | 创建一个压缩文件, 一般是把tar文件压成Z文件. 或是gz文件 |
|     TAGS     |       更新所有的目标, 以备完整地重编译使用        |
| check 或 test |        一般用来测试makefile的流程         |