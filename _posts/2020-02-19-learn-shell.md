---
title: shell 学习笔记
date: 2019-10-15 22:35:21
tags: [shell]
categories: [learn, Linux]
snippet: shell
seo:
  date_modified: 2020-02-22 17:02:55 +0800
---


# 基础

## 括号的使用总结

单括号、双括号、单方括号、双方括号

- 单括号 `()` 没作用；
- `$()` 是命令替换；
- `(())` 可以进行算数运算，c 语言风格的 for 命令；
- 单方括号 `[]` 是测试命令，**需要有空格**；
- `$[]` 是数学运算，作用与 `$(())` 一样，后者更常用；
- `$[[]]` 是高级的字符串运算，**需要有空格**；



如果两个命令一起运行，可以把它们放在同一行，之间使用分号隔开：

```bash
$ date ; who
```

使用 `./test1` 的方式执行脚本，需要文件有执行权限，用 `chmod u+x` 赋予权限。另外，创建文件时，`umask` 的值决定了新文件的默认权限设置。（参加书本第 7 章）

使用 `sh test1` 的方式执行脚本，不需要文件有执行权限。



使用 `echo` 需要注意的点：

- 默认情况下，不需要使用引号，但是字符串里面有单引号或双引号就有问题了
- 可用单引号或双引号
- 使用 `-n` 参数可以不换行，**不知为何使用 sh 执行时，`-n` 不起作用？**
- 单引号中不能使用变量

因此建议，养成使用双引号的习惯。



使用环境变量：

- `set` 命令查看当前环境变量列表；
- 使用 `echo $PATH`、`echo $USER` 等方法使用；



## 使用变量

- 在变量、等号和值之间不能出现空格；
- 区分大小写；
- 引用变量有两种方式：`$variable` 或 `${variable}`



## 命令替换

- 反引号 `
- `$()` 格式，最好用这种方式。

```bash
testing=`date`
testing=$(date +%y%m%d)
```



内联输入重定向：

使用符号 `<<`，还必须指定一个文本标记来划分输入数据的开始和结尾。任何字符串都可作为文本标记，但在数据的开始和结尾文本标记必须一致。shell 会用 `PS2` 环境变量（参见第六章）中定义的次提示符来提示输入数据。

```bash
$ wc << EOF
heredoc> test string 1
heredoc> test string 2
heredoc> test string 3
heredoc> EOF
       3       9      42
```



管道：

- 管道两边的命令会同时运行
- 第一个命令输出的同时，输出会被立即送给第二个命令。数据传输不会用到任何中间文件或缓冲区。

```bash
$ rpm -qa | sort
```



## 数学运算

bash shell 只支持整数运算。

三种方式：

- `expr` 命令，符号两边必须有空格，特殊符号如 `*` 需要转义
- 使用方括号`$[operation]`，不必注意空格，可以使用任何算数运算
- 使用双括号 `$((operation))`，**推荐使用**。其实相当于双括号里面执行算数运算。

```bash
$(expr 5 \* 2)
$(expr 5 + 2)
var4=$[$var1 * ($var2 - $var3)]
$((var2 * var1))
```



浮点数运算，使用内建的 bash 计算器——`bc`。

使用 `bc` 命令进入计算器，`quit` 退出。`-q` 命令行选项可以不显示欢迎信息。

浮点数运算是由 `scale` 变量控制的，如果不指定的话，就是 0

bash计算器还能支持变量

```bash
$ bc -q
3.14/5
0
scale=4
3.14/5
.6280

var1=10
var2=10*2
print var2
20
quit
$ 
```



在脚本中使用 `bc`，使用命令替换的方式，格式为：

`variable=$(echo "options; expression" | bc)`

```shell
#!/bin/bash
var1=100
var2=45
var3=$(echo "scale=4; $var1 / $var2" | bc)
echo "The answer for this is $var3"
```

`bc` 和内联重定向结合使用，可以让多个运算过程变得清晰，需要注意的是，在计算器里面创建的变量，不能在 shell 脚本中使用。

```bash
# bc 和内联输入重定向结合使用
var8=$(bc << EOF
scale = 4
a1 = $var4 * $var5
b1 = $var6 * $var7
a1 + b1
EOF
)
```



## 退出脚本

退出状态码是一个 0～255的整数值，一个成功结束的命令的退出状态码是 0。如果超过这个范围，最终的结果就是指定的数值除以 256 后得到的余数。

使用 `$?` 来保存上个已执行命令的退出状态码。

使用 `exit` 命令指定退出状态码。

常用的退出状态码：

| 状态码 | 描述                           |
| ------ | ------------------------------ |
| 0      | 命令成功结束                   |
| 1      | 一般性未知错误                 |
| 2      | 不适合的shell命令              |
| 126    | 命令不可执行                   |
| 127    | 没找到命令                     |
| 128    | 无效的退出参数                 |
| 128+x  | 与 Linux 信号 x 相关的严重错误 |
| 130    | 通过 Ctrl+C 终止的命令         |
| 255    | 正常范围之外的退出状态码       |

## 数组

对其它 shell 而言，数组变量的可移植性并不好，应尽量减少使用。下面的例子在 zsh 中运行，结果就不一样。

```bash
#!/bin/bash

# 定义数组
myarray=(one two three four)

# one
echo $myarray

# one
echo ${myarray[0]}

# one two three four
echo ${myarray[*]}

myarray[2]=seven

unset myarray[2]

# one two four
echo ${myarray[*]}

# 这个位置是空的
echo ${myarray[2]}

# four
echo ${myarray[3]}

unset myarray

# 空的
echo ${myarray[*]}
```



# 循环语句

## if-then 语句

```shell
if command
then
	commands
fi

# 或者

if command; then
	commands
fi

if grep $testuser /et/passwd
then
	...
fi
```

`if-then-else`

```shell
if command
then
	commands
else
	commands
fi

if command
then
	commands
elif command
then
	commands
fi
```



## test 命令

```shell
# test 命令没有命令的话，以非零的退出状态码退出
if test
then
    echo "test command is ok."
else
    echo "test command is false."
fi

# 可以使用 test 命令确定变量中是否有内容
my_variable="23"
if test $my_variable
then
    echo "variable is not empty."
else
    echo "variable is empty."
fi
```



使用方括号定义测试条件，**第一个方括号之后和第二个方括号之前必须有空格**：

- 数值比较
- 字符串比较
- 文件比较



### 数值比较

`n1 -eq n2`

`n1 -ge n2`

`n1 -gt n2` 

`n1 -le n2`

`n1 -lt n2`

`n1 -ne n2`

```bash
# 方括号前后必须要有空格
# test 命令中不能使用浮点数
if [ $value1 -gt $value2 ]
then
    echo "$value1 is greater than $value2"
elif [ $value1 -lt $value2 ]
then
    echo "$value1 is less than $value2"
else
    echo "$value1 is equal $value2"
fi
echo
```



### 字符串比较

`str1 = str2`

`str1 != str2`

`str1 < str2`

`str1 > str2`

`-n str1` 检查 str1 的长度是否非0

`-z str1` 检查 str1 的长度是否为0

注意，字符串顺序比较中：

- **大于号和小于号必须转义**
- 大于和小于顺序和 sort 命令所采用的不同

在比较测试中，大写字母被认为是小于小写字母的。而 sort 命令相反。

比较测试中用的是标准的 ASCII 顺序。sort 命令使用的是系统的本地化语言设置中定义的排序顺序。对于英语，本地化设置指定了在排序顺序中小写字母出现在大写字母前。

```shell
if [ $USER != $testuser ]
then
	...
fi

if [ $var1 \> $var2 ]
then
	...
fi
```



### 文件比较

`-d file`

`-f file`

`-e file`

`-r file`

`-w file`

`-x file`

`-O file` 检查 file 是否存在并属于当前用户所有

`-G file` 检查 file 是否存在并且默认组与当前用户相同

`-s file` 检查 file 是否存在并非空

`file1 -nt file2`

`file1 -ot file2`

使用 `-nt` 或 `-ot` 比较文件之前，最后先检查文件是否存在：

```bash
# 文件不存在，返回 false
if [ badfile1 -nt badfile2 ]
then
	echo "The badfile1 file is newer than badfile2"
else
	echo "The badfile1 file is older than badfile2"
fi

# 结果是
The badfile1 file is older than badfile2
```

### 复合条件测试

```shell
[ condition1 ] || [ condition2 ]

[ condition1 ] && [ condition2 ]
```

### 双括号和双方括号

双括号里面可以进行算数运算。

双方括号里面可以进行字符串处理。

记住，不管是方括号还是双方括号，都需要有空格。而单括号的命令替换和双括号的算数运算，不需要有空格。

```bash
# 双括号里面不需要使用 $ 
if ((val1 ** 2 > 90))
then
    # 赋值时 = 两边可以有空格
    ((val2 = val1 ** 2))
    echo "The square of $val1 is $val2"
fi

# 方括号两边必需要有空格，模式匹配功能
if [[ $USER == x* ]]
then
    echo "Hello $USER."
else
    echo "Sorry, I do not know you."
fi
```



## case 命令

```shell
case variable in
pattern1 | pattern2) commands1;;
pattern3) commands2;;
*) default commands;;
esac
```

注意结尾的双分号

```shell
case $USER in
xiaocan | barbara)
    echo "Welcome, $USER"
    echo "Please enjoy your visit.";;
testing)
    echo "Special testing account.";;
jessica)
    echo "Do not forget to log off when you're done.";;
*)
    echo "Sorry, your are not allowed here";;
esac
```



## for 命令

```shell
for var in list
do
	command
done
```

示例：

```shell
for test in Alabama Alaska Arizona Arkasas California
do
    echo The next state is $test
done

# 不同于其他编程语言，在 for 循环结束后，也能够使用 test 变量
echo $test

# 对单引号转义，或者使用双引号包括它
for test in I don\'t know if "this'll" work
do
    echo "word: $test"
done

# 使用双引号包括多个词汇
for test in Nevada "New Hampshire" "New York"
do
    echo "City: $test"
done

# 字符串拼接的常用方法
list=$list" Connecticut"


# 怎么打印带有空格的字符呢？
files="states"
# 更新字段的分割符
IFS=$'\n'
for state in $(cat $files)
do
    echo "Visit beautiful $state"
done

```

for 循环假定每个值都是空格分割的。使用 `IFS` （内部字段分隔符 internal field separator）指定分隔符。

```shell
IFS.OLD=$IFS
IFS=$'\n'
# 之后再还原 IFS 的值
IFS=$IFS.OLD

IFS=:
# 换行符、冒号、分号、双引号
IFS=$'\n':;"
```

读取目录：

```shell
# 可以指定多个目录
for file in /Users/xiaocan/Documents/learn/shell-learn/* /home/rich/badtest
do
    # 防止 file 名称有空格，所以添加引号
    if [ -d "$file" ]
    then
        echo "$file is a directory."
    elif [ -f "$file" ]
    then
        echo "$file is a file."
    fi
done
```



### C 语言风格的 for 命令

```shell
# 括号两边的空格不是必需的
for (( i = 1; i <= 10; i++ ))
do
    echo "The next number is $i"
done
echo

for (( a=1, b=10; a <= 10; a++, b--))
do
    echo "$a - $b"
done
```



## while 命令

```shell
while test command
do
	commands
done
```

示例：

```bash
var1=10
while [ $var1 -gt 0 ]
do
    echo $var1
    var1=$(( var1 - 1 ))
done
echo

var1=5
# 多个测试命令
# 只有最后一个命令成立时才会停止。如果互换了下面两条命令的位置，就一直循环下去了
while echo $var1
        [ $var1 -gt 0 ]
do
    echo "This is inside the loop"
    var1=$(( var1 - 1 ))
done
```

## until 命令

```shell
until test command
do
	commands
done
```

示例：

```shell
var1=100

until [ $var1 -eq 0 ]
do
    echo $var1
    var1=$(( var1 - 25 ))
done
echo

# 使用多个测试命令
var1=100
until echo $var1
        [ $var1 -eq 0 ]
do
    echo Inside the loop.
    var1=$(( var1 - 25 ))
done
```

## break 和 continue

和编程语言中的 break 和 continue 类似，不同的是，在 shell 中它们后面可以跟一个数字，指定跳出的层数。

```shell
# 注意在 while 中使用 continue，下面的例子不会结束
var2=1
while [ $var2 -lt 15 ]
do
    if [ $var2 -gt 5 ] && [ $var2 -lt 10 ]
    then 
        continue
    fi
    echo "The value is $var2"
    var2=$((var2+1))
done
```

## 处理循环的输出

可以在 `done` 命令之后添加一个处理命令。

```shell

# 在 done 后面使用管道或重定向命令
for state in "North Dakata" Connecticut Illinois Alabama Tennessee
do
    echo "The state is $state"
done | sort
# 或者
# done > test.txt
```



# 处理用户的输入

## 常用参数变量

```shell
$0 脚本名，带有路径
# 使用 basename 只获取名称
$(basename $0)

$1 ... $9, ${10}, ${11} ...

# 在使用参数前注意检查是否存在
if [ -n "$1" ]
then
	...
fi

# 特殊参数变量
$# 获取参数的个数

${!#} 获取最后一个参数。如果没有参数，${!#} 返回的是脚本的名称

# 这种方法也是返回参数的个数，如果没有参数，返回 0
last=$#
echo "The parameter number is $last"

$* 把所有的输入当作一个整体

$@ 会分割输入的参数，可使用 for 循环分别获取

for param in "$@"
do
	...
done
```

## `shift` 命令移动变量

作用：将每个参数变量向左移动一个位置。变量 `$0` 的值不会变。

如果某个参数被移除，它的值就被丢弃了，无法再恢复。

```shell
#!/bin/bash

# shift 后面也可以跟参数，表示一次移动多个
count=1
while [ -n "$1" ]
do
    echo "Parameter #$count = $1"
    count=$((count+1))
    shift
done

# 使用 shift 之后，就无法再获取参数的值了
echo "The second parameter is $2"
```

## 处理选项

命令行选项后面可以带参数；

shell 会用双破折号来表明选项列表结束，在双破折号之后，就可以将剩下的命令行参数当作参数；

```shell
while [ -n "$1" ]
do
    case "$1" in
        -a) echo "Found the -a option" ;;
        -b) echo "Found the -b option, with parameter value $2"
            shift ;;
        -c) echo "Found the -c option" ;;
        # 使用 -- 把命令选项和参数分隔开
        --) shift
            break;;
        *) echo "$1 is not an option";;
    esac
    # 移动变量
    shift
done

# 读取参数
count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$((count+1))
done

# 测试
$ sh testsh.sh -a -b 23 -c -d -- param1 param2
```

弊端，将多个选项放在一起就不能用了，比如 `-ab`

### 使用 getopt 

`getopt optstring parametrs`

optstring 中列出要在脚本中用到的每个命令行选项字母，在每个需要参数值的选项字母后加一个冒号。

```bash
$ getopt ab:cd -a -b test1 -cde test2 test3
getopt: illegal option -- e
 -a -b test1 -c -d -- test2 test3
```

如果指定了一个不在 optstring 中的选项，会抛出错误信息，使用 `-q` 选项可以忽略。

在脚本中使用 getopt 命令：

`set` 命令的选项之一是双破折号（—），它会将命令行参数替换成 set 命令的命令行值。

```bash
# 使用 getopt 处理命令参数
# getopt 处理带空格的参数有问题
set -- $(getopt -q ab:cd "$@")

# 剩下的和上面的处理一样
```

弊端，处理带空格的参数值有问题。

### 使用 getopts

`getopts optstring variable`

每次调用它时，只处理命令行上检测到的一个参数。处理完所有的参数之后，会退出并返回一个大于 0 的退出状态码。

要去掉错误消息的话，可以在 optstring 之前加一个冒号。

用到两个环境变量。如果选项需要跟一个参数值，`OPTARG` 环境变量就会保存这个值。`OPTIND` 环境变量保存了参数列表中 getopts 正在处理的参数位置。

```shell
#!/bin/bash

echo
while getopts :ab:c opt
do
    case "$opt" in
        # 这边不需要单破折号
        a) echo "Found the -a option" ;;
        b) echo "Found the -b option, with value $OPTARG" ;;
        c) echo "Found the -c option" ;;
        *) echo "Unknown option: $opt"
    esac
done

# 移动参数
shift $((OPTIND - 1))

count=1
for param in "$@"
do
    echo "Parameter #$count: $param"
    count=$((count+1))
done


# 使用示例
# 选项可以放在一起
$ ./test19.sh -ab test1 -c

# 参数中可以有空格
$ ./test19.sh -ab "test test" -c

# 参数和选项不用空格分开
$ ./test19.sh -abtest

# 会将没有定义的选项输出为问号


```



## 将选项标准化

有些字母选项在 Linux 世界里已经有了某种程度的标准含义。



## 获取用户输入 read

使用 `read` 命令：`-p` `-s` `-n` `-t` 

```shell
read name
echo "Your name is $name"

# 使用 -p 参数
read "Please input your name: " name
echo "Your name is $name"

# read 一次性读入多个变量
read -p "please input two number: " a b
echo "The sum of the two numbers is: $((a+b))"

# 使用 -s 参数，隐藏输入内容
read -s -p "Please input your password: " password
echo "Is your password really $password?"

# read 后面不指定变量，会放在 REPLY 环境变量中
read -p "Enter your name: "
echo "Your name is $REPLY"

# 使用 -t 指定等待的秒数
if read -t 5 -p "Please enter your name: " name
then
    echo "Hello $name"
else
    echo "Sorry, too slow!"
fi

# 使用 -n 指定输入的字符数。输入的字符数达到指定的值，就会自动回车完成输入
read -n1 -p "Do you want to continue [Y/N]? " answer
case $answer in
Y | y) echo
        echo "fine, continue on...";;
N | n) echo
        echo "Ok, goodbye"
        exit;;
esac
```

读取文件：

```shell
count=1
# 每次从文件中读取一行
cat states | while read line
do
    echo "Line $count: $line"
    count=$((count+1))
done
echo "Finished processing the file"
```

# 呈现数据

Linux 系统将每个对象当作文件处理。Linux 用文件描述符（file descriptor）来标识每个文件对象。

每个进程一次最多可以有九个文件描述符。bash shell 保留了个前三个（0、1、2）。

0 STDIN，标准输入；

1 STDOUT，标准输出；

2 STDERR，标准错误；

当在命令行上只输入 `cat` 命令时，它会从 STDIN 接受输入。输入一行，cat 命令就会显示出一行。

```bash
# 只重定向错误
$ ls -la badfile 2> errorfile

# 重定向错误和数据
$ ls -la test test2 badfile 2> errorfile 1> outfile

# 重定向错误和数据到一个文件
$ ls -la test test2 badfile &> outfile
# 注意，为了避免错误信息散落在输出文件中，相较于标准输出，bash shell 自动赋予了错误消息更高的优先级。
# 所以输出文件中，错误信息集中显示在最开头
```

## 在脚本中重定向输出、输入

在重定向到文件描述符时，必需在文件描述符数字之前加一个 `&`

```shell
echo "This is an error msg" >&2
echo "This is a normal msg"
```

如果脚本中有大量数据需要重定向，重定向每个 echo 语句就很烦琐。可以使用 `exec`。

```shell
exec 1>testout

echo "This is a test of redirecting all outoupt"
echo " from a script to another file"
echo "This is an other msg" >&2

exec 2>testerror

echo "This is an other normal msg"
echo "This is an other msg2" >&2
```

重定向输入：

```shell
exec 0< states

count=1

while read line
do
    echo "State #$count: $line"
    count=$((count+1))
done
```

## 创建自己的重定向

其它6个3～8的文件描述符可用作输入或输出重定向。

```shell
exec 3>test28out
# 当然也可以追加
# exec 3>>test28out

echo "1. This should display on the monitor"
echo "and this should be stored in the file" >&3
```

一旦重定向了 STDOUT 或 STDERR，怎么再切回来呢？

```shell
# 在脚本中临时重定向输出，然后恢复默认输出设置
# 3->monitor
exec 3>&1
echo "2. This should display on the monitor"
# 1->file
exec 1>test28out2
echo "This should be stored in the file"
echo "3. This should display on the monitor" >&3

# 1->monitor
exec 1>&3
echo "4. This should display on the monitor"
```

同样的方法用在重定向输入中：

```shell
#!/bin/bash

# 保存 STDIN，之后再恢复
exec 6<&0

exec 0< states

count=1
while read line
do
    echo "State #$count: $line"
    count=$((count+1))
done

exec 0<&6
read -p "Are you done now? " answer
case $answer in
    Y|y) echo "Goodbye";;
    N|n) echo "Sorry, this is the end";;
esac
```

创建读写文件描述符，即打开单个文件描述符来作为输入和输出，注意，**任何读或写都会从文件指针上次的位置开始**：

```shell
#!/bin/bash

# 创建读写文件描述符

exec 3<>testfile

# 读第一行
read line <&3
echo "Read: $line"
# 此时文件指针在第二行的开头
echo "This is a test line" >&3

#testfile
# This is the first line.
# This is the second line.
# This is the third line.

# 运行结束查看 testfile
# This is the first line.
# This is a test line
# ine.
# This is the third line.
```

关闭文件描述符 `exec 3>&-`，关闭之后就不能使用了。不知为何在 mac 上没有效果：

```shell
#!/bin/bash

exec 3>test31file
echo "This is a test line of data" >&3

# 关闭文件描述符
# mac没有生效
echo 3>&-

echo "This won't work" >&3
```

如果在关闭之后，随后在脚本中打开了同一个输出文件，会用一个新文件来替换已有文件。

```shell
#!/bin/bash

exec 3>test32file
echo "This is a test line of data" >&3
exec 3>&-

cat test32file

# 会覆盖上面的文件
exec 3>test32file
echo "This'll override file" >&3
```

## 列出打开的文件描述符 lsof

`lsof` 会列出整个 Linux 系统打开的所有文件描述符。

在很多 Linux 系统中，lsof 命令位于 `/usr/sbin` 目录，要想以普通用户账号来运行，必需通过全路径名来引用：`/usr/sbin/lsof`

`-p` 指定进程ID（PID）

`-d` 指定要显示的文件描述符编号

`-a` 用来对其它两个选项的结果执行布尔 AND 运算

`$$` 环境变量，当前进程的PID

```bash
$ lsof -a -p $$ -d 0,1,2
COMMAND   PID    USER   FD   TYPE DEVICE  SIZE/OFF NODE NAME
zsh     45488 xiaocan    0u   CHR   16,0 0t1828098 4717 /dev/ttys000
zsh     45488 xiaocan    1u   CHR   16,0 0t1828098 4717 /dev/ttys000
zsh     45488 xiaocan    2u   CHR   16,0 0t1828098 4717 /dev/ttys000
```

`FD` 文件描述符以及访问类型（r-read，w-write，u-读写）

`TYPE` 文件的类型（`CHR` 代表字符型，`BLK` 代表块型，`DIR` 代表目录，`REG` 代表常规文件）

与 STDIN、STDOUT 和 STDERR 关联的文件类型是字符型。

```shell
#!/bin/bash

exec 3> test33file1
exec 6> test33file2
exec 7< states

/usr/sbin/lsof -a -p $$ -d 0,1,2,3,6,7,8

# 结果都是 REG 类型的，代表它们都是常规的文件
$ sh test33-lsof.sh
COMMAND   PID    USER   FD   TYPE DEVICE SIZE/OFF       NODE NAME
bash    48443 xiaocan    0u   CHR   16,1  0t22389       5055 /dev/ttys001
bash    48443 xiaocan    1u   CHR   16,1  0t22389       5055 /dev/ttys001
bash    48443 xiaocan    2u   CHR   16,1  0t22389       5055 /dev/ttys001
bash    48443 xiaocan    3w   REG    1,4        0 8635172069 /Users/xiaocan/Documents/learn/shell-learn/test33file1
bash    48443 xiaocan    6w   REG    1,4        0 8635172070 /Users/xiaocan/Documents/learn/shell-learn/test33file2
bash    48443 xiaocan    7r   REG    1,4       50 8634917711 /Users/xiaocan/Documents/learn/shell-learn/states
```

## 阻止命令输出

将文件重定向到 `/dev/null` 文件。

可以用它来清空日志文件：`cat /dev/null > logfile`

## 创建临时文件

系统上任何用户都有权限读写 `/tmp` 目录中的文件。

```shell
# mac 上的运行结果
$ mktemp
/var/folders/gc/j2xc0rlx5bx4frx_54gb6sbr0000gn/T/tmp.olLTfDFA

$ mktemp testing.XXXXXX
testing.t3DvSz

# 只有属主用户有权限
$ ls -l testing.t3DvSz
-rw-------  1 xiaocan  staff  0 Feb 13 20:15 testing.t3DvSz

# -t 参数指定在临时文件夹中创建文件
$ mktemp -t testing.XXXXXX
/var/folders/gc/j2xc0rlx5bx4frx_54gb6sbr0000gn/T/testing.XXXXXX.B5qQrUge

# -d 参数创建临时文件夹
$ mktemp -d 


# 在脚本中使用
# tempfile=$(mktemp test34.XXXXXX)
```



## 记录消息 tee

同时重定向到控制台和文件中。

```bash
$ date | tee testfile

echo "test message" | tee testfile

# 使用 -a 追加
echo "test message" | tee -a testfile
```

# 函数

## 创建函数

```shell
function name {
	commands
}

# 或者

name() {
	commands
}
```

注意，函数必需先定义，然后才能使用。

使用举例：

```shell
#!/bin/bash

# 使用函数

function func1 {
    echo "This is an example of a function"
}

count=1
while [ $count -le 5 ]
do
    func1
    count=$((count+1))
done

# 会覆盖上面的定义
function func1 {
	echo "This is another example"
}

func1

```

## 返回值

有三种方式获取：

第一，默认最后一个命令的退出状态码；

```bash
#!/bin/bash
# 不推荐使用
func1() {
    echo "trying to display a non existent file"
    ls -l badfile
}

func1
echo "The exit status is: $?"
```

第二，使用 `return` 命令；

```shell
#!/bin/bash
# 使用 return 命令
# 有两点需要注意：
# 1. 函数一结束就取返回值
# 2. 退出状态码必须是 0～255
function db1 {
    read -p "Enter a value: " value
    echo "doubling the value"
    return $((value * 2))
}
db1
echo "The new value is $?"
```

第三，使用函数输出，推荐使用；

```shell
#!/bin/bash

function db1 {
    read -p "Enter a value: " value
    echo $((value * 2))
}
# 命令替换
result=$(db1)
echo "The new value is $result"
```

## 向函数传递参数

同脚本获取命令行参数一样，函数也使用 `$1` … `$9` … 获取参数。`$0` 是函数名。

```shell
#!/bin/bash

# 向函数传递参数

function addem {
    if [ $# -eq 0 ] || [ $# -gt 2 ]
    then
        echo -1
    elif [ $# -eq 1 ]
    then
        echo $(($1 + $1))
    else
        echo $(($1 + $2))
    fi
}

echo "Adding 10 and 15: $(addem 10 15)"

```

## 在函数中使用变量

默认情况下，在脚本中定义的变量都是全局变量，在函数内外都可以使用。

```shell
#!/bin/bash

# 默认都是全局变量

function db1 {
    value=$((value * 2))
}

read -p "Enter a value: " value
db1
echo The new value is: $value
```

声明局部变量，在变量前加上 `local` 关键字。

```shell
#!/bin/bash

# 定义局部变量

function func1 {
    local temp=$((value+5))
    result=$((temp * 2))
}

temp=4
value=6

func1
# 22
echo "The result is $result"
# 4
echo "The temp value is $temp"
```

## 向函数传递数组

```shell
#!/bin/bash

# 给函数传递数组

wrong() {
    echo "The parameters are: $@"
    thisarray=$1
    echo "The received array is ${thisarray[*]}"
}

myarray=(1 2 3 4 5)
echo "The origin array is ${myarray[*]}"
# 这种传递数组的方式不行
wrong $myarray
echo

# 正确使用方法
right() {
    echo "The parameters are: $@"
    # 在内部创建一个新的变量
    newarray=($(echo $@))
    echo "The received array is ${newarray[*]}"
}

right ${myarray[*]}

# 结果是
# The origin array is 1 2 3 4 5
# The parameters are: 1
# The received array is 1
# 
# The parameters are: 1 2 3 4 5
# The received array is 1 2 3 4 5
```

## 函数返回数组



## 递归

```shell
#!/bin/bash

# 递归
# 计算一个数的阶乘

function factorial {
    if [ $1 -eq 1 ]
    then
        echo 1
    else
        local temp=$(($1 - 1))
        local result
        result=$(factorial $temp)
        echo $((result * $1))
    fi
}

read -p "Enter value: " value
result=$(factorial $value)
echo "The factorial of $value is: $result"
```

## 创建库

创建库文件，然后在多个脚本中引用，方便函数重用。

示例库函数：

```shell
cat myfuncs 
# my script functions

function addem {
    echo $(($1 + $2))
}

function multem {
    echo $(($1 * $2))
}

function divem {
    if [ $2 -ne 0 ]
    then
        echo $(($1 / $2))
    else
        echo -1
    fi
}
```

和环境变量一样，shell 函数仅在定义它的 shell 会话内有效。在命令行中运行 myfuncs 脚本，**shell 会创建一个新的 shell 运行它**，之后在当前的 shell 中也无法使用它：

```bash
$ ./myfuncs 
$ addem 1 2
bash: addem: command not found
```

`source` 命令会在当前 shell 上下文中执行命令，而不是创建一个新 shell。source 命令有一个快捷的别名，称作**点操作符（dot operator）**。

```shell
#!/bin/bash
# 使用库函数

# 这种方式不正确
# ./myfunc

# 使用这种方式执行
. ./myfuncs

result=$(addem 10 15)
echo "The result is $result"
```

## 在命令行上使用库函数

在命令行上创建函数：

```bash
$ function divem { echo $(($1 / $2));}
$ divem 5 2
2

$ function multem {
> echo $(($1 * $2))
> }
$ multem 2 5
10
```

在 `.bashrc` 中定义函数，或者使用点操作符加载库函数。这样就可以在 shell 中是使用了。



# 控制脚本




