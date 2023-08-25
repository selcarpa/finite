---
title: "用netty实现Trojan（一）"
date: 2023-08-25T09:48:31+08:00
draft: false
---

# 用netty实现Trojan（一）

## 完整代码

[selcarpa/surfer tag(1.13-SHAPSHOT)](https://github.com/selcarpa/surfer/releases/tag/1.13-SNAPSHOT)

## 选用技术栈

- kotlin
- Netty
- Trojan

## 思路

netty作为Java网络编程的重要框架，可以很好的实现tcp/udp等协议的接收与发送，在本文中，将介绍如何使用netty去实现trojan协议，兼容v2fly中的tls+ws+trojan的配置方式。那么如何实现这一形式的协议的收发。

在此配置中trojan协议的数据包会作为二进制数据，在websocket的二进制数据帧中进行传输，而websocket的内容会以tcp形式的包，传输在tls的连接之中，对应这部分的传输内容，我们需要实现的内容有这一些：



## 什么是Trojan

[Trojan protocol](https://trojan-gfw.github.io/trojan/protocol.html)

## 目标

兼容实现[v2fly/v2fly-core](https://github.com/v2fly/v2ray-core)的ws+tls+trojan配置方式的客户端及服务端

实现http和socks5入口代理，用以在http或者其他客户端中使用此代理

## 介绍

项目采用kotlin语言，它是一种基于JVM的静态类型编程语言，它可以编译成Java字节码，完全兼容Java的生态系统，可以与Java代码无缝互操作。它的语法与Java非常相似，但是它有更多的特性，比如：null安全、扩展函数、运算符重载、lambda表达式、属性代理等等。

Netty是一个异步事件驱动的网络应用程序框架，用于快速开发可维护的高性能协议服务器和客户端。Netty是一个NIO客户端-服务器框架，使用Netty可以快速开发网络应用，例如服务器和客户端协议。Netty提供了一种新的方式来使开发网络应用程序，这种新的方式使得它很容易使用和有很强的扩展性。

## 全文说明

本文基于curl作为访问客户端进行测试，使用方式如下

```shell
export http_proxy=http://127.0.0.1:14271 # 设置http的环境变量
export https_proxy=http://127.0.0.1:14271 # 设置https的环境变量

curl -v http://www.google.com # 进行一次http的请求
curl -v https://www.google.com # 进行一次https的请求
```
