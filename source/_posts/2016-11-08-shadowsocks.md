---
layout: post
title: 配置Shadowsocks
tags: 技巧
date: 2016-11-08 22:56:50
---

### 配置VPS

<!-- more -->

1. 创建完实例后，打开终端进入到私钥所在文件夹。

2. 更改文件权限

    chmod 400 key.pem

3. 连接服务器：

   ssh -i key.pem  ubuntu@服务器IP

4. 获取管理员权限

   sudo -s

5. 更新

   apt-get update

6. 安装

   python-pip

7. 安装关键程序

   pip install shadowsocks

8. 配置

    vi /etc/shadowsocks.json

     单用户内容如下：
      {
      “server”:”0.0.0.0″,
      “server_port”:11383,  // 端口号为8989或11383
      “local_address”:”127.0.0.1″,
      “local_port”:1080,
      “password”:”password”,  // 密码可自行填写
      “timeout”:300,
      “method”:”aes-256-cfb”,
      “fast_open”: false
      }

     或多用户如下：

     {
     “server”:”0.0.0.0″,
     “local_address”:”127.0.0.1″,
     “local_port”:1080,
     “port_password”:{
     “8989”:”elors1270″,
     “11383”:”elors1270″
     },
     “timeout”:300,
     “method”:”aes-256-cfb”,
     “fast_open”: false
     }

9. 启动 shadowsocks:

    ssserver -c /etc/shadowsocks.json -d start

10. 添加开机启动项，即使意外重启也会自动启动shadowsocks

   vi /etc/rc.local
   将第9步的命令完整复制到文件的第二行，保存并退出.

至此，VPS的配置完毕，之后可以使用SS客户端来实现科学上网。

***


### 配置VPN

1. 连接服务器并获取管理员权限。

2. 安装pptp service

    apt-get update

    aptitude install pptpd

3. 编辑pptp配置文件

    sudo vim /etc/pptpd.conf

    localip 192.168.240.1

    remoteip 192.168.240.2-9

4. 使用Google Public DNS

    sudo vim /etc/ppp/pptpd-options

    去掉 `ms-dns` 的注释，并作修改如下

    ms-dns 8.8.8.8

    ms-dns 8.8.4.4

5. 配置访问VPN的用户名和密码。可以重复添加，分配给不同的用户。

    echo "USERNAME pptpd PASSWORD \*" | sudo tee -a /etc/ppp/chap-secrets

6. 重启服务

    sudo /etc/init.d/pptpd restart

7. 配置数据转发

    sudo vim /etc/sysctl.conf

    去掉 `net.ipv4.ip_forward=1` 行注释

    sudo sysctl -p

8. 网络地址转换

    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

9. 确保服务器重启后服务可用

    sudo vim /etc/rc.local 在 `exit 0` 上面加一行

    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

10. 进入ec2控制台，修改安全选项，在 `inbound` 和 `outbound` 里面都加入以下内容

   All TCP    TCP    0-65535    0.0.0.0/0

   All UDP    UDP    0-65535    0.0.0.0/0

   All ICMP    All    N/A     0.0.0.0/0

11. 在客户端主机上建立VPN连接，填入虚拟机公网dns地址，用户名，密码，设置vpn的dns。
