---
title: " 在 ISPConfig 中部署 Laravel 应用的一件脚本"
date: 2025-02-22T22:54:00+08:00
---
确认安装了composer
```
#!/bin/bash

# 一键化部署脚本，需用root权限执行
if [[ $EUID -ne 0 ]]; then
   echo "此脚本必须以root身份运行" 
   exit 1
fi

DEPLOY_USER="deployer"
PROJECT_NAME="project-name"
TARGET_PATH="/tmp/project-name"
WEB_ROOT="/var/www/clients/client0/web1/web"
APP_PATH="$WEB_ROOT/project-name"
WEB="web1"

# 创建部署用户
adduser --disabled-password --gecos "" deployer
usermod -aG client0 deployer

# 生成SSH密钥（自动化接受默认路径）
sudo -u deployer ssh-keygen -t ed25519 -N "" -f /home/deployer/.ssh/id_ed25519
echo -e "\n请将以下公钥添加到GitHub："
cat /home/deployer/.ssh/id_ed25519.pub
read -p "按回车继续..." # 等待确认密钥已添加

# 获取仓库地址
read -p "请输入Git仓库地址（SSH格式）: " REPO_URL
until [[ $REPO_URL =~ ^git@ ]]; do
    read -p "地址格式不正确，请使用SSH格式（如 git@github.com:user/repo.git）: " REPO_URL
done

# 克隆项目（带格式验证）
echo "正在克隆仓库到 $TARGET_PATH ..."
if ! sudo -u deployer git clone "$REPO_URL" "$TARGET_PATH" 2>&1 | tee -a clone.log; then
    echo -e "\n❌ 克隆失败，请检查："
    echo "1. 仓库地址是否正确"
    echo "2. SSH密钥是否已添加到GitHub"
    echo "3. 日志文件 clone.log"
    exit 1
fi

# 安装依赖
cd "$TARGET_PATH" || exit 1
sudo -u deployer composer install --no-dev --prefer-dist --optimize-autoloader

# 环境配置
sudo -u deployer cp .env.example .env
# 使用vi打开.env文件进行编辑
echo "请编辑 .env 文件并保存后，按任意键继续..."
sudo -u deployer vi .env
echo "请编辑 admin.php 文件并保存后，按任意键继续..."
sudo -u deployer vi config/admin.php
sudo -u deployer php artisan key:generate
sudo -u deployer php artisan migrate
sudo -u deployer php artisan storage:link
sudo -u deployer php artisan optimize:clear

# 检查WEB_ROOT目录下是否有项目，如果有则删除
if [ -d "$WEB_ROOT/$PROJECT_NAME" ]; then
    echo "删除旧的$PROJECT_NAME目录..."
    rm -rf "$WEB_ROOT/$PROJECT_NAME"
fi

# 移动项目到目标目录
echo "移动$PROJECT_NAME到目标目录..."
mv "$TARGET_PATH" "$WEB_ROOT"

# 设置权限
echo "设置权限..."
chown -R "$WEB":client0 "$WEB_ROOT"
cd "$APP_PATH" || exit 1
chmod -R 775 storage bootstrap/cache

echo "✅ 部署完成！"
```
