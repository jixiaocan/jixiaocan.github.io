---
title: chown、chmod 命令使用简介
date: 2019-11-13 20:32:11
tags: [Linux, command]
categories: [learn, Linux]
snippet: linux, 命令行, chown, chmod, 文件权限
seo:
  date_modified: 2020-02-22 17:02:55 +0800
---

# chown
chown命令用于改变文件或目录的用户和用户组。它的用法是：
```
chown [OPTION]... [OWNER][:[GROUP]] FILE...
```
注：`:group` 也可以用 `.group` 表示。

使用方法示例：
```bash
# 修改 temp.txt 所属的用户是 xiaocan
$ chown xiaocan temp.txt

# 修改 temp.txt 所属的组是 xiaocan
$ chown .xiaocan temp.txt
# 或者
$ chown :xiaocan temp.txt

# 同时更改文件所属的用户和组
$ chown root:root temp.txt

# 使用 -R 参数递归修改
$ chown -R root:root dir2/
```

# chmod
chmod 命令用于改变文件或目录权限。它的用法是：
```
chmod [OPTION]... MODE[,MODE]... FILE...
```
模式有两种格式：一种是采用权限字母和操作符表达式；另一种是采用数字。

| 权限位 | 全称 | 含义 | 对应数字 |
|----|----|----|---|
|r | read | 可读权限 | 4 |
|w | write | 可写权限 | 2 |
|x | execute | 可执行权限 | 1 |
|- |   | 没有权限 | 0 |

用户类型有：u（所属用户）、g（所属组）、o（其它用户）、a（所属用户）。

操作符：+（增加权限）、-（减少权限）、=（设置权限）

常用方法示例：
```bash
# 设置所有用户的权限为空
$ chmod a= file1.txt
# 等同于
$ chmod 000 file1.txt

# 设置文件所属的用户有读权限
$ chmod u+r file1.txt

# 设置文件所属的组有读权限
$ chmod g+r file1.txt

# 多个权限操作一起使用
$ chmod ug+r,o-r file1.txt

$ chmod u=rwx,g=rx,o=x file1.txt
# 等同于
$ chmod 751 file1.txt

# 使用 -R 递归授权
$ chmod -R 777 dir2/
```

