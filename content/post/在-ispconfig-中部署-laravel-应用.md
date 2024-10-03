---
title: 在 ISPConfig 中部署 Laravel 应用
date: 2024-10-03T21:00:00+08:00
---
1. 在ispconfig后台add new website
 - Auto-Subdomain选择www.
 - 勾选Let's Encrypt SSL
 - PHP选择PHP-FPM
 - SEO Redirect选择www.domain.tld => domain.tld
 - 勾选Rewrite HTTP to HTTPS
 - nginx Directives填写
```
location / {
    root {DOCROOT}laravel/public; #laravel是项目名
    try_files $uri laravel/public/$uri/ /laravel/public/index.php?$query_string;
}
```
2. 在新建的site中查看Document Root,在jenkins对应item的configure - Build Steps - Execute shell
```
cp -r /var/lib/jenkins/workspace/laravel /var/www/clients/client0/web4/web
```
这里会报权限问题
```
# ls -ld /var/www/clients/client0/web4/web
```
输出：drwxrw-xr-x 5 web4 client0 4096 Oct  3 13:18 /var/www/clients/client0/web4/web

把用户jenkins加入client0组
```
sudo usermod -aG client0 jenkins
chmod 775 /var/www/clients/client0/web4/web
```
访问会报ERROR 403错误，可以使用命令
```
tail -f /var/www/clients/client0/web4/log/error.log
```
查出问题出在laravel目录访问权限上，接着设置laravel项目配置

#### 安装 Composer

```
apt install composer
```

#### 下载或上传 Laravel 项目

此步骤已做

#### 设置目录权限

Laravel 需要对 storage 和 bootstrap/cache 目录有写权限：

```
sudo chown -R www-data:www-data /var/www/your-laravel-project
sudo chmod -R 775 /var/www/your-laravel-project/storage
sudo chmod -R 775 /var/www/your-laravel-project/bootstrap/cache
```

#### 安装依赖

```
cd /var/www/your-laravel-project
composer install #提示Continue as root/super user [yes]? no
su - jenkins
cd /var/www/clients/client0/web4/web/laravel
composer install
```

#### 设置环境配置
复制 .env.example 并重命名为 .env，设置数据库连接、应用名称等
```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:cTwQyhnXdk7yFLmFTEDRcWukIseQ=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack
LOG_DEPRECATIONS_CHANNEL=null
LOG_LEVEL=debug

DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=laravel
DB_USERNAME=xxx
DB_PASSWORD=xxx

[...]
```
设置好数据库连接信息、应用密钥等。生成应用密钥：

```
php artisan key:generate
```

#### 配置 Nginx

#### 配置数据库

进入 MySQL：
```
CREATE DATABASE `laravel-xxx`;
```

#### 运行迁移和种子数据（可选）

如果你有数据库迁移或种子数据需要运行，可以执行：

```
php artisan migrate
php artisan db:seed
```

#### 调试和优化（可选）
为生产环境优化 Laravel：
```
php artisan config:cache
php artisan route:cache
php artisan view:cache
```
这些命令将缓存配置文件、路由和视图，提高应用的性能。

#### 清理缓存可用
```
php artisan config:clear
php artisan cache:clear
php artisan view:clear
```
