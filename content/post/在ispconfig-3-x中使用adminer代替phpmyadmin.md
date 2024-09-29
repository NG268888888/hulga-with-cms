---
title: 在ispconfig 3.x中使用adminer代替phpmyadmin
date: 2024-09-29T11:04:00+08:00
---
命令安装phpmyadmin会自动安装php8.3和扩展，所以用adminer代替。

```
wget https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1-mysql-en.php
mv ./adminer-4.8.1-mysql-en.php /var/www/apps/adminer.php
```

配置nginx以通过http://ip:port/adminer访问数据库

/etc/nginx/sites-available/adminer
```
server {
    listen 8081;
    server_name _;

    root /var/www/apps;

    location /adminer {
        index adminer.php;  # 这里设置默认访问 adminer.php
        try_files $uri $uri/ /adminer.php?$query_string;
    }

    location ~ \.php$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_pass unix:/var/lib/php8.1-fpm/apps.sock;
    }
}
```
### 启用 Nginx 配置

在 /etc/nginx/sites-available 目录下创建了 adminer 配置文件后，创建一个符号链接以启用它：

`sudo ln -s /etc/nginx/sites-available/adminer /etc/nginx/sites-enabled/`

### 测试配置并重新加载Nginx

首先测试Nginx配置文件是否正确，如果有错，会提示具体位置

`sudo nginx -t`

如果一切正常，重新加载Nginx：

`service nginx reload`

或者在/etc/nginx/sites-enabled/000-apps.vhost添加
```
    location /adminer {
        index adminer.php;  # 这里设置默认访问 adminer.php
        try_files $uri $uri/ /adminer.php?$query_string;
    }
```
已经安装好mysql，账号密码在/etc/mysql/debian.cnf中

### 可能遇到的错误和解决办法

> 2024/09/29 02:06:04 [error] 198783#198783: *7 open() "/var/www/apps/adminer" failed (2: No such file or directory)

默认访问都是yourdomain/adminer.php，你访问的是yourdomain/adminer，如果nginx配置有错误，会有403，nginx错误日志在/var/log/nginx/error.log
