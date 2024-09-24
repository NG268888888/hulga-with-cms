---
title: ubuntu 22.04安装ispconfig3.1
date: 2024-09-24T09:03:00+08:00
---
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
cgi.fix_pathinfo=0
[...]
date.timezone="Europe/Berlin"
```

```
service php8.1-fpm reload
```

### 在 Ubuntu 22.04 上删除所有与 PHP 相关的包(仅记录)

```
dpkg -l | grep php
```

**删除 PHP 包**

```
sudo apt-get purge 'php*'
```

**清理未使用的依赖项**

```
sudo apt-get autoremove
```

**检查是否删除成功**

```
dpkg -l | grep php
```

**清理 APT 缓存（可选）**

```
sudo apt-get clean
```

### phpmyadmin

```
sudo apt-get install phpmyadmin
```

将看到以下问题：

Web server to reconfigure automatically: <-- select none (because only apache2 and lighttpd are available as options)

MySQL application password for phpmyadmin: <-- Press Enter

### Let's Encrypt

```
apt install certbot
certbot register #输入注册邮箱
```

### awstats

```
apt-get -y install awstats geoip-database libclass-dbi-mysql-perl
```

编辑/etc/cron.d/awstats，注释掉代码

### fail2ban

```
apt-get -y install fail2ban
vi /etc/fail2ban/jail.local #监测写这个文件
service fail2ban restart
```

如果没有开启 **iptables** 或 **ufw** 服务，**Fail2ban** 将失去其主要的防护功能，因为 Fail2ban 默认通过这些防火墙服务来阻止恶意 IP 地址。

`apt-get install ufw`

`service nginx restart`

### ispconfig 3.x

```
cd /tmp  
wget -O ispconfig.tar.gz https://www.ispconfig.org/downloads/ISPConfig-3-stable.tar.gz  
tar xfz ispconfig.tar.gz  
cd ispconfig3*/install/
php -q install.php
```

**如果改变默认端口，记得加进后台防火墙设置里**

### 访问phpmyadmin地址提示502 Bad Gateway解决办法

安装phpmyadmin会自动安装php8.3，等安装完ispconfig3.x后，访问 http://server1.example.com:8081/phpmyadmin 会提示502 Bad Gateway，解决步骤

1. 查看nginx错误日志

`tail -f /var/log/nginx/error.log`

> 2024/09/23 06:15:46 [error] 32284#32284: *505 connect() failed (111: Unknown > >    error) while connecting to upstream
> [...]
> upstream: "fastcgi://127.0.0.1:9000"

Nginx无法连接到fastcgi://127.0.0.1:9000

ispconfig安装之后在/etc/nginx/sites-available/apps.vhost，找到phpmyadmin那里，里面写的是localhost:9000,注释掉，改成fastcgi_pass unix:/var/lib/php8.1-fpm/apps.sock;

重启nginx

### ispconfig Jobqueue(任务队列)卡住了

`/usr/local/ispconfig/server/server.sh`

会输出错误原因，解决后任务队列就正常了

可以通过后台 **`Monitor`** > **`Show Jobqueue`** 来查看挂起的任务。

**检查系统日志**

如果以上步骤未能解决问题，可以检查相关日志文件，查看是否有进一步的错误信息：

- ISPConfig日志通常位于 `/var/log/ispconfig/ispconfig.log`。
- 检查 `/var/log/syslog` 或者 `/var/log/messages` 是否有系统级的错误。

### 安装ispconfig提示`Failed to reload php8.1-fpm.service: Unit php8.1-fpm.service not found`

安装ispconfig 3.1以上版本，如果提示`Failed to reload php8.1-fpm.service: Unit php8.1-fpm.service not found`，ubuntu 22.04默认是php8.1，ispconfig 3.X版本支持php 8.1
