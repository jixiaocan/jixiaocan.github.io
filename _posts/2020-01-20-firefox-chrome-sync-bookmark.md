---
title: 火狐和谷歌浏览器同步书签
date: 2019-11-15 21:35:20
tags: [工具]
categories: [learn, Tools]
snippet: 火狐, Firefox, 谷歌, Google Chrome, 书签同步, 书签搜索, Firefox地址栏搜索功能
seo:
  date_modified: 2020-02-22 17:02:55 +0800
---

在使用多个浏览器的时候，遇到的一个问题就是书签如何同步。[Everhelper](https://www.everhelper.me/synchronizer.php) 是个不错的插件，它有 Firefox 和 Google Chrome 插件。

然而美中不足的是它不支持 Safari 浏览器。

由于平常还需要在 Windows 上使用，因此主要使用 Firefox 和 Chrome 浏览器，再加上 EverSync 插件，从而实现跨浏览器、跨平台的书签同步。

在 Mac 上还可以使用 Alfred 搜索 Safari 和 Chrome 的书签，比如 `bm linux` 搜索标题包含 linux 的书签，非常便捷。

那在 Windows 上如何快速地搜索书签呢？

其实，在 Firefox 的地址栏就可以快速搜索（`Ctrl+L` 定位到地址栏）：
- 添加 `^` 以匹配浏览历史；
- 添加 `*` 以**匹配书签**；
- 添加 `+` 以匹配标签；
- 添加 `%` 以匹配最近打开的标签页；
- 添加 `#` 以匹配页面标题；
- 添加 `$` 以匹配网址；
- 添加 `?` 以匹配建议；

例如，要查找一个名为 Mozilla Firefox Support 的已加入书签的页面，可以输入 `mozilla *` 搜索书签。

还可以通过搜索字符串 `mozilla * support #` 进一步限定，只搜索标题上带有 mozilla 和 support 的书签页面。



