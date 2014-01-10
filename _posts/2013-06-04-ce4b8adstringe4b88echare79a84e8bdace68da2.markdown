---
author: bydiao
comments: true
date: 2013-06-04 08:54:37+00:00
layout: post
slug: c%e4%b8%adstring%e4%b8%8echar%e7%9a%84%e8%bd%ac%e6%8d%a2
title: C++中string与char*的转换
wordpress_id: 367
categories:
- C++&amp;STL
tags:
- C++
---

这回一个老生常谈的问题了，每次遇到这个问题都会去百度。但最近发现，要成为一个优秀的程序员，很多代码都要熟烂于心，而不是遇到问题就去百度。因此，经常去百度的问题，要尝试着记住，起码要记录在自己的博客上，而不是总去查阅别人的博客。


string转char*
[code lang="cpp"]
string abc;
const char* pc1 = abc.c_str();
const char* pc2 = abc.data();
[/code]

因为string是只读的，所以转换到char*，也是只读的，如想修改，需要另一个数组，用strcpy函数复制过去，再修改。

char* 转string
[code lang="cpp"]
char* p = "abc";
string a(p);

string b;
b.assign(p,strlen(p));
[/code]

可以在初始化中转换，也可以实例化以后用assign方法转换。
