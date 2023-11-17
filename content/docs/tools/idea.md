---
title: "借助wslg使用Linux版本的idea"
date: 2023-08-29T23:56:17+08:00
draft: true
---

## 缘由

大部分Java程序员都会使用idea在Windows下进行开发，然后部署到Linux服务器上，大部分情况下，Java强大的跨平台性使得这种方式没有问题。但是！在某些时候，开发使用到的平台api会出现一些问题，导致跨平台很麻烦，比如selenium相关开发，需要用到selenium组件，此组件需要配置一些运行环境，为了避免运行环境差异导致的