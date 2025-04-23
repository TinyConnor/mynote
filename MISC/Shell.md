### shell启动方式
1. 当程序执行：`./xxx.sh`*创建子进程执行*
2. 指定解释器运行：`/bin/bash xxx.sh`*创建子进程执行*
3. source 和 . ：`source xxx.sh`和`. xxx.sh`*当前进程执行*

---
### 定义变量
- variable=value
- variable='value'
- variable="value" *当两个变量拼接时必须用双引号*
---
### 使用变量
- $variable
- ${variable}
---
### 将命令结果赋值变量
- variable=`command`
- variable=$(command)
---
### 删除变量
- unset
---
### 特殊变量
|   变量    |                         含义                          |
| :-----: | :-------------------------------------------------: |
|   $0    |                      当前脚本的文件名。                      |
| $n（n≥1） | 传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是 $1，第二个参数是 $2。 |
|   $#    |                   传递给脚本或函数的参数个数。                    |
|   $*    |                   传递给脚本或函数的所有参数。                    |
|   $@    |      传递给脚本或函数的所有参数。当被双引号`" "`包含时，$@ 与 $* 稍有不同.      |
|   $?    |                 上个命令的退出状态或者获取函数返回值。                 |
|   $$    |     当前 Shell 进程 ID。对于 Shell 脚本，就是这些脚本所在的进程 ID。      |

---
### 字符串拼接
并排放

---
### 读取从键盘上输入的数据
`read <变量>`
`read -p "提示信息" <变量>`

---
### 退出当前进程
exit

---
### 对整数进行数学运算
`((<运算式>))`

---
### 逻辑与或
`command1 && command2`
`command1 || command2`

---
### 检测某个条件是否成立
* test \<expression\>
* \[expression\]

|     选 项     |         作 用         |
| :---------: | :-----------------: |
|     -eq     |      判断数值是否相等       |
|     -ne     |      判断数值是否不相等      |
|     -gt     |      判断数值是否大于       |
|     -lt     |      判断数值是否小于       |
|     -ge     |     判断数值是否大于等于      |
|     -le     |     判断数值是否小于到等于     |
|   -z str    |   判断字符串 str 是否为空    |
|   -n str    |    判断字符串str是否为非空    |
|    =和==     |    判断字符串str是否相等     |
| -d filename | 判断文件是否存在，并且是否为目录文件。 |
| -f filename | 判断文件是否存在，井且是否为普通文件。 |

---
### 管道
command1 | command2

---
### if语句
```shell
if condition 
then ​ 
	statement(s) 
fi
```
---
### if else语句
```shell
if condition 
then 
	statement1 
else 
	statement2 
fi
```
---
### if elif else语句
```shell
if condition1 
then 
	statement1 
elif condition2 
then ​ 
	statement2

…… 
else 
	statementn 
fi
```
---
### case in语句
```shell
case expression in ​ 
	pattern1) ​ 
		statement1 ​ 
		;; ​ 
	pattern2) ​ 
		statement2 ​ 
		;; ​ 
	pattern3) ​ 
		statement3 ​ 
		;; ​ 
	…… ​ 
	*) ​ 
		statementn 
esac
```
---
### for in 语句
```shell
for variable in value_list 
do
	statements 
done
```

value_list

- 直接给出具体的值
- 给出一个取值范围
- 使用命令的执行结果
- 使用 Shell 通配符
- 使用特殊变量
---
### while 循环
```shell
while condition 
do ​ 
	statements 
done
```
---
### 函数
```shell
function name() { ​ 
	statements ​
	[return value] 
}
```
---
