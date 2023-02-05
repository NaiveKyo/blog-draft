---
title: Java Networking Overview
author: NaiveKyo
top: false
hide: false
img: 'https://cdn.jsdelivr.net/gh/NaiveKyo/CDN/img/20220425110836.jpg'
coverImg: /img/20220425110836.jpg
cover: false
toc: true
mathjax: false
date: 2023-02-05 22:09:38
summary: "Java 网络概览"
categories: "Java"
keywords: ["Java"]
tags: ["Java", "Network"]
---

# Java Networking

## API Specification

- `java.net`；
- `javax.net`；
- `javax.net.ssl`；
- `com.sun.net.httpserver`；
- `com.sun.net.httpserver.spi`；
- `jdk.net`；



## Java Networking Overview

`java.net` 包为下列功能提供了接口和类：

- Addressing；
- TCP 链接；
- 使用 UDP 发送/接收 数据包；
- 定位网络资源；
- 安全：
  - 认证；
  - 权限；

本文旨在从高层次浏览 `java.net` 包提供的工具。

- 具体的 api 参考 [networking API](https://docs.oracle.com/javase/8/docs/api/java/net/package-summary.html) ；
- 基础网络概念可以参考 [Trail: Custom Networking](https://docs.oracle.com/javase/tutorial/networking/index.html)；

### Addressing

`java.net` 提供以下和地址相关的类：

- `java.net.InetAddress`；
- `java.net.Inet4Address`；
- `java.net.Inet6Address`；
- `java.net.SocketAddress`；
- `java.net.InetSocketAddress`；

（1）有三个类对应 IP 地址：InetAddress、Inet4Address、Inet6Address。

InetAddress 代表一个 IP 地址，它是一个 32 或者 128 位无符号数字，而 IP 是构建 TCP 和 UDP 等协议的底层协议。Inet4Address 表示 32 位的 IP 地址（Ipv4），它是 InetAddress 的子类。Inet6Address 表示 128 位的 IP 地址（Ipv6），它也继承自 InetAddress；

（2）有两个类对应 Socket（套接字） 地址：SocketAddress 和 InetSocketAddress。

SocketAddress 是一个表示套接字地址的抽象，独立于特定的协议，它主要用于子类化特定协议。InetSocketAddress 就是一个例子，它实现了 SocketAddress，表示 IP socket address。包括 IP 地址 + 端口号、主机名 + 端口号、或者只有端口号。

### Making TCP Connections

下面的类是和创建 TCP 连接有关：

- `java.net.ServerSocket`；
- `java.net.Socket`；

如果向实现 client 和 server 之间简单的 TCP 连接，可以使用它们。

ServerSocket 表示服务器上等待和侦听来自客户端的服务请求的套接字。Socket 则表示服务器和客户端之间通信的端点。当服务器获取到从某个服务发过来的请求，它会创建一个 Socket 用于和客户端通信并继续监听 ServerSocket 上的其他请求。客户端也会创建一个 Socket 用于和服务端通信。

一旦建立了连接：client/server Socket 就可以使用`getInputStream()` 和 `getOutputStream` 方法进行数据传输。

### Sending/Receiving Datagram Packets via UDP

下面是通过 UDP 收发数据包的相关操作：

- `java.net.DatagramPacket`；
- `java.net.DatagramSocket`；

DatagramPacket 表示数据报文包。数据报报文用于无连接传输（connectionless），通常包含目标地址和端口信息；

DatagramSocket 是一个套接字，用于通过 UDP 在网络上发送和接收数据报数据包。

DatagramSocket 通过调用 `send(...)` 方法发送一个 DatagramPacket，该方法的参数就是 DatagramPacket，对应的还有一个 `receive(...)` 方法用于接收数据包。

`java.net.MulticastSocket` 类可用于向多播组发送/接收数据报。它是 DatagramSocket 的一个子类，为多播添加了功能。

### Locating/Identifying Network Resources

下面的类和定位网络资源有关：

- `java.net.URI`；
- `java.net.URL`；
- `java.net.URLClassLoader`；
- `java.net.URLConnection`；
- `java.net.URLStreamHandler`；
- `java.net.HttpURLConnection`;
- `java.net.JarURLConnection`；

使用最多的类是 URI、URL、URLConnection 以及 HttpURLConnection；

（1）URI 表示资源的 Uniform Resource Identifier（统一资源标识符）；它是资源的标识符，但不一定是该资源的定位器。

（2）URL 表示 Uniform Resource Locator（统一资源定位器）。URL 是 URI 的子集，尽管类 URL 没有继承 URI，简而言之，URL告诉我们如何访问资源，而 URI 可以，也可以不。The Uniform Resource Name（URN）是 URI 的另一个子集，但是 Java 中没有关于它的类。

（3）URLConnection 是所有这些类的父类：表示应用程序和由 URL 标识的网络资源之间的连接。给定一个 URL 和协议，`URL.openconnection()` 将返回该协议的适当URLConnection 的实例。（URL 中包含 protocol 的信息）。该实例结合 `URLConnection.connect()` 方法提供了打开连接并访问 URL 的功能。

（4）HttpURLConnection 是 URLConnection 的子类中使用最广泛的。它是为 http 协议提供的，用于访问 web 服务器上的内容的协议。

比如说某个 URL 的访问协议是 http，则调用 openConnection 方法返回的就是 HttpURLConnection 实例。

### Security

Security 包含和认证-权限相关的类。Authentication 和用户认证相关，包括用户名、用户密码的校验。比如用户要访问某些资源就需要通过认证才可以访问。下面是和认证相关的类：

- `java.net.Authenticator`；
- `java.net.PasswordAuthentication`；

除了用于用户身份验证的方法外，抽象类 Authenticator 还有用于查询所请求的身份验证的方法(参见`getRequestingXXX()`)。通常会使用它的子类实例调用 `setDefault(Authenticator a)` 方法向系统注册认证器。（注意如果存在 security manager，它会检查安全策略是否允许 `NetPermission“setDefaultAthenticator”`。)然后，当系统需要身份验证时，它将调用 `requestPasswordAuthentication()` 等方法。

PasswordAuthentication 可以简单的存储用户名和密码；

### Permissions

- `java.net.SocketPermission`；
- `java.net.NetPermission`；

SocketPermission 由几部分组成：host、可选的 port，以及可能在该 host 上执行的操作：connect、accept、listen 或者 resolve。它包括确定一个 SocketPermission 是否等于另一个或暗示另一个 Permission 的方法。SocketPermission 可以包含在 PermissionCollection 中，以便于检查权限是否存在。

NetPermission 是用于各种已命名网络权限的类。当前有三种：

- `setDefaultAuthenticator`：前面提到过；
- `requestPasswordAuthentication`；
- `specifyStreamHandler`；

NetPermission 可以包含在 PermissionCollection 中，以便于检查权限是否存在。

更多关于 JDK 的权限信息，可以参考：[Permisson](https://docs.oracle.com/javase/8/docs/technotes/guides/security/permissions.html)



# Trail：Custom Networking

Java 平台受到高度重视，部分原因是它适合编写使用 Internet 和万维网上的资源并与之交互的程序。

参考：https://docs.oracle.com/javase/tutorial/networking/index.html

主要讲了一些关于网络的基础知识以及如何编写 Java 网络代码，但是没有涉及到 SocketChannel；

关于 AIO 在网络编程中的应用可以参考 OpenJDK 文档：

- https://openjdk.org/projects/nio/



进度：

- https://docs.oracle.com/javase/8/javase-books.htm
- https://docs.oracle.com/javase/8/docs/
- https://docs.oracle.com/javase/8/docs/technotes/guides/net/index.html
- https://docs.oracle.com/javase/8/docs/technotes/guides/net/overview/overview.html
- https://docs.oracle.com/javase/tutorial/networking/index.html

