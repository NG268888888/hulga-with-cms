---
title: Github代码仓库、ispconfig部署laravel项目，Drone实现CI/CD
date: 2025-02-23T09:51:00+08:00
---
# Drone部署流程

### ubuntu 22安装docker

```
 curl -fsSL https://get.docker.com -o get-docker.sh
 sudo sh get-docker.sh
```

### Server Installation with Github

https://docs.drone.io/server/provider/github

step1结束之后，

### Create a Shared Secret

在Drone server的服务器运行

```
openssl rand -hex 16
```

**Server和ISPConfig安装到同一个VPS上**，但需要注意以下潜在冲突和配置要点：

1. **端口占用**
  
  - ISPConfig占用端口：8080（管理面板）、80/443（Web服务）
  - Drone默认使用80/443或自定义端口（如:3000）
  - 解决方案：为Drone配置非标准端口或通过子域名反向代理

```
docker run \
 --volume=/var/lib/drone:/data \
 --env=DRONE_GITHUB_CLIENT_ID=your-id \
 --env=DRONE_GITHUB_CLIENT_SECRET=super-duper-secret \
 --env=DRONE_RPC_SECRET=super-duper-secret \
 --env=DRONE_SERVER_HOST=drone.company.com \
 --env=DRONE_SERVER_PROTO=http \
 --publish=3000:80 \
 --restart=always \
 --detach=true \
 --name=drone \
 drone/drone:2
```

“--publish=3000:80 \“ 3000是Drone server端口

可能用到的命令

```
docker ps -a
docker rm 3sd341
```

**反向代理配置（Nginx示例）** 略过

```
# ISPConfig主站
server {
    listen 80;
    server_name yourdomain.com;
    # ... ISPConfig原有配置
}

# Drone服务
server {
    listen 80;
    server_name ci.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        include proxy_params;
    }
}
```

2. **防火墙规则**

```
ufw allow 80/tcp
ufw allow 443/tcp
```

### Drone服务的HTTPS配置

#### 方法1：反向代理模式（推荐）

在ispconfig的Site选项卡创建刚才在docker配置中填的域名“drone.company.com”

vi /etc/nginx/sites-available/drone.company.com.vhost

```
# 在ISPConfig创建的Nginx站点配置中修改
server {
  ...
    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Host $http_host;
    }
}
```

sudo systemctl reload nginx

然后登录drone.company.com尽快完成注册

#### 方法2：Drone原生HTTPS（Docker部署）

https://docs.drone.io/server/https 未测试

#### 2. 安装 Drone Runner（SSH Runner）

```
docker run --detach \
  --env=DRONE_RPC_PROTO=https \
  --env=DRONE_RPC_HOST=drone.company.com \
  --env=DRONE_RPC_SECRET=super-duper-secret \
  --publish=4000:4000 \
  --restart always \
  --name runner \
  drone/drone-runner-ssh
```

“--publish=4000:4000 \“ 4000不要和Server的端口相同

### 三、项目配置步骤

#### 1. 在 Drone 中激活仓库

- 登录 Drone Web 界面
- 找到你的 GitHub 仓库并启用

> 激活以后，检查一下 GIthub 仓库的 Webhooks

![Drone CI For Github —— 打造自己的CI/CD工作流（一）](https://cdn.learnku.com/uploads/images/202010/01/22816/vNXWrhj1sB.webp!large)

**生成SSH密钥对**：
在部署Drone Server的VPS上生成一个SSH密钥对（如果还没有）：

```
ssh-keygen -t rsa -b 4096 -C "drone-deploy-key" -f ~/.ssh/drone_deploy_key
```

**将公钥添加到服务器B**：
将生成的公钥添加到该服务器的`~/.ssh/authorized_keys`中。

**将私钥配置到Drone**：
将生成的私钥文件（`drone_deploy_key`）添加到Drone Web中，并将其作为一个安全变量配置到工作流中。

![截屏20250220 221925png](file:///Users/logon/Desktop/截屏2025-02-20%2022.19.25.png?msec=1740061192350)

#### 2. 配置仓库 Secrets

在 Drone 仓库设置中添加：

- `host` - 目标服务器 IP
- `user` - SSH 用户名（如 root）
- `ssh_key` - 服务器 SSH 私钥（带 BEGIN OPENSSH PRIVATE KEY 格式）
- `web_root` - 网站目录（如 /var/www/clients/client1/web1/web）

### 四、创建 `.drone.yml` 配置文件

在激活的github项目中添加.drone.yml，每次提交到github，都会自动

```
kind: pipeline
type: ssh
name: deploy

server:
  host: host
  user: user          # SSH 登录用户名
  ssh_key: 
    from_secret: ssh_key      # 存储私钥的 secret 名称

steps:
...
```

### 五、在用的.drone.yml

```
kind: pipeline
type: ssh
name: deploy

server:
  host:
    from_secret: deploy_host # Drone后台项目Settings->Secrets
  user:
    from_secret: deploy_user
  ssh_key:
    from_secret: ssh_key

steps:
- name: 部署代码
  commands:
    - git config --global --add safe.directory /var/www/clients/client0/web2/web/project-name
    - cd /var/www/clients/client0/web2/web/project-name
    - git pull origin # Drone server、ssh runner部署在服务器B，把部署项目（服务器A）的root pub key添加到Github

- name: fix-permissions
  commands:
    - chown -R web2:client0 /var/www/clients/client0/web2/web
    # - find /var/www/clients/client0/web2/web/evoai -type d -exec chmod 755 {} \;
    # - find /var/www/clients/client0/web2/web/evoai -type f -exec chmod 644 {} \;
    - chmod -R 775 /var/www/clients/client0/web2/web/project-name/storage
    - chmod -R 775 /var/www/clients/client0/web2/web/project-name/bootstrap/cache  

- name: Cache Optimization
  commands:
    - cd /var/www/clients/client0/web2/web/project-name
    - php artisan config:cache
    - php artisan route:cache
    - php artisan view:cache

trigger:
  branch:
    - main
  event:
    - push
```
