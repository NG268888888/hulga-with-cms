---
title: ispconfig一键安装脚本
date: 2024-12-21T14:54:00+08:00
---
# 配置环境
```
sudo vi /etc/hosts

```
ip   server1.example.com     server1

```
sudo vi /etc/hostname
```
server1
reboot

sh运行在ubuntu 24上，默认使用php8.3-fpm

```
#!/bin/bash

set -e  # 遇到错误时退出脚本
apt-get update

# 检查并禁用 AppArmor
echo "Checking and disabling AppArmor..."
if systemctl is-active --quiet apparmor; then
    service apparmor stop
    update-rc.d -f apparmor remove
    apt-get remove -y apparmor apparmor-utils
else
    echo "AppArmor is already disabled."
fi

# 安装 MySQL
echo "Checking and installing MySQL..."
if ! command -v mysql >/dev/null; then
    apt-get install -y mysql-client mysql-server sudo
    echo "MySQL installed. You can run 'sudo mysql_secure_installation' later to secure the installation."
else
    echo "MySQL is already installed."
fi

# 安装 Nginx
echo "Checking and installing Nginx..."
if ! command -v nginx >/dev/null; then
    apt-get install -y nginx
    service apache2 stop || true
    update-rc.d -f apache2 remove || true
    service nginx start
else
    echo "Nginx is already installed."
fi

# 安装 PHP 8.3
echo "Checking and installing PHP 8.3..."
if ! php -v | grep -q "PHP 8.3"; then
    sudo add-apt-repository -y ppa:ondrej/php
    sudo apt-get update
    apt-get install php8.3 php8.3-cli php8.3-cgi php8.3-fpm php8.3-gd php8.3-mysql php8.3-imap php8.3-curl php8.3-intl php8.3-pspell php8.3-sqlite3 php8.3-tidy php8.3-xsl php8.3-zip php8.3-mbstring php8.3-soap php8.3-opcache php8.3-common php8.3-readline php8.3-xml

else
    echo "PHP 8.3 is already installed."
fi

# 配置 PHP
echo "Configuring PHP..."
PHP_INI="/etc/php/8.3/fpm/php.ini"
if ! grep -q "cgi.fix_pathinfo=0" "$PHP_INI"; then
    sed -i 's/^cgi.fix_pathinfo=.*/cgi.fix_pathinfo=0/' "$PHP_INI"
    sed -i 's/^;date.timezone =.*/date.timezone = "Europe\/Berlin"/' "$PHP_INI"
    service php8.3-fpm reload
else
    echo "PHP is already configured."
fi

# 安装 Let's Encrypt
echo "Checking and installing Let's Encrypt..."
if ! command -v certbot >/dev/null; then
    apt-get install -y certbot
    certbot register --agree-tos --no-eff-email
else
    echo "Let's Encrypt is already installed."
fi

# 安装 PureFTPd
echo "Checking and installing PureFTPd..."
if ! dpkg -l | grep -q pure-ftpd-mysql; then
    apt-get install -y pure-ftpd-common pure-ftpd-mysql
else
    echo "PureFTPd is already installed."
fi

# 配置 PureFTPd
echo "Configuring PureFTPd..."
PUREFTPD_CONF="/etc/default/pure-ftpd-common"
if grep -q "^STANDALONE_OR_INETD=" "$PUREFTPD_CONF" && grep -q "^VIRTUALCHROOT=" "$PUREFTPD_CONF"; then
    echo "PureFTPd configuration already set."
else
    sed -i 's/^STANDALONE_OR_INETD=.*/STANDALONE_OR_INETD=standalone/' "$PUREFTPD_CONF"
    sed -i 's/^VIRTUALCHROOT=.*/VIRTUALCHROOT=true/' "$PUREFTPD_CONF"
fi

# 配置 TLS 支持
TLS_FILE="/etc/pure-ftpd/conf/TLS"
if [ ! -f "$TLS_FILE" ] || [ "$(cat $TLS_FILE)" != "1" ]; then
    echo "Enabling FTP and TLS sessions..."
    echo 1 > "$TLS_FILE"
else
    echo "TLS support is already enabled."
fi

# 创建 SSL 证书
SSL_DIR="/etc/ssl/private/"
SSL_FILE="$SSL_DIR/pure-ftpd.pem"
if [ ! -f "$SSL_FILE" ]; then
    echo "Creating SSL certificate for PureFTPd..."
    mkdir -p "$SSL_DIR"
    openssl req -x509 -nodes -days 7300 -newkey rsa:2048 \
        -keyout "$SSL_FILE" -out "$SSL_FILE" \
        -subj "/C=DE/ST=YourState/L=YourCity/O=YourOrganization/OU=IT Department/CN=server1.example.com/emailAddress=youremail@example.com"
    chmod 600 "$SSL_FILE"
else
    echo "SSL certificate for PureFTPd already exists."
fi

# 重启 PureFTPd 服务
echo "Restarting PureFTPd service..."
service pure-ftpd-mysql restart

echo "PureFTPd and TLS configuration completed."

# 重启 PureFTPd 服务
echo "Restarting PureFTPd service..."
service pure-ftpd-mysql restart



# 安装 AWStats
echo "Checking and installing AWStats..."
if ! dpkg -l | grep -q awstats; then
    apt-get install -y awstats geoip-database libclass-dbi-mysql-perl
    sed -i 's/^/#/' /etc/cron.d/awstats
else
    echo "AWStats is already installed."
fi

# 安装 Fail2ban
echo "Checking and installing Fail2ban..."
if ! dpkg -l | grep -q fail2ban; then
    apt-get install -y fail2ban
    cat <<EOF > /etc/fail2ban/jail.local
[DEFAULT]
bantime = 3600
findtime = 600
maxretry = 5

[nginx-http-auth]
enabled = true
EOF
    service fail2ban restart
else
    echo "Fail2ban is already installed."
fi

# 安装 ISPConfig 3.x
echo "Checking and installing ISPConfig..."
if [ ! -d "/tmp/ispconfig3" ]; then
    cd /tmp
    wget -O ispconfig.tar.gz https://www.ispconfig.org/downloads/ISPConfig-3-stable.tar.gz
    tar xfz ispconfig.tar.gz
    cd ispconfig3*/install/
    php -q install.php
else
    echo "ISPConfig is already installed."
fi

# 重启所有服务
echo "Restarting Nginx and PHP-FPM..."
service nginx restart
service php8.3-fpm restart

echo "Installation complete. Please verify all components."

```

# Adminer 安装和配置脚本

```
# 安装 Adminer
echo "Checking and installing Adminer..."
ADMINER_FILE="/var/www/apps/adminer.php"
if [ ! -f "$ADMINER_FILE" ]; then
    echo "Downloading Adminer..."
    wget -q https://github.com/vrana/adminer/releases/download/v4.8.1/adminer-4.8.1-mysql-en.php -O "$ADMINER_FILE"
    echo "Adminer installed at $ADMINER_FILE."
else
    echo "Adminer is already installed."
fi

# 配置 Nginx 以支持 Adminer
echo "Configuring Nginx for Adminer..."
NGINX_CONF="/etc/nginx/sites-available/adminer"
if [ ! -f "$NGINX_CONF" ]; then
    mkdir -p /var/www/apps
    cat <<EOF > "$NGINX_CONF"
server {
    listen 8081;
    server_name _;

    root /var/www/apps;

    location /adminer {
        index adminer.php;
        try_files \$uri \$uri/ /adminer.php?\$query_string;
    }

    location ~ \.php\$ {
        include fastcgi_params;
        fastcgi_param SCRIPT_FILENAME \$document_root\$fastcgi_script_name;
        fastcgi_pass unix:/var/run/php/php8.3-fpm.sock;
    }
}
EOF

    echo "Creating symbolic link to enable Nginx site configuration..."
    ln -sf "$NGINX_CONF" /etc/nginx/sites-enabled/adminer

    echo "Testing Nginx configuration..."
    if nginx -t; then
        echo "Reloading Nginx..."
        service nginx reload
        echo "Adminer Nginx configuration applied."
    else
        echo "Nginx configuration test failed. Please check the configuration file: $NGINX_CONF."
        exit 1
    fi
else
    echo "Nginx configuration for Adminer is already in place."
fi

echo "Adminer installation and Nginx configuration completed. Access Adminer at http://<your-ip>:8081/adminer."
```

记得把   server_name _;改成    server_name yourdomain;
