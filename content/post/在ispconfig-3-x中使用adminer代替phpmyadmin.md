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
或者在/etc/nginx/sites-enabled/000-apps.vhost添加
```
    location /adminer {
        index adminer.php;  # 这里设置默认访问 adminer.php
        try_files $uri $uri/ /adminer.php?$query_string;
    }
```
已经安装好mysql，账号密码在/etc/mysql/debian.cnf中
