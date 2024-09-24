---
title: ubuntu 22.04安装ispconfig3.1
date: 2024-09-24T09:03:00+08:00
---
通过ip:端口访问无需修改vps hostname

### Disable AppArmor

AppArmor 是一个安全扩展（类似于 SELinux），应该提供扩展的安全性。我们将交叉检查是否已安装，并在必要时将其删除。在我看来，你不需要它来配置一个安全的系统，而且它通常会导致更多的问题而不是优点（在你完成一周的故障排除后想想它，因为某些服务没有按预期工作，然后你发现一切正常，只有 AppArmor 导致了问题）。因此，我禁用它（如果您想稍后安装 ISPConfig，这是必须的）。

```
service apparmor stop  
update-rc.d -f apparmor remove  
apt-get remove apparmor apparmor-utils
```

### 安装mysql

```
apt-get install mysql-client mysql-server sudo
```

从 MySQL 5.7 开始，MySQL 自动为 root 用户使用 `auth_socket` 插件进行身份验证，而不再使用传统的密码验证。这意味着 root 用户可以通过系统的 socket 连接（例如通过 `sudo`）自动验证，而无需密码。

```
sudo mysql_secure_installation #mysql设置，不需要运行此命令
```

### Nginx

```
apt-get install nginx
service apache2 stop
update-rc.d -f apache2 remove
service nginx start
```

### PHP8.1

我们可以通过 [PHP-FPM](http://php-fpm.org/) 让 PHP 8 在 nginx 中工作（PHP-FPM（FastCGI Process Manager）是一种替代 PHP FastCGI 实现，具有一些对任何网站有用的附加功能大小，尤其是繁忙的站点），我们安装如下：

```
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
apt-get install php8.1 php8.1-cli php8.1-cgi php8.1-fpm php8.1-gd php8.1-mysql php8.1-imap php8.1-curl php8.1-intl php8.1-pspell php8.1-sqlite3 php8.1-tidy php8.1-xsl php8.1-zip php8.1-mbstring php8.1-soap php8.1-opcache libonig5 php8.1-common php8.1-readline php8.1-xml
```

编辑/etc/php/8.1/fpm/php.ini

```
... and set cgi.fix_pathinfo=0 and your timezone:

[...]
cgi.fix_pathinfo=**0**
[...]
date.timezone=**"Europe/Berlin"**
```

```
service php8.1-fpm reload
```

