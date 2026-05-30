---
categories:
- docs
title: Implementing Trojan with Netty (Part 1)
date: 2023-08-25T09:48:31+08:00
tags:
- Netty
- java
- kotlin
- surfer
- trojan
draft: false
---

## Full Code

[selcarpa/surfer tag(1.13-SNAPSHOT)](https://github.com/selcarpa/surfer/releases/tag/1.13-SNAPSHOT)

## Tech Stack

- Kotlin
- Netty
- Trojan

## Introduction

### Kotlin

The project uses Kotlin, a statically typed JVM-based programming language. It compiles to Java bytecode and is fully compatible with the Java ecosystem, seamlessly interoperating with Java code. Its syntax is very similar to Java but offers additional features such as null safety, extension functions, operator overloading, lambda expressions, and property delegation.

### Netty

Netty is an asynchronous event-driven network application framework for rapid development of maintainable, high-performance protocol servers and clients. Netty is an NIO client-server framework that enables quick development of network applications such as server and client protocols. Netty provides a new way to develop network applications that is easy to use and highly extensible.

### What is Trojan

The [Trojan](https://trojan-gfw.github.io/trojan/protocol.html) protocol is a proxy protocol similar to SOCKS. Its key feature is the ability to transmit within a TLS connection, hiding traffic within normal HTTPS connections to evade traffic censorship. For authentication, Trojan uses a 56-byte field to verify client and server identity. In interactive messages, the Trojan protocol's `Trojan request` structure is almost identical to the SOCKS5 `SOCKS5 CMD`.

In early implementations, the Trojan protocol was designed to listen on port 443, performing TCP protocol interaction after SSL authentication. Starting with Trojan on v2ray, the Trojan protocol can be transmitted over any stream. In some proxy configurations, transmission is often done via TLS+WS+Trojan, where Trojan protocol content is sent as regular HTTP traffic, effectively hiding traffic characteristics while leveraging CDN services for better routing.

## Approach

Netty, as a key Java networking framework, handles TCP/UDP protocol reception and transmission well. This article introduces how to use Netty to implement the Trojan protocol, compatible with v2fly's TLS+WS+Trojan configuration.

In this setup, Trojan protocol packets are transmitted as binary data within WebSocket binary frames, while WebSocket content travels as TCP packets within TLS connections. To handle this, we need to encapsulate Trojan packets into WebSocket binary frames and parse WebSocket binary frames back into Trojan packets.

## Goals

- Implement a client and server compatible with [v2fly/v2fly-core](https://github.com/v2fly/v2ray-core)'s WS+TLS+Trojan configuration.
- Implement HTTP and SOCKS5 inbound proxies for use with HTTP clients and other tools.

## Exploration

- Explore the implementation of SOCKS5 UDP ASSOCIATE.

## Testing

This article uses curl as the client for testing:

```shell
export http_proxy=http://127.0.0.1:14271
export https_proxy=http://127.0.0.1:14271

curl -v http://www.google.com
curl -v https://www.google.com
```

> *This article is translated by deepseek-v4-flash (model: deepseek/deepseek-v4-flash).*
