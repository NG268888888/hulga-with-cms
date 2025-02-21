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

```
ssh-keygen -t ed25519 #生成pub key粘贴到github项目库settings的ssh key
cd /var/www/clients/client0/webX/web
git clone git项目
chown -R webX:client0 your-laravel-project
chmod 775 your-laravel-project
chmod -R 775 your-laravel-project/storage
chmod -R 775 your-laravel-project/bootstrap/cache

sudo adduser deployer --disabled-password
sudo usermod -aG client0 deployer

chmod 664 your-laravel-project/composer.lock
su - deployer
apt install composer
cd /var/www/clients/client0/webX/web/your-laravel-project

composer update
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
php artisan storage:link
```

#### 运行迁移和种子数据（可选）

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

由于后台使用了laravel-admin，访问domain.com/admin会提示您与网站连接并不完全安全，需要修改，laravel项目config - admin.php中的    'https' => env('ADMIN_HTTPS', false),改为true

**https://ruben-vilar.medium.com/how-to-deploy-a-laravel-application-in-ispconfig-ffa42ba24d93**
**https://angeloavv.medium.com/how-to-deploy-a-laravel-application-into-an-ispconfig-server-using-gitlab-pipelines-62bb0fc0285e**


