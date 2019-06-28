---
title: Ubuntu上自建SSR
date: 2019-06-29 00:53:02
tags:
---

记录一下使用VPS自建ShadowSocksR的过程。



##  登陆

使用ssh连上自己的VPS。这一步有可能连不上，表现为ping得通，就是连不上去，偶尔可能可以连上，但绝大部分时候连不上，似乎只要是ssh到境外就会被阻断。这个时候就看VPS提供商的本事了，有的提供商会提供一个反向代理连VPS，只要国内能访问这个VPS提供商，就可以利用其提供的反向代理连VPS。

ssh连不上似乎并不影响SSR，因此只要我们能想办法在机器上执行指令，建立SSR服务端，就不妨碍我们的目标。



## 安装SSR

### 安装
```bash
wget https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocksR.sh
chmod +x shadowsocksR.sh
./shadowsocksR.sh 2>&1 | tee shadowsocksR.log
```

安装过程会要求选择加密算法，混淆协议，设置密码等等，按照提示输入指令即可，输出会存在`shadowsocksR.log`里。



### 卸载

```bash
./shadowsocksR.sh uninstall
```



### 相关路径

安装后，可以在如下路径寻找相关的文件：
配置文件：/etc/shadowsocks.json
里面可以更改相关的配置。

日志文件：/var/log/shadowsocksr.log
错误日志的输出，主要是fail2ban需要用到。

代码目录：/usr/local/shadowsocks
基本不会用到，但偶尔看看代码确认问题还是可以的。



### 管理指令

```bash
/etc/init.d/shadowsocks start
/etc/init.d/shadowsocks stop
/etc/init.d/shadowsocks restart
/etc/init.d/shadowsocks status
```



### 配置

加密算法建议使用chacha20-ietf-poly1305，或者aes-256-gcm

关于混淆协议，混淆参数。SSR的混淆，本质上是伪装流量，因此不正确的混淆可能会被识别出特征，建议了解一下再配置。

注意这里的配置是为了安全，保密，不是为了性能，因此不要贪图性能用过时算法或协议。



## 使用fail2ban

由于目前已知某些人会对SSR服务进行探测，以及部分人会尝试暴力破解，因此需要使用fail2ban保护VPS。不要以为这东西没用，运行几天，规则里面就有一堆封禁IP。

fail2ban原理是利用程序的日志输出，获取一些导致失败操作的IP，对其进行封禁。



### 安装fail2ban：

```bash
sudo apt-get install fail2ban
```



### 配置fail2ban规则

创建`/etc/fail2ban/filter.d/shadowsocks.conf`，内容如下：

```
[Definition]

_daemon = shadowsocks

failregex = ^\s+ERROR\s+tcprelay\.py:\d+ can not parse header when handling connection from ::ffff:<HOST>:\d+$
```

这段内容来自SSR源码里的fail2ban配置，但可能是因为太久没人维护了，原配置无法正确匹配出IP，我稍微改了一下它的正则，这个版本是可用的。

创建`/etc/fail2ban/jail.d/shadowsocks.conf`，内容如下：

```
[shadowsocks]
enabled = true
filter = shadowsocks
port = all
banaction = iptables-allports
logpath = /var/log/shadowsocksr.log
maxretry = 1
bantime = -1
findtime = 315360000
```

注意里面`maxretry`代表可以失败的次数，`bantime`填`-1`代表永久封禁，`findtime`表示检测间隔，该时间段失败次数超过`maxretry`就会触发封禁。



### fail2ban管理指令

```bash
fail2ban-client start
fail2ban-client stop
fail2ban-client status // 查看总体状态
fail2ban-client status shadowsocks // 查看shadowsocks规则的状态
fail2ban-client set shadowsocks unbanip x.x.x.x // 解除shadowsocks规则对x.x.x.x的封禁
```



## 享受自由的互联网

把SSR和fail2ban都运行起来，就可以享受自由的互联网了。