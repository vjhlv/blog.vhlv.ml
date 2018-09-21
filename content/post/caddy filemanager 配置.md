+++
title = "caddy filemanager 配置"
date = "2018-9-19"
author = "vhlv"
cover = "shell.jpg"
description = "caddy filemanager 配置"

+++

### caddy filemanager 配置



```sh
http://cloud.xxx.xxx:80 {
 timeouts none
 redir https://cloud.xxxx.xxx{url}
}
#https 站点
https://cloud.viiv.ren:443 {
 root /var/www/files
 timeouts none
 tls xxx@xxx.com
 gzip
 filemanager / /var/www/files {
 database /data/caddy/filemanager.db
 }
 #访问日志
 log /data/caddy/logs/access.log
 #错误日志
 errors log /var/www/logs/errors.log {
        rotate_size 50     #50M以后，自动分割  
        rotate_age  30     #分割文件最多保留30天  
        rotate_keep 100    #最多保留100个分割文件
        rotate_compress    #压缩日志文件的选项。gzip是唯一支持的格式。
        }

}
```