---
categories:
  - tools
title: "快速开始cloudflare ddns"
date: 2024-03-27T09:26:00+08:00
draft: false
tags:
  - ddns
  - cloudflare
  - cloudflare-ddns
---

## 介绍

本文讲述如何用最简单的方式快速开始使用cloudflare ddns。

## 准备操作

### 1. 注册cloudflare账号

本文不会详细将这个，请参考其他教程。

### 2. 获取cloudflare的api key

1. 访问https://dash.cloudflare.com/profile/api-tokens，右上角处可以切换为简体中文
2. 点击创建令牌，选择编辑区域DNS模板
3. 名称任意填写
4. 令牌权限选择编辑区域DNS，令牌的权限务必选择
    - 区域-区域-读取
    - 区域-dns-编辑
5. 区域资源，选择需要用于ddns的域名
6. 客户端IP（可选），这个用于区别调用客户端的白名单，由于本文讲解ddns，本身访问的公网IP就是动态的，所以这里最好不要填写
7. TTL，定义此令牌将保持活动状态的时间长度。这个不需要填写

![创建令牌](../cf-ddns/images/000134.png "创建令牌")

完成后点击创建令牌，并拷贝token

![创建令牌](../cf-ddns/images/000303.png "拷贝token")

### 3. 获取zone id

1. 访问https://dash.cloudflare.com/
2. 选择需要用于ddns的域名
3. 页面右下角即可看到zone id

![获取zone id](../cf-ddns/images/000532.png "获取zone id")

## 快速开始

### Linux x64

#### 基础使用方式

1. 从[下载页](https://github.com/selcarpa/cloudflare-ddns/releases/latest)下载cf-ddns-linux-x64-(version)-RELEASE.kexe
2. 将下载的文件重命名为cf-ddns，并且给予运行权限
    ```shell
        mv cf-ddns-linux-x64-(version)-RELEASE.kexe
     ```
3. 将cf-ddns.kexe放到一个目录，例如/usr/local/bin
    ```shell
    mv cf-ddns /usr/local/bin
    ```
4. 执行命令，这行命令将开始进行300秒一次的ip检查，一旦ip变化，将会更新cloudflare的域名解析
    ```shell
    cf-ddns.kexe -gen -zoneId=<替换为上文的zoneId> -authKey=<替换为上文的token> -domain=<替换为想使用的域名> -v4=<true: 开启ipv4, false: 关闭ipv4> -v6=<true: 开启ipv6, false: 关闭ipv6>
    ```

#### 配合systemd使用

1. 从[下载页](https://github.com/selcarpa/cloudflare-ddns/releases/latest)下载cf-ddns-linux-x64-(version)-RELEASE.kexe
2. 将下载的文件重命名为cf-ddns，并且给予运行权限
    ```shell
        mv cf-ddns-linux-x64-(version)-RELEASE.kexe
     ```
3. 将cf-ddns.kexe放到一个目录，例如/usr/local/bin
    ```shell
    mv cf-ddns /usr/local/bin
    ```
4. 创建服务
    1. 创建服务文件
       ```shell
        # 创建服务
        vim /etc/systemd/system/cloudflare-ddns.service
       ```
    2. 填充内容
       ```ini
       [Unit]
       Description=cloudflare-ddns
       After=network.target
       
       [Service]
       Type=simple
       ExecStart=/usr/bin/cf-ddns -gen -zoneId=<替换为上文的zoneId> -authKey=<替换为上文的token> -domain=<替换为想使用的域名> -v4=<true: 开启ipv4, false: 关闭ipv4> -v6=<true: 开启ipv6, false: 关闭ipv6>
       Restart=on-failure
       
       [Install]
       WantedBy=multi-user.target
       ```
    3. 启动服务
       ```shell
       systemctl start cloudflare-ddns
       ```
    4. 设置开机启动
       ```shell
       systemctl enable cloudflare-ddns
       ```
#### 配合cron使用

1. 从[下载页](https://github.com/selcarpa/cloudflare-ddns/releases/latest)下载cf-ddns-linux-x64-(version)-RELEASE.kexe
2. 将下载的文件重命名为cf-ddns，并且给予运行权限
    ```shell
        mv cf-ddns-linux-x64-(version)-RELEASE.kexe
     ```
3. 将cf-ddns.kexe放到一个目录，例如/usr/local/bin
    ```shell
    mv cf-ddns /usr/local/bin
    ```
4. 编辑cron
    ```shell
    crontab -e
    ```
5. 添加一行
    ```shell
    */5 * * * * /usr/local/bin/cf-ddns -gen -zoneId=<替换为上文的zoneId> -authKey=<替换为上文的token> -domain=<替换为想使用的域名> -v4=<true: 开启ipv4, false: 关闭ipv4> -v6=<true: 开启ipv6, false: 关闭ipv6>
    ```


#### docker方式

##### docker-cli

```shell
docker run -d \
    --network host \
    --name cf-ddns \
    --restart unless-stopped \
    selcarpa/cloudflare-ddns:latest \
    -gen -zoneId=<替换为上文的zoneId> -authKey=<替换为上文的token> -domain=<替换为想使用的域名> -v4=<true: 开启ipv4, false: 关闭ipv4> -v6=<true: 开启ipv6, false: 关闭ipv6>
```

##### docker-compose

```yaml
services:
  cf-ddns:
    image: selcarpa/cloudflare-ddns:latest
    network_mode: host
    container_name: cf-ddns
    restart: unless-stopped # 开机自启
    command:
      -gen -zoneId=<替换为上文的zoneId> -authKey=<替换为上文的token> -domain=<替换为想使用的域名> -v4=<true:
        开启ipv4, false:
          关闭ipv4> -v6=<true:
            开启ipv6, false: 关闭ipv6>
```