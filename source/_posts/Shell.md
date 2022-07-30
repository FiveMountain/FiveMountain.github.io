---
title: Shell
date: 2021-09-19 10:43:37
categories:
- Learning Notes
---



# 认识 Shell

**Shell概念**

Shell是一个用C语言编写的程序，是用户使用Linux的桥梁。Shell既是一种命令语言，也是一种程序设计语言。
Shell是指一种应用程序，这个应用程序提供一个界面，用户通过这个界面访问操作系统内核的服务。

**Shell脚本**

Shell Script, 是一种为 Shell 编写的脚本程序。Shell 和 Shell Script 是两个不同的概念。

**Shell环境**

Linux的Shell种类众多，常见的有：

- Bourne Shell (/usr/bin/sh 或 /bin/sh)
- Bourne Again Shell (/bin/bash)
- C Shell (/usr/bin/csh)
- K Shell (/usr/bin/ksh)
- Shell for Root (/sbin/sh)
- ...

**第一个 Shell 脚本**

```shell
#!/bin/bash
echo "Hello World!"
```

`#!`是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行



# Shell 变量

变量命名规则：

- 命名只能使用英文字母，数字和下划线，首个字符不能以数字开头。
- 中间不能有空格，可以使用下划线 `_`。
- 不能使用标点符号。
- 不能使用bash里的关键字（可用help命令查看保留关键字）。



变量类型：

- **局部变量**：在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。
- **环境变量**：所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。
- **shell变量**：由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行。

> 命令

|  -   |     命令     |                    描述                    |
| :--: | :----------: | :----------------------------------------: |
| 使用 |    ${xxx}    | 花括号可选，建议加；已定义变量可被重新定义 |
| 只读 | readonly xxx |              将变量定义为只读              |
| 删除 |  unset xxx   |    删除后不能再次使用。不能删除只读变量    |



## Shell 字符串

字符串可以用单引号，也可以用双引号，也可以不用引号。

- **单引号**：任何字符都会原样输出，变量无效
- **双引号**：可以有变量，可以出现转义字符

```shell
#!/bin/bash
name='wue'
# 使用单引号拼接
str1='my name is ${name}. (use single quotes)'
str2='my name is '${name}'. (use single quotes)'
# 使用双引号拼接
str3="my name is ${name}. (use double quotes)"
str4="my name is "${name}". (use double quotes)"
# 输出
echo ${str1}
echo ${str2}
echo ${str3}
echo ${str4}

# 结果
my name is ${name}. (use single quotes)
my name is wue. (use single quotes)
my name is wue. (use double quotes)
my name is wue. (use double quotes)
```

> 字符串长度、根据下标提取子串、查找字符位置

```shell
#!/bin/bash

name='wue'
str="my name is ${name}."

# 字符串长度
echo ${#name}

# 提取子字符串
echo ${str:3:4}
echo ${str:3}
                                                                                                         
# 查找字符位置(哪个字符先出现就计算哪个)
echo `expr index "${str}" name`
```



## Shell 数组

bash支持一维数组（不支持多维数组），并且没有限定数组的大小。

数组元素的下标由 0 开始编号。获取数组中的元素要利用下标，下标可以是整数或算术表达式。

> 定义数组

```shell
# 在Shell中，用括号表示数组，数组元素用空格分割开
array_name=(val1 val2 val3 ... valn)

# 还可以单独定义数组的各个分量
array[0]=val0
array[1]=val1
array[n]=valn
# ↑可以不使用连续的下标，而且下标的范围没有限制
```

> 读取数组

```shell
# 读取数组元素值的一般形式
val=${array[n]}

# 使用 @/* 可以获取数组中的所有元素
echo ${array[@]}
echo ${array[*]}

# 获取数组长度
echo ${#array[@]}
echo ${#array[*]}
```



## Shell 注释

```
# shell中注释以 # 开头

:<<EOF
多行注释可以像这样设置
'EOF'也可以用其他符号代替
EOF
```



# Shell 参数传递

我们可以在执行Shell脚本时，向脚本传递参数。

脚本内获取参数的格式为`$n`。n为正整数，1为第一个参数，以此类推...

```shell
#!/bin/bash
# author: wue

echo "执行的文件名：$0"
echo "第一个参数为：$1"
echo "第二个参数为：$2"
echo "第三个参数为：$3"


# 执行结果
$ ./wue.sh 1 2 3
执行的文件名：./wue.sh
第一个参数为：1
第二个参数为：2
第三个参数为：3
```

> 一些用来处理参数的特殊字符

| 参数处理 |                        说明                         |
| :------: | :-------------------------------------------------: |
|    $#    |                传递到脚本的参数个数                 |
|    $*    |       以一个单字符串显示所有向脚本传递的参数        |
|    $@    |  与$*相同，但使用时加引号，并在引号中返回每个参数   |
|    $$    |              脚本运行的当前进程的ID号               |
|    $!    |            后台运行的最后一个进程的ID号             |
|    $-    |     显示Shell使用的当前选项，与set命令功能相同      |
|    $?    | 显示最后命令的退出状态。0表示无误，其他值表示有错误 |

```shell
#!/bin/bash
# author: wue

echo "第一个参数：$1"
echo "第二个参数：$2"
echo "参数的个数：$#"
echo "\$*的结果：$*"
echo "\$@的结果：$@"


# 执行的结果
$ ./wue.sh hello world
第一个参数：hello
第二个参数：world
参数的个数：2
$*的结果：hello world
$@的结果：hello world
```

**\$* 与 \$@ 比较**

"$*" 等价于 "1 2 3"（传递了一个参数）

"$@" 等价于 "1" "2" "3"（传递了三个参数）

```shell
#!/bin/bash
# author: wue

echo "--- \$* 演示 ---"
for i in "$*"; do
    echo $i
done

echo "--- \$@ 演示 ---"
for i in "$@"; do
    echo $i
done


# 执行结果
$ ./wue.sh hello world
--- $* 演示 ---
hello world
--- $@ 演示 ---
hello
world
```

> 其它

在为Shell脚本传递的参数中如果包含空格，应该使用单引号或者双引号将参数括起来。

在有参数时，可以使用对参数进行校验的方式处理以减少错误的发生：

```shell
if [ -n $1 ]; then
	echo "包含第一个参数"
else
	echo "不包含第一个参数"
fi
```

注意：中括号`[]`与其中间的代码应有空格隔开



Shell 里的中括号（包括单中括号与双中括号）可用于一些条件的测试：

- 算数比较。比如一个变量是否为零。`[ $var -eq 0 ]`
- 文件属性测试。比如一个文件是否存在：`[ -e $var ]`，是否是目录：`[ -d $var ]`
- 字符串比较。比如链各个字符串是否相同：`[[ $var1 = $var2 ]]`



# Shell 运算符

原生 Bash 不支持简单的数学运算，但是可以通过其他命令来实现，例如 `awk` 和 `expr`.

`expr` 是一个表达式计算工具，使用它能完成表达式的求值操作。

```shell
#!/bin/bash
# author: wue

val=`expr 2 + 2`
echo "result is: ${val}"


# 执行结果
$ ./wue.sh 
result is: 4
```

注意：

- 表达式和运算符之间要有空格。
- 完整的表达式要被反引号 ` `` ` 包含。



## 算数运算符

常用的算术运算符包括：`+  -  *  /  %  =  ==  !=` .

注意：条件表达式要放在方括号之间，并且要有空格，例如`[a == b]`是错误的，必须写成`[ a == b ]`

```shell
#!/bin/bash
# author: wue

a=20
b=10

val=`expr $a + $b`
echo "a + b = $val"

val=`expr $a - $b`
echo "a - b = $val"

val=`expr $a \* $b`
echo "a * b = $val"

val=`expr $a / $b`
echo "a / b = $val"

val=`expr $a % $b`
echo "a % b = $val"

if [ $a == $b ]
then
    echo "a == b"
fi
if [ $a != $b ]
then
    echo "a != b"
fi


# 执行结果
$ ./wue.sh 
a + b = 30
a - b = 10
a * b = 200
a / b = 2
a % b = 0
a != b
```

注意：

- 乘号 `*` 前必须加反斜杠 `\` 转义后才能实现乘法运算。
- 在 MAC 中 Shell 的 expr 语法是：`$((表达式))`, 此处表达式中的 `*` 不需要转义。



## 关系运算符

关系运算符只支持数字，不支持字符串，除非字符串的值是数字。

常用的关系运算符有：`-eq  -ne  -gt  -lt  -ge  -le` .

```shell
#!/bin/bash
# author: wue

a=20
b=10

if [ $a -eq $b ]
then
    echo "$a -eq $b : a 等于 b"
else
    echo "$a -eq $b: a 不等于 b"
fi

if [ $a -ne $b ]
then
    echo "$a -ne $b: a 不等于 b"
else
    echo "$a -ne $b : a 等于 b"
fi

if [ $a -gt $b ]
then
    echo "$a -gt $b: a 大于 b"
else
    echo "$a -gt $b: a 不大于 b"
fi

if [ $a -lt $b ]
then
    echo "$a -lt $b: a 小于 b"
else
    echo "$a -lt $b: a 不小于 b"
fi

if [ $a -ge $b ]
then
    echo "$a -ge $b: a 大于或等于 b"
else
    echo "$a -ge $b: a 小于 b"
fi

if [ $a -le $b ]
then
    echo "$a -le $b: a 小于或等于 b"
else
    echo "$a -le $b: a 大于 b"
fi


# 执行结果
$ ./wue.sh 
20 -eq 10: a 不等于 b
20 -ne 10: a 不等于 b
20 -gt 10: a 大于 b
20 -lt 10: a 不小于 b
20 -ge 10: a 大于或等于 b
20 -le 10: a 大于 b
```



## 布尔运算符

常用布尔运算符有：`!  -o  -a` .

```shell
#!/bin/bash
# author: wue

a=20
b=10

if [ $a != $b ]
then
    echo "$a != $b : a 不等于 b"
else
    echo "$a == $b: a 等于 b"
fi

if [ $a -lt 100 -a $b -gt 15 ]
then
    echo "$a 小于 100 且 $b 大于 15 : 返回 true"
else
    echo "$a 小于 100 且 $b 大于 15 : 返回 false"
fi

if [ $a -lt 100 -o $b -gt 100 ]
then
    echo "$a 小于 100 或 $b 大于 100 : 返回 true"
else
    echo "$a 小于 100 或 $b 大于 100 : 返回 false"
fi

if [ $a -lt 5 -o $b -gt 100 ]
then
    echo "$a 小于 5 或 $b 大于 100 : 返回 true"
else
    echo "$a 小于 5 或 $b 大于 100 : 返回 false"
fi


# 执行结果
$ ./wue.sh 
20 != 10 : a 不等于 b
20 小于 100 且 10 大于 15 : 返回 false
20 小于 100 或 10 大于 100 : 返回 true
20 小于 5 或 10 大于 100 : 返回 false
```



## 逻辑运算符

常用逻辑运算符有：`&&  ||` .

```shell
#!/bin/bash
# author: wue

a=20
b=10

if [[ $a -lt 100 && $b -gt 100 ]]
then
    echo "返回 true"
else
    echo "返回 false"
fi

if [[ $a -lt 100 || $b -gt 100 ]]
then
    echo "返回 true"
else
    echo "返回 false"
fi


# 执行结果
$ ./wue.sh 
返回 false
返回 true
```



## 字符串运算符

常用字符串运算符有：`=  !=  -z  -n  $` .

```shell
#!/bin/bash
# author: wue

a="abc"
b="def"

if [ $a = $b ]
then
    echo "$a = $b : a 等于 b"
else
    echo "$a = $b: a 不等于 b"
fi

if [ $a != $b ]
then
    echo "$a != $b : a 不等于 b"
else
    echo "$a != $b: a 等于 b"
fi

if [ -z $a ]
then
    echo "-z $a : 字符串长度为 0"
else
    echo "-z $a : 字符串长度不为 0"
fi

if [ -n "$a" ]
then
    echo "-n $a : 字符串长度不为 0"
else
    echo "-n $a : 字符串长度为 0"
fi

if [ $a ]
then
    echo "$a : 字符串不为空"
else
    echo "$a : 字符串为空"
fi


# 执行结果
$ ./wue.sh 
abc = def: a 不等于 b
abc != def : a 不等于 b
-z abc : 字符串长度不为 0
-n abc : 字符串长度不为 0
abc : 字符串不为空
```



## 文件测试运算符

常用文件测试运算符有：

| 操作符  |                         说明                         |
| :-----: | :--------------------------------------------------: |
| -b file |               检测文件是否是块设备文件               |
| -c file |              检测文件是否是字符设备文件              |
| -d file |                  检测文件是否是目录                  |
| -f file | 检测文件是否是普通文件（既不是目录，也不是设备文件） |
| -g file |               检测文件是否设置了SGID位               |
| -k file |        检测文件是否设置了黏着位（Sticky Bit）        |
| -p file |                检测文件是否是有名管道                |
| -u file |               检测文件是否设置了SUID位               |
| -r file |                   检测文件是否可读                   |
| -w file |                   检测文件是否可写                   |
| -x file |                  检测文件是否可执行                  |
| -s file |       检测文件是否不为空（文件大小是否大于0）        |
| -e file |             检测文件（包括目录）是否存在             |

```shell
#!/bin/bash
# author: wue

file="/root/wue.sh"
if [ -r $file ]
then
    echo "文件可读"
else
    echo "文件不可读"
fi

if [ -w $file ]
then
    echo "文件可写"
else
    echo "文件不可写"
fi

if [ -x $file ]
then
    echo "文件可执行"
else
    echo "文件不可执行"
fi

if [ -f $file ]
then
    echo "文件为普通文件"
else
    echo "文件为特殊文件"
fi

if [ -d $file ]
then
    echo "文件是个目录"
else
    echo "文件不是个目录"
fi

if [ -s $file ]
then
    echo "文件不为空"
else
    echo "文件为空"
fi

if [ -e $file ]
then
    echo "文件存在"
else
    echo "文件不存在"
fi


# 执行结果
$ ./wue.sh 
文件可读
文件可写
文件可执行
文件为普通文件
文件不是个目录
文件不为空
文件存在
```



# Shell echo命令

Shell 的 echo 指令用于字符串的输出。

```shell
#!/bin/bash
# author: wue

# 显示普通字符串
echo "Test echo."

# 双引号可以省略
echo Test echo.

# 显示转义字符
echo \"Test echo.\"

# 显示变量
# read 命令从标准输入中读取一行，并把输入行的每个字段的值指定给 Shell 变量
read name
echo "My name is ${name}."

# 显示换行
# -e 开启转义  \n: 换行
echo -e "OK!\n"
echo "It's a test."

# 显示不换行
# -e 开启转义  \c: 不换行
echo -e "OK! \c"
echo "It's a test."

# 显示结果定向至文件
echo "It's a test." > filename
```



# Shell printf命令

```shell
printf format-string [arguments...]
```

```shell
#!/bin/bash
# author: wue

printf "%-10s %-8s %-4s\n" 姓名 性别 体重
printf "%-10s %-8s %-4.2f\n" 郭靖 男 56.1234
printf "%-10s %-8s %-4.2f\n" 杨过 男 55.1234
printf "%-10s %-8s %-4.2f\n" 郭芙 女 46.1234


# 执行结果
$ ./wue.sh 
姓名     性别   体重
郭靖     男      56.12
杨过     男      55.12
郭芙     女      46.12
```

```shell
#!/bin/bash
# author: wue

# format-string 为双引号
printf "%d %s\n" 1 "abc"

# 单引号与双引号效果一样
printf '%d %s\n' 1 "abc"

# 没有引号也可以输出
printf %s abcdef

# 格式只指定了一个参数，但多出的参数仍然会按照该格式输出，format-string 被重用
printf %s abc def
printf "%s\n" abc def
printf "%s %s %s\n" a b c d e f

# 如果没有 arguments，%s 用 NULL 代替，%d 用 0 代替
printf "%s and %d\n"


# 执行结果
$ ./wue.sh 
1 abc
1 abc
abcdefabcdefabc
def
a b c
d e f
 and 0
```

## printf 的转义序列

| 序列  |                             说明                             |
| :---: | :----------------------------------------------------------: |
|  \a   |                警告字符，通常为ASCII的BEL字符                |
|  \b   |                             后退                             |
|  \c   | 抑制（不显示）输出结果中任何结尾的换行字符（只在%b格式指示符控制下的参数字符串中有效）<br/>而且任何留在参数里的字符、任何接下来的参数以及任何留在格式字符串中的字符，都被忽略 |
|  \f   |                       换页（formfeed）                       |
|  \n   |                             换行                             |
|  \r   |                   回车（Carriage return）                    |
|  \t   |                          水平制表符                          |
|  \v   |                          垂直制表符                          |
|  \\\  |                    一个字面上的反斜杠字符                    |
| \ddd  |       表示1到3位数八进制值的字符。仅在格式字符串中有效       |
| \0ddd |                   表示1到3位的八进制值字符                   |



# Shell test命令

test 命令用于检查某个条件是否成立，它可以进行数值、字符和文件三个方面的测试。



## 数值测试

常用：`-eq  -ne  -gt  -ge  -lt  -le` .

```shell
#!/bin/bash
# author: wue

a=100
b=100
if test $[a] -eq $[b]
then
    echo "相等"
else
    echo "不等"
fi


# 执行结果
$ ./wue.sh 
相等
```



## 字符串测试

常用：`=  !=  -z(字符串长度为0)  -n(字符串长度不为0)` .

```shell
#!/bin/bash
# author: wue

a="wue"
b="wuue"

if test $a = $b
then
    echo "相等"
else
    echo "不等"
fi


# 执行结果
$ ./wue.sh 
不等
```



## 文件测试

常用：`-e  -r  -w  -x  -s  -d  -f  -c  -b` .

此外，Shell 还提供了与(-a)、或(-o)、非(!)三个逻辑操作符用于将测试条件连接起来，其优先级为：`!`最高，`-a`次之，`-o`最低。



# Shell 流程控制

Shell 的流程控制不能存在空分支，如果某分支没有语句要执行，则不要写该分支。



## if else

```shell
if condition1
then
	command
	...
elif condition2
then
	command
	...
else
	command
	...
fi

# 一行
if [ $(ps -ef | grep -c "ssh") -gt 1 ]; then echo "true"; fi;
```



## for

```shell
for var in item1 item2 ... itemN
do
	command
	...
done

# 一行
for var in item1 ... itemN; do command1; ...; done;

# 赋值语句和下一步执行也可以放到代码之前的循环语句中
for ((i=0; i<5; i++));do
    echo "这是第 $[i+1] 次调用";
done;
```

in 列表可以包含替换、字符串和文件名。

in 列表是可选的，如果不使用，for 循环使用命令行的位置参数。



## while

while 循环用于不断执行一系列命令，也用于从输入文件中读取数据。

```shell
while condition
do
	command
	...
done
```

```shell
#!/bin/bash
# author: wue

int=1
while (( $int <= 5 ))
do
    echo $int
    let "int++"
done


# 执行结果
$ ./wue.sh 
1
2
3
4
5
```



## until

一般 while 循环优于 until 循环，但在少数情况下，until循环更有效。

```shell
until condition
do
	command
	...
done
```

```shell
#!/bin/bash
# author: wue

a=0
until [ $a -ge 10 ]
do
    echo $a
    a=`expr $a + 1`
done


# 执行结果
$ ./wue.sh 
0
1
2
3
4
5
6
7
8
9
```



## case...esac

```shell
case value in
item1)
	command
	...
	;;
item2)
	command
	...
	;;
*)
	command
	;;
esac
```

```shell
#!/bin/bash
# author: wue

name="wue"
case "${name}" in
    "tom")
        echo "TOM"
        ;;
    "wue")
        echo "WUE"
        ;;
    "jerry")
        echo "JERRY"
        ;;
esac


# 执行结果
$ ./wue.sh 
WUE
```



## 跳出循环

> break

break 命令允许跳出所有循环（终止执行后面的所有循环）。

```shell
#!/bin/bash
# author: wue

while :
do
    echo -n "输入 1 ~ 5 之间的数字: "
    read num
    case $num in
        1 | 2 | 3 | 4 | 5)
            echo "你输入的数字为: ${num}!"
            ;;
        *)
            echo "你输入的数字不是 1 ~ 5 之间的！游戏结束"
            break
            ;;
    esac
done


# 执行结果
$ ./wue.sh 
输入 1 ~ 5 之间的数字: 1
你输入的数字为: 1!
输入 1 ~ 5 之间的数字: 4
你输入的数字为: 4!
输入 1 ~ 5 之间的数字: 6
你输入的数字不是 1 ~ 5 之间的！游戏结束
```

> continue

continue 会终止当前循环，不执行后续语句，直接进入下一次循环。

```shell
#!/bin/bash
# author: wue

while :
do
    echo -n "输入 1 ~ 5 之间的数字: "
    read num
    case $num in
        1 | 2 | 3 | 4 | 5)
            echo "你输入的数字为: ${num}!"
            ;;
        *)
            echo "你输入的数字不是 1 ~ 5 之间的！"
            continue
            echo "游戏结束"
            ;;
    esac
done


# 执行结果
$ ./wue.sh 
输入 1 ~ 5 之间的数字: 3
你输入的数字为: 3!
输入 1 ~ 5 之间的数字: 6
你输入的数字不是 1 ~ 5 之间的！
输入 1 ~ 5 之间的数字: 2
你输入的数字为: 2!
输入 1 ~ 5 之间的数字: ^C
```



# Shell 函数

```shell
[ function ] funcName [()] {
	action;
	[return int;]
}
```

返回值，可以显式加 return 返回，如果不加，将以最后一条命令运行结果作为返回值。return 后跟数值（0 - 255）

函数返回值在调用该函数后通过 `$?` 获得，但 `$?` 仅对其上一条指令负责，一旦函数返回后其返回值没有立即保存入参数，那么其返回值将不能再通过 `$?` 获取。

注意：**所有函数在使用前必须定义**。

> 函数参数

在函数体内部，通过 `$n` 的形式来获取参数的值。

```shell
#!/bin/bash
# author: wue

funcWithParam() {
    echo "第一个参数为 $1"
    echo "第二个参数为 $2"
    echo "第十个参数为 $10 (error)"
    echo "第十个参数为 ${10} (true)"
    echo "参数共有 $# 个"
    echo "作为一个字符串输出所有参数：$*"
}

funcWithParam 1 2 3 4 5 6 7 8 9 45 62 99


# 执行结果
$ ./wue.sh 
第一个参数为 1
第二个参数为 2
第十个参数为 10 (error)
第十个参数为 45 (true)
参数共有 12 个
作为一个字符串输出所有参数：1 2 3 4 5 6 7 8 9 45 62 99
```

注意：当 n >= 10 时，需要用 `${n}` 来获取参数

一些用来处理参数的特殊字符详见上方 "Shell 参数传递"。

> 其它

函数与命令的执行结果可以作为条件语句使用。要注意的是，shell 中 0 代表 true，0 以外的值代表 false。

```shell
#!/bin/bash
# author: wue

echo "Hello World!" | grep -e Hello
echo $?
echo "Hello World!" | grep -e Bye
echo $?

if echo "Hello World!" | grep -e Hello; then
    echo true
else
    echo false
fi

if echo "Hello World!" | grep -e Bye; then
    echo true
else
    echo false
fi

function demoFun1(){
    return 0
}

function demoFun2(){
    return 12
}

if demoFun1; then
    echo true
else
    echo false
fi

if demoFun2; then
    echo true
else
    echo false
fi


# 执行结果
$ ./wue.sh 
Hello World!
0
1
Hello World!
true
false
true
false
```



# Shell 重定向

|      命令       |                       说明                       |
| :-------------: | :----------------------------------------------: |
| command > file  |               将输出重定向到 file                |
| command < file  |               将输入重定向到 file                |
| command >> file |         将输出以追加的方式重定向到 file          |
|    n > file     |       将文件描述符为 n 的文件重定向到 file       |
|    n >> file    | 将文件描述符为 n 的文件以追加的方式重定向到 file |
|     n >& m      |              将输出文件 m 和 n 合并              |
|     n <& m      |              将输入文件 m 和 n 合并              |
|     << tag      | 将开始标记 tag 和结束标记 tag 之间的内容作为输入 |

文件描述符 0 通常示标准输入(STDIN)，1 是标准输出(STDOUT)，2 是标准错误输出(STDERR)。



## 输出重定向

`command > file` 将命令执行结果存入file中，此命令将替换file中已存在的内容。如果需要在文件尾部追加，应使用 `>>` .

> 重定向测试

```shell
[root@wue ~]# ls
wue.sh
[root@wue ~]# who > users
[root@wue ~]# ls
users  wue.sh
[root@wue ~]# cat users 
root     pts/0        2021-09-29 17:12 (xxx.xxx.xxx.xxx)
```

> 覆盖(>)与追加(>>)测试

```shell
[root@wue ~]# cat txt
这是将要被覆盖的内容
[root@wue ~]# echo "这是覆盖的内容" > txt
[root@wue ~]# cat txt 
这是覆盖的内容
[root@wue ~]# echo "这是追加的内容" >> txt
[root@wue ~]# cat txt 
这是覆盖的内容
这是追加的内容
```



## 输入重定向

```shell
# 统计文件行数
[root@wue ~]# wc -l txt
2 txt
[root@wue ~]# wc -l < txt
2
```

> 同时重定向输入和输出

```shell
# 执行command，从文件infile读取内容，然后将输出写入到outfile中
command < infile > outfile
```



## 重定向深入

一般情况下，每个 Unix/Linux 命令运行时都会打开三个文件：

- 标准输入文件(stdin)：文件描述符为0，Unix 程序默认从 stdin 读取数据。
- 标准输出文件(stdout)：文件描述符为1，Unix 程序默认向 stdout 输出数据。
- 标准错误文件(stderr)：文件描述符为2，Unix 程序会向 stderr 流中写入错误信息。

```shell
# stderr 重定向到file
command 2>file

# stderr 追加到file
command 2>>file

# stdout and stderr 合并后重定向到file
 command > file 2>&1
# 或者以追加的方式
 command >> file 2>&1
```

注意：`2>` 这里中间没有空格，`2>` 一体的时候才表示错误输出。



## Here Document

Here Document 是 Shell 中的一种特殊的重定向方式，用来将输入重定向到一个交互式 Shell 脚本或程序。

```shell
command << delimiter
	document
delimiter
```

它的作用是将两个delimiter之间的内容(document)作为输入传递给command。

注意：

- 结尾的 delimiter 一定要顶格写，前面不能有任何字符，后面同样，包括空格和 tab 缩进。
- 开始的 delimiter 前后的空格会被忽略掉。

```shell
#!/bin/bash
# author: wue

wc -l << EOF
    test
    wue
    hello
    world
EOF


# 执行结果
$ ./wue.sh 
4
```



## /dev/null

如果希望执行某个命令，但又不希望在屏幕上显示输出结果，可以将重定向输出到 `/dev/null` .

`/dev/null` 是一个特殊的文件，写入它的内容都会被丢弃；如果尝试从该文件读取内容，那么什么也读取不到。

将命令重定向到它，会起到“禁止输出”的效果。



# Shell 文件包含

```shell
. filename		# 注意点(.)和文件名中间有空格
或者
source filename
```

被包含的文件不需要可执行权限。
