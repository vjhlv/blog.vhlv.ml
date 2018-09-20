+++
title = "caddy 安装配置"
date = "2018-9-20"
author = "vhlv"
cover = "shell.png"
description = "生产环境配置&使用"

+++

### 1、Install

Direct link to download: 

```bash
wget https://getcaddy.com -O getcaddy
chmod +x getcaddy
```

然后根据需求可选安装插件，比如我安装了http.cache, http.expires, http.ipfilter, http.minify等插件：

```bash
./getcaddy personal http.cache,http.expires,http.ipfilter,http.minify,http.nobots,http.ratelimit
```



## 生产环境配置&使用

安装完成后需要进行一些配置方便在生产环境使用。

先设置相关权限。

```sh
chown root:root /usr/local/bin/caddy
chmod 755 /usr/local/bin/caddy
setcap 'cap_net_bind_service=+eip' /usr/local/bin/caddy
```

`setcap 'cap_net_bind_service=+eip'` 是使服务程序运行在非root帐户下时，也能够banding到低端口。

下面新建配置文件目录，并新建一个 Caddyfile 文件，并写入一条 import 指令。

```sh
mkdir -p /etc/caddy
chown -R root:www-data /etc/caddy
mkdir -p /etc/ssl/caddy
chown -R www-data:root /etc/ssl/caddy
chmod 770 /etc/ssl/caddy
touch /etc/caddy/Caddyfile
echo 'import ./vhosts/*' > /etc/caddy/Caddyfile
```

然后创建网站文件的存放目录。

```bash
mkdir -p /var/www
chown www-data:www-data /var/www
chmod 755 /var/www
```

创建好上面的文件和目录之后，我们还需要把 caddy 配置成一个服务，方便管理和开机自动运行。

```bash
curl -s https://raw.githubusercontent.com/mholt/caddy/master/dist/init/linux-systemd/caddy.service -o /etc/systemd/system/caddy.service
# 从 github 下载 systemd 配置文件
chown root:root /etc/systemd/system/caddy.service   
# 配置权限
chmod 744 /etc/systemd/system/caddy.service
systemctl daemon-reload         
#重新加载 systemd 配置
systemctl enable caddy.service  
# 设置 caddy 服务自启动
systemctl start caddy.service   
# 启动 caddy 状态
systemctl status caddy.service
# 查看caddy 启动状态

```

## Caddyfile

Caddyfile 是 Caddy 的配置文件, 在上面我写入了一条 `import ./vhosts/*` 到Caddyfile中，表示 `/etc/caddy/vhosts`下的所有文件都会导入到 Caddy 的配置文件中，这样有多个网站的时候比较方便管理。 下面我在/etc/caddy/vhosts创建了一个 liuzhichao.com 的文件，配置如下：

```bash
www.liuzhichao.com {
    redir https://liuzhichao.com{uri}
}
liuzhichao.com {
        root /var/www/liuzhichao.com
        log / /var/log/caddy/liuzhichao.log "{remote} {when} {method} {uri} {proto} {status} {size} {>User-Agent} {latency}"
        tls mail@liuzhichao.com
        header / Strict-Transport-Security "max-age=31536000"
        gzip
        errors {
         404 404.html
         403 403.html
       }
       expires {
         match .css$ 1m
         match .js$ 1m
         match .png$ 1m
         match .jpg$ 1m
      }
      ipfilter / {
        rule  block
        blockpage /var/www/liuzhichao.com/403.html
        ip 148.251.8.250 136.243.37.219 144.76.38.40 69.197.177.50 199.58.86.211 5.9.97.200 144.76.91.79
      }
     rewrite {
        if {>User-agent} has "MJ12bot"
        to /forbidden
    }
     status 403 /forbidden
}
```

一般你只需要根据自己的域名和路径做下简单的修改即可。 配置说明如下：

```
www.liuzhichao.com {
    redir https://liuzhichao.com{uri}
}
```

是将 www 跳转到非 www 的域名。

```bash
tls mail@liuzhichao.com
```

tls后面改为你的邮箱地址，会自动配置 https。

```bash
header / Strict-Transport-Security "max-age=31536000"
```

是一条 https 的优化配置，加上之后，在[SSLLabs](https://www.ssllabs.com/ssltest/analyze.html?d=liuzhichao.com&hideResults=on)上测试评分可以拿到A+,想想之前使用 Nginx 的时候，网络上找了各种配置参考都只优化到了 A，所以 Caddy 的自动 Https 功能确实还是很方便的。

```bash
errors {
         404 404.html
         403 403.html
     }
```

是自定义错误页面配置。确保你网站的根目录有相应的文件，不然启动服务会报错。

```bash
 expires {
         match .css$ 1m
         match .js$ 1m
         match .png$ 1m
         match .jpg$ 1m
      }
```

expires 是控制页面的缓存，上面的配置是将 css,js,png,jpg 这样的静态资源缓存1个月。此配置依赖http.expires这个插件，如果你没有安装，配置后启动 caddy 会出错。

```bash
ipfilter / {
        rule  block
        blockpage /var/www/liuzhichao.com/403.html
        ip 148.251.8.250 136.243.37.219 144.76.38.40 69.197.177.50 199.58.86.211 5.9.97.200 144.76.91.79
     }
```

ipfilter是根据配置过滤到一些非正常的 IP，可以查看访问log，经常会有一些爬虫频繁的访问网站，没有任何用处反而加大服务器的负载，对于这样的 IP 可以直接过滤掉。blockpage是配置这些 IP 访问网址时显示的页面，依赖http.ipfilter插件。

```bash
 rewrite {
        if {>User-agent} has "MJ12bot"
        to /forbidden
    }
    status 403 /forbidden
```

与上面的ipfilter功能类似，都是过滤掉一些非正常的访问用户，不同的是ipfilter是屏蔽 IP，这段配置则是根据`User-agent` block掉一些爬虫。

然后将生成好的网站文件上传到配置文件中root 后面配置的目录即可。比如我是`/var/www/liuzhichao.com/`.

如果还有另外的网站，只需要在`/etc/caddy/vhosts`再新建一个配置文件重启下 caddy 服务即可。

