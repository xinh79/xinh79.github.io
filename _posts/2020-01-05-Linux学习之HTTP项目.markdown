---
layout:      post
title:       "Linux学习之HTTP项目"
subtitle:    "Linux System"
author:      "Ashior"
header-img:  "img/ReadNotes/bg-linux system.jpg"
catalog:     true
tags:
  - 学习笔记
  - Linux
  - 工作
---

> 此篇文章主要是根据Linux视频教程与网上的相关教程所制作的笔记，为书籍UNP做一个补充，使读者更容易理解书籍内容。

读书笔记请参考此篇文章：

- [APUE-1-文件IO](https://xinh79.github.io/2019/11/21/APUE-1-%E6%96%87%E4%BB%B6IO/) 
- [APUE-2-文件和目录](https://xinh79.github.io/2019/11/22/APUE-2-%E6%96%87%E4%BB%B6%E5%92%8C%E7%9B%AE%E5%BD%95/)
- [APUE-3-I/O库与数据文件和信息](https://xinh79.github.io/2019/11/22/APUE-3-IO%E5%BA%93%E4%B8%8E%E6%95%B0%E6%8D%AE%E6%96%87%E4%BB%B6%E5%92%8C%E4%BF%A1%E6%81%AF/)
- [APUE-4-进程基础](https://xinh79.github.io/2019/11/24/APUE-4-%E8%BF%9B%E7%A8%8B%E5%9F%BA%E7%A1%80/)
- [APUE-5-信号与线程](https://xinh79.github.io/2019/11/24/APUE-5-%E4%BF%A1%E5%8F%B7%E4%B8%8E%E7%BA%BF%E7%A8%8B/)
- [APUE-6-进程通信](https://xinh79.github.io/2019/11/28/APUE-6-%E8%BF%9B%E7%A8%8B%E9%80%9A%E4%BF%A1/)
- [Linux学习笔记-I](https://xinh79.github.io/2019/11/22/Linux%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-I/)
- [Linux学习笔记-II](https://xinh79.github.io/2019/11/27/Linux%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-II/)
- [Linux学习笔记-III](https://xinh79.github.io/2019/12/01/Linux%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-III/)
- [Linux学习笔记-IV](https://xinh79.github.io/2019/12/02/Linux%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0-IV/)

> 项目的源码练习可以参考此[github项目](https://github.com/xinh79/Server4C) 。整个项目的思路可以参考此篇文章。

[TOC]

----

## XML与json

#### XML

- XML的特点，有一个头文件`<?xml version="1.0" encoding="ut-8"?>`
- 由一系列的标签组成：逻辑上是树结构；根标签只有一个；一般成对出现`<hello></hello>`；不成对: `<hello/>`
- 标签区分大小写
- 标签可以设置属性`<font color="red"></font>`
- 注释：`<!-- xxxx -->`

#### json

json数组：`[字符串，布尔，整形，json数组，json对象]`
json对象：`{key1: value, key2: value}`键值不能重复，必须是字符串；value可以是字符串，布尔，整形，json数组，json对象。

可以用cjson来对json进行解析。

----

## http协议-应用层

**请求消息(Request)**：浏览器给服务器发

- 请求行：说明请求类型，要访问的资源，以及使用的http版本；
- 请求头：说明服务器要使用的附加信息
- 空行：空行是必须要有的，即使没有请求数据
- 请求数据：也叫主体，可以添加任意的其他数据

HTTP1.1的五种请求方法，GET和POST为常用的两种。

**GET**：请求指定的页面信息，并返回实体主体。
**POST**：向指定资源提交数据进行处理请求（例如提交表单或者上传文件）。数据被包含在请求体中。POST请求可能会导致新的资源的建立和/或已有资源的修改。

**GET方法会将数据显示在地址栏，而POST方法不会，一般提交表单数据等采用此方法更为安全。**

- 请求行：`HTTP /hello.c HTTP/1.1` `\r\n`
- 请求头：`Server: micro_httpd;User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux i686; rv:24.0) Gecko/201001 01 Firefox/24.0`......
- 空行：`\r\n`
- 请求数据：可有可无

**响应消息(Response)**：服务器给浏览器发

- 状态行：包括http协议版本号，状态码，状态信息
- 消息报头：说明客户端要使用的一些附加信息
- 空行：空行是必须要有的
- 响应正文：服务器返回给客户端的文本信息

状态代码有三位数字组成，第一个数字定义了响应的类别，共分五种类别:

- 1xx：指示信息--表示请求已接收，继续处理
- 2xx：成功--表示请求已被成功接收、理解、接收
- 3xx：重定向--要完成请求必须进行更进一步的操作
- 4xx：客户端错误--请求有语法错误或请求无法实现
- 5xx：服务器端错误--服务器未能实现合法的请求

----

## web服务器实现过程

1. 编写函数解析http请求：GET /hello.c HTTP/1.1\n\r，将上述字符串分为三部分解析出来
2. 编写函数根据文件后缀，返回对应的文件类型
3. sscanf读取格式化的字符串中的数据：使用正则表达式拆分，`[^ ]`的用法
4. 通过游览器请求目录数据：读指定目录内容：opendir、readdir、closedir；scandir扫描dir目录下，不包括子目录，的内容
5. http中数据特殊字符编码解码问题：编码、解码

#### 初始版本

1. 接收两个参数：端口号与工作路径
2. 将端口号转位int型，改变当前工作路径
3. 启动epoll模型

**处理读事件：**
正如前面所说，只处理用户拉去服务器的内容，所以整个源码只适用于处理读事件，如果有需要，可以修改`epoll_server.c`源码中的`epoll_run`方法。

**新建连接请求：**
成功之后，会在`do_accept`方法中打印：`New Client IP: %s, Port: %d, cfd = %d\n`。

**处理http请求：**
在函数`http_request`中使用正则表达式，[常用正则表达式](https://www.jb51.net/tools/regexsc.htm)，拆分出`get /xxx http/1.1`信息。如果是目录，则继续读取目录内容，然后拼一个html网页`<table></table>`发送给游览器，并且实现跳转功能`<a href=\"%s\">%s</a>`。如果是文件，则对文件进行读取传输操作。

**中文乱码：**
乱码会导致找不到对应的中文文件，所以要对其进行相应的编码与解码，对于Linux中，使用UTF-8存储，占用3个字节。

**打开网页标题一直转圈：**
应该准备一个`favicon.ico`文件。但是放了之后依旧出现这种情况。主动断开连接。

#### 线程池

为什么需要线程池？提高程序整体运行效率，因为预备了线程。

线程池分为管理线程与专门工作的线程。

1. 初始化进程：管理者线程，不处理任务；工作的线程，只负责处理任务，去任务队列中领取任务。
2. 需要有一个管理者线程：什么时候需要创建，每隔一段时间判断一次是否超过某个阈值(工作的线程/存活的线程大于85%)，如果超过则创建新的线程。什么时候销毁，如果使用率（工作的线程/存活的线程小于20%）。

----

## 简易细节具体实现

**服务器启动初始化过程：**

1. 改变工作目录（接收传入或者预设的默认工作路径）
2. 创建通信套接字socket（成功返回一个文件描述符，通常设定为AF_INET，SOCK_STREAM与0）
3. 将套接字与本地IP与端口号进行绑定，创建sockaddr_in，用于监听端口的信息（注意将主机字节序转为网络字节序）
4. 同时设置创建的套接字使用端口复用机制（setsockopt函数）
5. 调用listen开始对套接字进行监听
6. 使用的是epoll创建一棵树，将之前创建的sockaddr_in与socket绑定至树上

**开始监听：**

1. 使用epoll_wait函数，获取需要处理的事件个数
2. 跳过非读事件
3. 判断是否为新的连接请求或者是对数据的请求读操作

**新的连接请求：**

- 监听就是将新的连接节点信息（sockaddr_in）加入epoll树中

**游览器发起读数据请求：**

1. 使用自定义的get_line函数，逐一读取
2. 如果读取的数据大小为0，则说明客户端关闭连接
3. 服务器开始读取请求行、请求头、空行、请求数据信息
4. 服务器准备开始拼接，按照格式发送状态行、消息报头、空行、响应正文
5. 处理http请求
6. 对请求的数据进行解码，避免中文乱码
7. 判断是目录文件还是普通文件

**目录文件：**

- 拼接发送给给发送给游览器的报头
- 将目录中的文件读取出来
- 拼接成html时，使用`<a href=\"%s\">%s</a>`实现跳转

**普通文件：**

- 拼接发送给给发送给游览器的报头
- 循环读取数据文件然后发送给游览器

----

项目的源码练习可以参考此[github项目](https://github.com/xinh79/Server4C) 。整个项目的思路可以参考此篇文章。
