---
title: Drone相关
date: 2025-02-23T09:56:00+08:00
---
### Drone Runner 的三大核心特性

1. **任务执行器**
  
  - 真正运行 `.drone.yml` 中定义的 pipeline 步骤
  - 处理：代码拉取、依赖安装、测试运行、部署操作等
2. **环境隔离器**
  
  - 通过不同 Runner 类型实现环境隔离：
    - Docker Runner：在容器中执行任务
    - SSH Runner：通过 SSH 在远程服务器执行
    - Exec Runner：直接在主机执行
3. **资源调度器**
  
  - 自动分配构建任务到可用 Runner
  - 支持并行执行多个 pipeline

### 常见 Runner 类型对比

| 类型  | 适用场景 | 用户案例 | 特点  |
| --- | --- | --- | --- |
| **Docker Runner** | 需要环境隔离的构建任务 | 前端项目构建、Python测试 | 安全隔离，但需要 Docker 环境 |
| **SSH Runner** | 直接操作服务器（如你的案例） | 服务器部署、运维脚本执行 | 直接访问服务器，需配置 SSH 密钥 |
| **Kubernetes Runner** | 云原生环境 | 微服务部署到 K8s 集群 | 动态创建 Pod，资源弹性扩展 |
| **Exec Runner** | 简单本地任务 | 本地开发环境测试 | 无隔离，直接使用主机环境 |

---

### 最佳实践建议

1. **专用部署用户**：建议在 VPS 创建 `deployer` 专用账户（而非 root）
  
  ```
  adduser deployer
  usermod -aG www-data deployer
  ```
  
2. **权限控制**：通过 `sudoers` 文件限制部署账户权限
  
  ```
  # /etc/sudoers.d/deployer
  deployer ALL=(www-data) NOPASSWD: /usr/bin/composer, /usr/bin/npm
  ```
  
3. **日志监控**：查看 Runner 执行日志的命令
  
  ```
  docker logs -f drone-runner
  ```
  
4. **安全隔离**：建议为不同项目使用不同的 SSH 密钥对
  

### 常见问题排查

当 Runner 不工作时，可按以下顺序检查：

1. 网络连通性：`telnet <server-ip> 22`
2. 密钥验证：`ssh -i key.pem user@host`
3. 目录权限：`ls -ld /var/www/...`
4. 环境变量：确认 `.env` 文件存在且包含必要配置
5. Drone 日志：Web 界面查看 pipeline 详细错误信息

需要根据你的实际环境调整 Runner 配置，特别是注意保持 Drone Server 和 Runner 的版本兼容性。

### 密钥准备步骤

#### 1. 生成专用密钥对（在本地）

```
ssh-keygen -t ed25519 -f drone-deploy-key -C "drone-ci"
ssh-keygen -t ed25519 -C "drone-ci-$(date +%Y%m%d)"
# 生成两个文件：
# - drone-deploy-key    → 私钥（存到Drone）
# - drone-deploy-key.pub → 公钥（存到服务器）
```

#### 2. 服务器端配置（VPS A）

```
# 将公钥添加到授权列表
mkdir -p ~/.ssh
cat drone-deploy-key.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys

# 验证权限（重要！）
chown $USER:$USER ~/.ssh -R
```

### 三、密钥准备步骤

#### 1. 生成专用密钥对（在本地）

BASH

`ssh-keygen -t ed25519 -f drone-deploy-key -C "drone-ci" # 生成两个文件： # - drone-deploy-key → 私钥（存到Drone） # - drone-deploy-key.pub → 公钥（存到服务器）`

#### 2. 服务器端配置（VPS A）

BASH

`# 将公钥添加到授权列表 mkdir -p ~/.ssh cat drone-deploy-key.pub >> ~/.ssh/authorized_keys chmod 600 ~/.ssh/authorized_keys # 验证权限（重要！） chown $USER:$USER ~/.ssh -R`

---

### 四、安全最佳实践

1. **专用密钥原则**
  
  - 不要复用已有密钥
  - 每个项目/环境使用独立密钥
2. **权限最小化**
  

```
# 在目标服务器限制密钥权限
# /etc/ssh/sshd_config
Match User deploy-user
   AllowAgentForwarding no
   PermitTTY no
   ForceCommand /usr/local/bin/deploy-script
```

**密钥轮换机制**

- 每 90 天更换一次密钥
- 旧密钥需从服务器 authorized_keys 中移除

#### 快速测试方法（不通过 Drone）：

```
# 使用本地私钥测试连接
ssh -i drone-deploy-key user@vps-ip
```

### 进阶技巧

**多环境管理**：通过前缀区分密钥

```
# 生产环境
- name: ssh_key_prod
# 测试环境  
- name: ssh_key_stage
```

**审计日志**：

```
# 在服务器查看 SSH 登录记录
sudo grep "sshd" /var/log/auth.log | grep "Accepted"
```

#### SELinux 问题检测

```
# 查看安全日志
sudo ausearch -m avc -ts recent

# 临时禁用 SELinux 测试
sudo setenforce 0
```

#### 版本兼容性检查

运行以下命令查看 Drone 版本：

```
docker inspect drone-server | grep "DRONE_VERSION"
```

版本适配指南：

| Drone 版本 | 配置语法要求 |
| --- | --- |
| < 2.0 | 需要使用 `settings` 替代 `server` |
| ≥ 2.0 | 必须使用 `server` 字段 |

### 配置示例

```
kind: pipeline
type: ssh   # 明确指定使用 SSH runner

server:
  host:
    from_secret: DEPLOY_HOST  # 从 secrets 注入
  user:
    from_secret: DEPLOY_USER  # 从 secrets 注入
  ssh_key:
    from_secret: ssh_key      # SSH 私钥

steps:
- name: 部署代码
  commands:
    - cd /var/www/your-project
    - git pull origin main
    - composer install --no-dev

trigger:
  branch: main
```

```
kind: pipeline
type: ssh
name: deploy

server:
  host:
    from_secret: DEPLOY_HOST
  user:
    from_secret: DEPLOY_USER
  ssh_key:
    from_secret: SSH_KEY
  port: 22

steps:
# ========== 核心部署阶段 ==========
- name: 同步代码
  commands:
    # 创建临时构建目录
    - mkdir -p /tmp/drone-build
    # 克隆最新代码（使用GitHub Token）
    - git clone https://${GITHUB_TOKEN}@github.com/你的用户名/仓库名.git /tmp/drone-build
    # 同步到目标目录（排除敏感文件）
    - rsync -avz --delete --exclude=.env --exclude=storage /tmp/drone-build/ /var/www/clients/client0/web6/web/
    # 清理临时文件
    - rm -rf /tmp/drone-build

- name: 安装依赖
  commands:
    - cd /var/www/clients/client0/web6/web
    # PHP依赖
    - composer install --no-dev --optimize-autoloader
    # Node依赖（可选）
    - npm ci --production
    - npm run prod

- name: 设置权限
  commands:
    - sudo chown -R www-data:www-data /var/www/clients/client0/web6/web
    - sudo chmod -R 775 /var/www/clients/client0/web6/web/storage

# ========== 触发条件 ==========
trigger:
  branch:
    - main
  event:
    - push

# ========== 额外配置 ==========
volumes:
- name: npm_cache
  path: /var/www/clients/client0/web6/web/node_modules

environment:
  GITHUB_TOKEN:
    from_secret: GITHUB_TOKEN
```

### 关键配置说明

1. **GitHub Token 配置**
  
  - 在Drone Secrets中添加 `GITHUB_TOKEN`
  - 需要仓库的 `repo` 权限
  - 生成地址：`https://github.com/settings/tokens`

### 安全加固措施

1. **专用部署用户**

```
# 服务器上创建受限用户
sudo adduser deployer --shell /bin/bash --disabled-password
sudo usermod -aG www-data deployer
```

**SSH Jailkit限制**

```
# 限制用户目录访问
sudo apt install jailkit
sudo jk_init -v -j /home/jail netutils basicshell
sudo jk_jailuser -m -j /home/jail deployer
```

**审计日志**

```
- name: 记录部署日志
  commands:
    - echo "[$(date)] Deployed by ${DRONE_COMMIT_AUTHOR}" >> /var/log/deploy.log
```

```
# 在 pipeline 添加环境变量
environment:
  APP_ENV: production
  APP_KEY:
    from_secret: LARAVEL_APP_KEY
  DB_PASSWORD:
    from_secret: DB_PASSWORD
```

**扩展建议**：

```
# 在 install-dependencies 前添加
- name: copy-env-file
  commands:
    - cp /path/to/secure/.env /var/www/clients/client0/web2/web/.env

# 在 laravel-optimization 前添加
- name: storage-link
  commands:
    - php artisan storage:link
```
