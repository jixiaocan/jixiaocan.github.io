---
title: Linux 中正则表达式的使用
date: 2020-02-20 18:04:45
tags: [正则表达式]
categories: [learn, Linux]
snippet: linux, regex, 正则表达式, shell, awk, sed
seo:
  date_modified: 2020-03-09 21:47:22 +0800
---


# 概述

有不止一种正则表达式。它通过正则表达式引擎（regular expression engin）实现的。

在 Linux 中，有两种流行的正则表达式引擎：

- POSIX 基础正则表达式（basic regular expression，BRE）引擎；
- POSIX 扩展正则表达式（extended regular expression，ERE）引擎；

sed 编辑器只符合 BRE 引擎。gawk 支持 ERE 引擎。

# 基础正则表达式

## 点号和字符组

点号：匹配除换行符之外的任意单个字符。

字符组（character class）：匹配方括号里面的任意一个字符，比如 `[ch]`。

排除型字符组：在字符组里面添加 `^`，比如 `[^ch]`。

```bash
$ cat data6
This is a test of a line.
The cat is sleeping.
That is a very nice hat.
This test is at line four.
at ten o'clock.

# 注意最后一行没有匹配上，因为 at 前面的换行符不能匹配点号
$ sed -n '/.at/p' data6
The cat is sleeping.
That is a very nice hat.
This test is at line four.

# 匹配 cat 或 hat
$ sed -n '/[ch]at/p' data6
The cat is sleeping.
That is a very nice hat.

# 匹配除了 cat 或 hat 之外的字符
$ sed -n '/[^ch]at/p' data6
This test is at line four.

# 多个字符组一起使用
$ echo "Yes" | sed -n '/[Yy][Ee][Ss]/p'
Yes
# 但也会匹配这个
$ echo "Yess" | sed -n '/[Yy][Ee][Ss]/p'
Yess
# 优化，加上行首和行尾的限制
$ echo "Yess" | sed -n '/^[Yy][Ee][Ss]$/p'
```

## 区间

比如 `[0-9]` 匹配数字 0～9，相当于 `[0123456789]`。

`[a-h]` 匹配字母 a～h。

还可以指定多个不连续区别，比如 `[a-ch-m]`，匹配 a～c 和 g～m 中的字母。

## 特殊字符组

BRE 还包含了一些特殊的字符组：

| 组          | 描述                                           |
| ----------- | ---------------------------------------------- |
| [[:alpha:]] | 匹配任意字母，不管是大写还是小写               |
| [[:alnum:]] | 匹配任意字母和数字                             |
| [[:blank:]] | 匹配空格或制表符                               |
| [[:digit:]] | 匹配0～9之间的数字                             |
| [[:lower:]] | 匹配小写字母                                   |
| [[:print:]] | 匹配可打印字符                                 |
| [[:punct:]] | 匹配标点符号                                   |
| [[:space:]] | 匹配任意空白字符：空格、制表符、NL、FF、VT和CR |
| [[:upper:]] | 匹配大写字母                                   |

```bash
$ echo "abc" | sed -n '/[[:digit:]]/p'
$ echo "abc" | sed -n '/[[:alnum:]]/p'
abc
# 匹配标点符号
$ echo "This is a test" | sed -n '/[[:punct:]]/p'
$ echo "This is, a test" | sed -n '/[[:punct:]]/p'
This is, a test
```

## 星号

星号匹配它前面的字符 0 次或多次。

```shell
echo "ik" | sed -n '/ie*k/p'
ik
$ echo "iek" | sed -n '/ie*k/p'
iek
$ echo "ieek" | sed -n '/ie*k/p'
ieek
```

**星号和点号一起使用**，匹配任意数量的任意字符。

```bash
$ echo "This is a regular pattern expression" | sed -n '/regular.*expression/p'
This is a regular pattern expression
```

星号和字符组一起使用：

```bash
echo "bt" | sed -n "/b[ae]*t/p"
bt
$ echo "bat" | sed -n "/b[ae]*t/p"
bat
$ echo "baaaeeet" | sed -n "/b[ae]*t/p"
baaaeeet
```

# 扩展正则表达式

POSIX ERE 模式包括了一些可供 Linux 应用和工具使用的额外符号。gawk 能够识别 ERE 模式，但 sed 编辑器不能。

## 问号

问号匹配前面的字符 0 次或 1 次。

```bash
echo "bt" | gawk '/be?t/{print $0}'
bt
$ echo "bet" | gawk '/be?t/{print $0}'
bet
$ echo "beet" | gawk '/be?t/{print $0}'
$ 
```

和字符组结合使用：

```bash
echo "bet" | gawk '/b[ae]?t/{print $0}'
bet
$ echo "bat" | gawk '/b[ae]?t/{print $0}'
bat
$ echo "baet" | gawk '/b[ae]?t/{print $0}'
$ 
```



## 加号

加号匹配前面的字符 1 次或多次。

```bash
$ echo "bet" | gawk '/be+t/{print $0}'
bet
$ echo "beeet" | gawk '/be+t/{print $0}'
beeet
$ echo "bt" | gawk '/be+t/{print $0}'
$
```

和字符组结合使用：

```bash
$ echo "bat" | gawk '/b[ae]+t/{print $0}'
bat
$ echo "baet" | gawk '/b[ae]+t/{print $0}'
baet
$ echo "bt" | gawk '/b[ae]+t/{print $0}'
$
```



## 花括号

`{m}` 匹配字符 m 次；

`{m, n}` 匹配字符至少 m 次，至多 n 次。

```bash
$ echo "bt" | gawk '/be{1}t/{print $0}'
$ echo "bet" | gawk '/be{1}t/{print $0}'
bet
$ echo "beet" | gawk '/be{1}t/{print $0}'
$ echo "beet" | gawk '/be{1,2}t/{print $0}'
beet
```

## 管道符号

用**逻辑或**方式匹配字符，其中一个匹配就可以。

```bash
$ echo "The cat is asleep" | gawk '/dog|cat/{print $0}'
The cat is asleep
```

## 表达式分组

```bash
$ echo "Sat" | gawk '/Sat(urday)?/{print $0}'
Sat
$ echo "Saturday" | gawk '/Sat(urday)?/{print $0}'
Saturday
$ echo "cat" | gawk '/(c|a)a(b|t)/{print $0}'
cat

```

# 实战

下面的例子计算环境变量 PATH 中定义的目录里的可执行文件的数量。

```bash
$ echo $PATH | sed 's/:/ /g'
/home/xiaocan/gems/bin /usr/local/sbin /usr/local/bin /usr/sbin /usr/bin /sbin /bin /usr/games /usr/local/games

$ cat countfiles
#!/bin/bash
# count number of files in your PATH
mypath=$(echo $PATH | sed 's/:/ /g')
count=0
for path in $mypath
do
	check=$(ls $path)
	for item in $check
	do
		count=$(($count+1))
	done
	echo "$path - $count"
	count=0
done

$ sh countfiles
/home/xiaocan/gems/bin - 7
/usr/local/sbin - 0
/usr/local/bin - 6
/usr/sbin - 237
/usr/bin - 1699
/sbin - 273
/bin - 176
/usr/games - 1
/usr/local/games - 0

```





