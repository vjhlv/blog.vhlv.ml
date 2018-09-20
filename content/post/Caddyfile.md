+++
title = "Caddyfile"
date = "2018年9月20日"
author = "vhlv"
cover = "shell.png"
description = "Caddy config file"

+++
```sh
http://axiong.me {
    redir https://axiong.me
}
https://axiong.me {
    #tls off
    #tls admin@example.com
    tls /etc/ssl/caddy/certs/axiong.me/fullchain.cer /etc/ssl/caddy/certs/axiong.me/ssl.key
    minify
    gzip
    log / /var/log/caddy/pub-axiong.me_access.log "{combined}" {
        rotate_size 100 # Rotate a log when it reaches 100 MB
        rotate_age  14  # Keep rotated log files for 14 days
        rotate_keep 10  # Keep at most 10 rotated log files
        rotate_compress # Compress rotated log files in gzip format
    }
    errors /var/log/caddy/pub-axiong.me_error.log {
        404 404.html # Not Found
        rotate_size 100 # Rotate a log when it reaches 100 MB
        rotate_age  14  # Keep rotated log files for 14 days
        rotate_keep 10  # Keep at most 10 rotated log files
        rotate_compress # Compress rotated log files in gzip format
    }
    root /var/www/axiong.me/public
    git {
        repo https://github.com/nickfan/axiong.me
        path /var/www/axiong.me
        then hugo --destination=/var/www/axiong.me/public
        hook /webhook [你在github后台设置的webhook的口令]
        hook_type github
        clone_args --recursive
        pull_args --recurse-submodules
        interval 3600
    }
    hugo
}
```