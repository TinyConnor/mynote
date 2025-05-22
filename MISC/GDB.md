>编译时需要加 *-g*选项
>gcc -g test.c -o test

# 简单使用
`gdb <可执行文件>`:启动GDB
`fil(file) <可执行文件>`:在直接使用gdb进入gdb模式下，可以使用这种方式打开文件
`l(list)`:从第一行开始例出原码,显示10行，回车显示接下来10行
`b(break) <line>`:在指定行打断点
`break <func>`:在函数入口打断点
`info break`:查看断点信息
`r(run)`:运行程序
`n(next)`:单步执行程序
`c(continue)`:继续运行程序
`p(print) <variate>`:打印变量值
`bt`:查看函数堆栈
`finish`:退出函数
`q(wuit)`:退出GDB

---
---
# 详细使用
### 1.启用与退出
#### 1.1启用GDB
```shell
gcc -g text.c -o test

#方法1：直接启用可执行文件
gdb test
#方法2：先进入GDB，在打开文件
gdb#进入GDB界面
(gdb) fil test#打开文件
```

>命令：
>*file（简写fil）*  
格式：*file <可执行文件名>*  
作用：载入**当前目录下**的对应名称的可执行文件。

#### 1.2查看源码
```shell
(gdb) l 7#打印第7行附近10行内容
(gdb) l #继续上次打印后的10行
```

>命令：*list（简写为l）*  
格式：*list [行号]* 
作用：打印给定行号周围10行的源代码。若不提供行号，则接续打印上次的源代码。

#### 1.3退出
```shell
(gdb) q
(gdb) quit
```

>命令：*quit（简写为q）*
格式：*quit*（无参数）  
作用：退出gdb。

---
### 2.断点、观察点、捕捉点、信号、线程停止
#### 2.1断点BreakPoint
```shell
(gdb) b main #在main函数入口打断点
(gdb) b 11 #在11行打断点
(gdb) b +offset #在当前所在行后的offset打断点
(gdb) b -offset #在当前所在行前的offset打断点
(gdb) b filename:linenum #在指定文件指定行位置打断点
(gdb) b filename:function #在指定文件指定函数入口处打断点
(gdb) break *address #在指定内存地址处打断点
(gdb) b #在下一条命令处打断点
(gdb) b if i=100 #i=100时打断点
```

>命令：*break（简写为b）*
  格式：*break 函数名|行号*  
  作用：在给定函数名或行数处设置断点。

```shell
#停止条件维护
(gdb) condition bnum expression #修改断点号为bnum的停止条件
(gdb) condition bnum #清楚断点号为bnum的停止条件
(gdb) ignore bnum count #忽略断点号为bnum的停止条件count次
#为停止条件设定运行命令
(gdb)break foo if x>0  
(gdb)commands  
(gdb)printf "x is %d ",x  
(gdb)continue  
(gdb)end
#断点设置在函数foo中，断点条件是x>0，如果程序被断住后，也就是，一旦x的值在foo函数中大于0，GDB会自动打印出x的值，并继续运行程序。
#如果你要清除断点上的命令序列，那么只要简单的执行一下commands命令，并直接在打个end就行了。
```

#### 2.2观察点WatchPoint
**观察点**一般来观察某个表达式（变量也是一种表达式）的值是否有变化了，如果有变化，马上停住程序。
```shell
(gdb)wa expr #expr表达式/变量有变化时，马上停住
(gdb)rwa expr #expr被读时，停住程序
(gdb)awa expr #expr被写时，停住程序
```

>命令：*watch（简写为wa）*
  格式：*watch 表达式|变量*  
  作用：在给定函数名或行数处设置观察点。

#### 2.3捕捉点CatchPoint
**捕捉点**来补捉程序运行时的一些事件。如：载入共享库（动态链接库）或是C++的异常
```shell
(gdb)ca event #事件发生程序停止
#1、throw 一个C++抛出的异常。（throw为关键字）  
#2、catch 一个C++捕捉到的异常。（catch为关键字）  
#3、exec 调用系统调用exec时。（exec为关键字，目前此功能只在HP-UX下有用）  
#4、fork 调用系统调用fork时。（fork为关键字，目前此功能只在HP-UX下有用）  
#5、vfork 调用系统调用vfork时。（vfork为关键字，目前此功能只在HP-UX下有用）  
#6、load 或 load 载入共享库（动态链接库）时。（load为关键字，目前此功能只在HP-UX下有用）  
#7、unload 或 unload 卸载共享库（动态链接库）时。（unload为关键字，目前此功能只在HP-UX下有用）
(gdb)tca event #只设置一次捕捉点，听徐停住以后，该点被删除
```

>命令：*catch（简写为ca）*
  格式：*catch 事件*  
  作用：事件发生时停止程序

#### 2.4信号Signals
信号是一种软中断，是一种处理异步事件的方法。
```shell
(gdb) handle SIGSEGV nostop # 段错误时不暂停程序 
(gdb) handle SIGINT nopass # Ctrl+C不传递给程序，仅GDB处理 
(gdb) handle SIGUSR1 print # 仅打印SIGUSR1信息，不暂停程序
```

>命令 *handle*
>格式 *handle <信号名> <动作>*
>作用：修改信号的处理行为
>动作：
>nostop  当被调试的程序收到信号时，GDB不会停住程序的运行，但会打出消息告诉你收到这种信号。
  stop  当被调试的程序收到信号时，GDB会停住你的程序。This implies the print keyword as well.
  print  当被调试的程序收到信号时，GDB会显示出一条信息。
  noprint  当被调试的程序收到信号时，GDB不会告诉你收到信号的信息。
  pass  / noignore  当被调试的程序收到信号时，GDB不处理信号。这表示，GDB会把这个信号交给被调试程序处理
  nopass  / ignore  当被调试的程序收到信号时，GDB不会让被调试程序来处理这个信号。

#### 2.5线程停止Thread Stops
如果程序是多线程的话，可以定义断点是否在所有的线程上，或是在某个特定的线程。

```shell
(gdb)info threads #查看所有线程
(gdb)thread 2 #切换到线程2
(gdb)b func thread 2 #在函数func入口处设置断点，进线程2触发
(gdb)b func thread 2 if i>100 #在当i>100时函数func入口处设置断点，进线程2触发
```

---
### 3.运行
```shell
(gdb)r #从头开始执行
(gdb)c #当前位置继续执行，到下一个断点结束
(gdb)u 12 #当前位置执行到12行停止
```

> 命令：*run（简写为r）* 
> 格式：*run [参数]*  
> 作用：**从头**运行程序。参数即为从命令行传入的参数。

> 命令：*continue（简写为c）*  
> 格式：*continue [n]*  
> 作用：**从当前位置继续**运行程序，直到遇到下一个断点或程序运行完毕。如果给出了参数 *n*，则跳过所有断点 n 次。

> 命令：*until（简写为u）* 
> 格式：*until 行号*  
> 作用：**从当前位置继续**运行程序，直到指定行号处才停下来。

---
### 4.单步执行
```shell
(gdb)n #单步执行，若有函数调用，将函数当做一条语句执行
(gdb)s #单步执行，若有函数调用，进入函数中单步执行
```

> 命令：*next（简写为n）*  
> 格式：*next [n]*  
> 作用：单步执行。若当前行有函数调用，则**把这个函数作为一个整体执行**（即不进入函数内部）。若给出参数 *n*，则执行 n 步。

> 命令：*step（简写为s）*  
> 格式：*step [n]*  
> 作用：单步执行。若当前行有函数调用，则**进入该函数内部**。若给出参数 `n`，则执行 n 步。

---
### 5.输出变量/函数值
```shell
(gdb) p a #打印一次变量a的值
(gdb)disp a #每一次停下来都打印变量a的值
```

> 命令：*print（简写为p）*  
> 格式：*print 变量名*  
> 作用：打印**一次**变量名/函数调用对应的值。

> 命令：*display（简写为disp）*  
> 格式：*display 变量名*  
> 作用：设置在每一次停下来时（如到断点时，单步执行等）都打印该变量名/函数调用对应的值。

---
### 6.查看某些信息
```shell
(gdb)i lo #查看局部变量信息
(gdb)i b #查看断点信息
(gdb)bt #查看当前函数调用栈的所有信息
(gdb)bt 3 #查看栈顶上3层栈信息
(gdb)bt -3 #查看栈顶下3层栈信息
(gdb)f 3 #查看栈顶开始下第三层 单层的信息，frame 0 为栈顶
(gdb)up 2 #当前栈向上移动2层的信息
(gdb)up #当前栈向上移动1层的信息
(gdb)down 3 #当前栈向下移动3层的信息
(gdb)down #当前栈向下移动1层的信息
```

>命令：*info（简写为i）*  
  格式：*info 类型* 
  作用：打印对应类型的信息。
  类型可以是breakpoints（断点，简写b）、locals（局部变量，简写lo）、display（被设为总是显示的变量，简写disp）等

>命令：*backtrace(简写bt)*
>格式：*bt 参数*
>作用：查看当前函数栈信息 

>命令：*frame (简写f)*
>格式：*f 层数*
>作用：查看栈顶下第n层的栈信息

>命令：*up-silently(简写up)*
>格式：*up 行数*
>作用：当前栈向上移动n层的栈信息

>命令：*down-silently(简写down)*
>格式：*down 行数*
>作用：当前栈向下移动n层的栈信息

---
### 7.删除/禁用/启用/清楚某些东西
```shell
(gdb)dis b 1 #禁用1号断点
(gdb)en b 1 #启用1号断点
(gdb)d breakpoints 1 #删除1号停止点(断点、观察点、捕捉点)
(gdb)cl function #清楚函数上的所有停止点
(gdb)cl #清除所有停止点
```

> 命令：*disable（简写为dis）*  
> 格式：*disable 类型 [编号]*  
> 作用：**临时禁用**某些类型的对应编号的东西，待会儿讲。

> 命令：*delete（简写为d）*  
> 格式：*delete 类型 [编号]*  
> 作用：**删除**某些类型的对应编号的东西。

> 命令：*enable（简写为en）*  
> 格式：*enable 类型 [编号]*  
> 作用：**启用**某些类型的对应编号的东西。

> 命令：*clear（简写为cl）*  
> 格式：*clear 类型 [编号]*  
> 作用：**清除**某些类型的对应编号的东西。

---
### 8.编辑和搜索源代码
```shell
(gdb)edit #编辑整个文件代码
(gdb)edit 3 #编辑指定行数代码
(gdb)edit func #编辑指定函数代码
(gdb)edit file.c:3 #编辑指定文件指定行数代码
(gdb)edit file.c:func #编辑指定文件指定函数
(gdb)fo regexp #从当前位置向后搜索源代码，可以使用正则表达式
(gdb)rev regexp #从当前位置向前搜索源代码，可以使用正则表达式
```

> 命令：*forward-search（简写为fo）*  
> 格式：*fo [正则表达式]*  
> 作用：从当前位置向后搜索指定内容

> 命令：*reverse-search(简写为rev)*  
> 格式：*rev [正则表达式]*  
> 作用：从当前位置向前搜索指定内容

---
### 9.获取帮助
```shell
(gdb)h b #查看断点帮助
(gdb)h info #查看info帮助
```

>命令：*help（简写为h）*
  格式：*help 待查询的命令（待查询的命令可以用简写）*
  作用：显示待查询的命令的帮助。

---
### 10.指定源文件路径
用-g编译过后的执行程序中只是包括了源文件的名字，没有路径名。
```shell
(gdb)dir dirname #添加源文件搜索路径
#添加多个路径时，linux用','隔开 Windows用';'隔开
(gdb)dir #清楚所有自定义源文件搜索路径信息
(gdb)show directories #展示定义源文件的搜索路径
```

>命令：*directory（简写为dir）*
  格式：*directory [路径]*
  作用：添加源文件所有路径

---
### 11.查看内存
```shell
(gdb)info  file.c:func #打印出所指定的源码在运行时的内存地址
(gdb)disassemble func #打印函数运行时的汇编代码
```

---
---