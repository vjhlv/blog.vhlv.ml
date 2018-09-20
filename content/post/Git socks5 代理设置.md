+++
title = "Git socks5 代理设置"
date = "2018年9月19日"
author = "vhlv"
cover = "git.p"
description = "首先确认ip和端口，例如ip为127.0.0.1, 端口为1080, 打开终端，运行以下命令："

+++

### Git socks5 代理设置

首先确认ip和端口，例如ip为127.0.0.1, 端口为1080, 打开终端，运行以下命令：

```shell
git config --global http.proxy 'socks5://127.0.0.1:1080'
git config --global https.proxy 'socks5://127.0.0.1:1080'
```

查看：cat ~/.gitconfig

发现是多了这两项配置

```json
[http]
proxy = socks5://127.0.0.1:1080
[https]
proxy = socks5://127.0.0.1:1080
```