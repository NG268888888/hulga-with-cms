---
title: " 在 ISPConfig 中部署 Laravel 应用的一键脚本"
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

DEPLOY_USER=""
PROJECT_NAME=""
TARGET_PATH="/tmp/${PROJECT_NAME}"
WEB=""
GROUP=""
WEB_ROOT="/var/www/clients/${GROUP}/${WEB}/web"
APP_PATH="${WEB_ROOT}/${PROJECT_NAME}"

KEY_NAME="id_ed25519"
KEY_PATH="/home/${DEPLOY_USER}/.ssh/${KEY_NAME}"

# 创建部署用户
if id "${DEPLOY_USER}" &>/dev/null; then
    echo "用户 ${DEPLOY_USER} 已存在，跳过创建"
else
    adduser --disabled-password --gecos "" "${DEPLOY_USER}"
fi

# 添加用户到 client0 组
usermod -aG "${GROUP}" "${DEPLOY_USER}"

# 检查是否已有密钥
if [ -f "${KEY_PATH}" ] && [ -f "${KEY_PATH}.pub" ]; then
    echo "检测到已有 SSH 密钥：${KEY_PATH}"
    read -p "是否重新生成密钥对？这将覆盖当前文件 [y/N]: " REGEN

    if [[ "$REGEN" =~ ^[Yy]$ ]]; then
        echo "重新生成 SSH 密钥对..."
        sudo -u deployer ssh-keygen -t ed25519 -N "" -f "$KEY_PATH"
    else
        echo "继续使用已有密钥。"
    fi
else
    echo "未发现密钥，正在生成新 SSH 密钥..."
    sudo -u deployer mkdir -p /home/deployer/.ssh
    sudo -u deployer chmod 700 /home/deployer/.ssh
    sudo -u deployer ssh-keygen -t ed25519 -N "" -f "$KEY_PATH"
fi

# 显示公钥内容
echo -e "\n请将以下公钥添加到 GitHub 仓库的 Deploy Keys，并开启 'Allow read access'："
echo "------------------------------------------------------------"
cat "${KEY_PATH}.pub"
echo "------------------------------------------------------------"
read -p "添加完毕后，按回车继续..."

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
sudo -u deployer composer update
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
chown -R "$WEB":"$GROUP" "$WEB_ROOT"
cd "$APP_PATH" || exit 1
chmod -R 775 storage bootstrap/cache

echo "✅ 部署完成！"
```
# 可能遇到的问题
WEB_ROOT="/var/www/clients/$GROUP/$WEB/web"与WEB_ROOT="/var/www/clients/${GROUP}/${WEB}/web"是等价的。虽然两种写法都可以，但在以下情况建议用花括号，
示例：
```
FILE="$PROJECT_NAME_backup"      # 错误，变量名会被解析成 PROJECT_NAME_backup
FILE="${PROJECT_NAME}_backup"    # 正确，明确变量范围

sudo -u deployer ssh-keygen -t ed25519 -N "" -f /home/deployer/.ssh/id_ed25519
```

-N ""	是让私钥没有密码保护（适用于自动化）

-C 这个是 注释（comment），会加在生成的公钥文件中，用于标识这对密钥的用途或来源。

-f <path>	指定密钥保存路径。如果你写的是相对路径，它会在当前工作目录下生成。

默认使用“id_ed25519”生成密钥对，使用github不会报错，
如果你之前用如下命令生成：
```
sudo -u deployer ssh-keygen -t ed25519 -f /home/deployer/.ssh/drone-ci-20250530
```
通过ssh克隆github代码，会遇到“git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.”

你需要确保 Git 用这个私钥。推荐加一个 ~/.ssh/config 文件，内容如下：
```
sudo -u deployer bash -c 'cat > /home/deployer/.ssh/config <<EOF
Host github.com
    HostName github.com
    User git
    IdentityFile /home/deployer/.ssh/drone-ci-20250530
    IdentitiesOnly yes
EOF'

sudo -u deployer chmod 600 /home/deployer/.ssh/config
```
这样 git 命令就会使用你指定的密钥。


# 如何让 GitHub 在不同项目使用不同的 SSH 密钥？

## ✅ 解决方案：使用 `~/.ssh/config` 按项目“域名别名”绑定不同的密钥

GitHub 默认只有一个 `github.com`，但你可以人为给不同项目设置别名（Host alias），然后绑定不同的私钥。

---

### 🧠 举个例子：

你有两个项目：

| 项目  | SSH地址                               | 目标               |
| --- | ----------------------------------- | ---------------- |
| A项目 | `git@github.com:user/project-a.git` | 用 `id_project_a` |
| B项目 | `git@github.com:user/project-b.git` | 用 `id_project_b` |

### 步骤如下：

---

## 🔹 第一步：在 `.ssh/config` 里配置不同 Host alias

```bash
# ~/.ssh/config

Host github-project-a
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_project_a
    IdentitiesOnly yes

Host github-project-b
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_project_b
    IdentitiesOnly yes
```

---

## 🔹 第二步：在 `.git` 或 clone 时使用对应 Host

改写 clone 命令：

```bash
# 对于 A 项目
git clone git@github-project-a:user/project-a.git

# 对于 B 项目
git clone git@github-project-b:user/project-b.git
```

Git 会自动根据 `Host` 匹配你 `.ssh/config` 中的配置，从而使用对应的密钥文件。

---

## ✅ 最终效果

你可以：

* 为每个项目生成一对密钥
* `.ssh/config` 中配置多个 `Host`
* clone 时使用别名（如 `github-project-a`）
* 不冲突、不共用同一私钥
* 项目安全、易于管理

---

# linux删除已创建的用户

要删除已创建的用户 `aaa`，你可以根据是否要保留其主目录、文件等选择不同命令：

---

## 🧹 **1. 删除用户但保留主目录和文件**

```bash
sudo deluser aaa
```

* 只会从系统中删除用户账户
* `/home/aaa/` 等文件不会删除
* 通常用于只是禁用登录权限，但保留数据

---

## 🧼 **2. 删除用户并同时删除主目录、邮箱等个人数据**

```bash
sudo deluser --remove-home aaa
```

* 删除用户
* 删除 `/home/aaa/` 目录
* 如果配置了邮件服务，也会删除其邮箱目录（如 `/var/mail/aaa`）

---

## 💣 **3. 删除用户 + 删除其所有系统痕迹（慎用）**

```bash
sudo deluser --remove-all-files aaa
```

* 删除用户
* 删除该用户在系统中**拥有的所有文件**（哪怕不在 `/home/aaa` 下）

⚠️ **这一步可能会误删项目或其他位置的数据，除非你确信该用户的文件不再需要。**

---

## 🧰 附加操作（清理用户组）

如果你还创建了与用户名同名的组，可以删除该组：

```bash
sudo delgroup aaa
```

---

## ✅ 建议流程（安全且干净）：

```bash
sudo deluser --remove-home aaa
sudo delgroup aaa
```

---

