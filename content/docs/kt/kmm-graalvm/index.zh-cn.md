---
categories:
  - docs
title: "kotlin多平台与graalvm如何共存"
date: 2024-05-17T19:30:00+08:00
draft: true
authors: [ "selcarpa" ]
toc:
  enable: true
  auto: true
tags:
  - kotlin
  - kotlin-native
  - kotlin-multiplatform
  - graalvm
---

## 介绍

在 [cloudflare-ddns](https://github.com/selcarpa/cloudflare)，项目中，我想要实现在使用kotlin开发多个平台(
windows-x64，linux-x64，linux-arm64)
原生运行的内容，在开发初期，我选用了kotlin-multiplatform，但ktor依赖当前未提供linux-arm64的支持，所以选用了graalvm作为方案补充，本文将介绍如何在kotlin-multiplatform与graalvm共存，目标是尽可能共享代码。

## 问题分析

在kotlin
multiplatform中，我们可以使用kotlin编写多个平台的代码，但是在实际开发中，我们会发现有些库并不支持所有平台，比如ktor-client-curl，它并不支持linux-arm64平台，这时我们可以使用ktor-client-okhttp替代，但是在kotlin-multiplatform中，我们无法直接使用ktor-client-okhttp，因为它并不支持native平台，这时我们可以使用graalvm来解决这个问题。

另外项目当前使用的是kotlin-multiplatform，如果我们使用graalvm，graalvm的插件不支持kotlin-multiplatform，graalvm的插件当前仅对kotlin-jvm有效，所以我们需要找到一种方法，让kotlin-multiplatform与graalvm共存。

- 软链接： Linux有软链接的概念，我们可以将kotlin-multiplatform的代码软链接到kotlin-jvm中，这样我们可以在graalvm子项目中直接使用kotlin-multiplatform的代码。
  > Linux下软链接有如下特点：
  >1. 可以跨越文件系统：软链接可以指向同一系统内的不同文件系统中的文件或目录。
  >2. 指向目录：软链接可以指向目录。
  >3. 大小很小：软链接本身的文件大小很小，只占用存储路径信息的空间。
  >4. 支持相对路径：软链接可以使用相对路径来指向目标文件或目录。使用相对路径创建软链接在一些情况下非常方便，尤其是当文件或目录的相对位置不会改变时。
  >
  > windows下也有软链接的概念，与Linux类似。使用mklink命令可以创建软链接。

## 软链接教程

### Linux

在Linux下，我们可以使用ln命令来创建软链接。使用以下命令可以创建软链接：

```shell
ln -s /path/to/target /path/to/link
```

- /path/to/target：目标文件或目录的路径。
- /path/to/link：软链接的路径。
- -s：创建软链接。

### Windows

在Windows下，我们可以使用mklink命令来创建软链接，注意该命令只能在cmd环境下使用，不能在powershell下使用。使用以下命令可以创建软链接：

```shell
mklink /D path\to\link path\to\target
```

- /D：创建目录的软链接。
- path\to\link：软链接的路径。
- path\to\target：目标文件或目录的路径。

## 实现

### 创建kotlin-multiplatform项目

结构如下：

```text
.
├── build.gradle.kts
├── gradle
│   └── wrapper
├── gradle.properties
├── gradlew
├── gradlew.bat
├── settings.gradle.kts
└── src
    ├── commonMain
    ├── jvmMain
    └── nativeMain
```

build.gradle.kts文件大致如下：

```kotlin
plugins {
    kotlin("multiplatform") version "2.0.0"
}

//  ...

kotlin {
    applyDefaultHierarchyTemplate()
    linuxX64 {
        config()
    }
    mingwX64 {
        config()
    }
    jvm {
        withJava()
        val jvmJar by tasks.getting(org.gradle.jvm.tasks.Jar::class) {
            duplicatesStrategy = DuplicatesStrategy.EXCLUDE
            doFirst {
                manifest {
                    attributes["Main-Class"] = "MainKt"
                }
                from(configurations.getByName("runtimeClasspath").map { if (it.isDirectory) it else zipTree(it) })
            }
        }
    }
    sourceSets {
        val commonMain by getting {
            dependencies {
                //  ...
            }
        }

        val nativeMain by getting {
            dependencies {}
        }

        val linuxX64Main by getting {
            dependencies {
                implementation("io.ktor:ktor-client-curl:$ktor_version")
            }
        }

        val jvmMain by getting {
            dependencies {
                //  ...
            }
        }

    }
}
```

在这个项目中，我们可以在commonMain中编写通用代码，然后在nativeMain中编写原生平台的代码，jvmMain中编写jvm平台的代码。
使用 **./gradlew linuxX64Binaries**可以编译linux-x64平台的代码，使用 **./gradlew jvmJar**可以编译jvm平台的代码。

### 链接kotlin-multiplatform项目

在项目路径下创建一个kotlin-jvm文件夹，然后使用软链接将kotlin-multiplatform项目中的src目录链接到kotlin-jvm项目中。最终的目录结构如下：

```text


```
