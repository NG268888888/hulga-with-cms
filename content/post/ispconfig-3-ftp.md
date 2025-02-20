---
title: " ispconfig 3 ftp"
date: 2024-12-22T23:12:00+08:00
---
按照ispconfig 3官网教程可以正常使用ftp,前提是没有开启ispconfig后台的防火墙，如果开启需要进行一系列设置。

ispconfig 3 安装的pure-ftpd-mysql，这表明你的 PureFTPd 配置使用了 MySQL 来存储用户信息，ispconfig 3不再支持vsftpd

**配置被动模式**：
PureFTPd 默认启用了被动模式。你需要指定被动端口范围：

编辑 `/etc/pure-ftpd/conf/PassivePortRange`：

确保端口 21 (FTP 控制连接) 和 TLS 数据传输端口范围（如 20000-30000）已开放。最好在后台firewall手动添加。

配置完成后，重启 PureFTPd：

```
echo "30000 31000" | sudo tee /etc/pure-ftpd/conf/PassivePortRange
sudo ufw allow 21/tcp
sudo ufw allow 30000:31000/tcp
sudo ufw reload
sudo systemctl restart pure-ftpd-mysql
```

**配置**FileZilla**站点管理器**：

* 打开 FileZilla，点击 **文件 > 站点管理器**。
* 新建站点，输入以下信息：

  * **主机**：服务器的 IP 地址或域名。
  * **端口**：为空，默认21。
  * **协议**：`FTP - 文件传输协议`
  * **加密**：`要求显式的 FTP over TLS`。
  * **登录类型**：`正常`。
  * **用户名**：ISPConfig 中创建的 FTP 用户名。
  * **密码**：FTP 用户对应的密码。

### **1. 检查 PureFTPd 服务是否正常运行**

运行以下命令，确保服务正常运行：

```bash
sudo systemctl status pure-ftpd-mysql
```

如果服务未运行或状态显示错误，请尝试重启：

```bash
sudo systemctl restart pure-ftpd-mysql
```

查看日志文件以获取更多信息：

```bash
sudo tail -f /var/log/syslog
```

- - -

### **2. 确保端口已开放**

PureFTPd 使用以下端口：

* **端口 21**：FTP 控制连接。
* **被动模式端口范围**：通常在 30000-31000。

检查防火墙是否允许这些端口：

```bash
sudo ufw allow 21/tcp
sudo ufw allow 30000:31000/tcp
sudo ufw reload
```

- - -

### **3. 检查 TLS 配置**

* 确认 TLS 已启用：

  ```bash
  cat /etc/pure-ftpd/conf/TLS
  ```

  确认输出为 `1`。
* 确保 SSL 证书路径和权限正确：

  ```bash
  ls -l /etc/ssl/private/pure-ftpd.pem
  ```

  权限应为 `600`。

- - -

### **4. FileZilla 配置检查**

确保 FileZilla 的设置与 PureFTPd 配置一致：

1. 打开 **FileZilla**，进入 **站点管理器**。
2. 输入以下信息：

* **主机**：服务器的 IP 地址或域名。
* **端口**：`21`。
* **协议**：`FTP - 文件传输协议`
* **加密**：`需要使用明文 FTP over TLS`。
* **登录类型**：`正常`。
* **用户名**：ISPConfig 创建的 FTP 用户名。
* **密码**：FTP 用户密码。

3. **首次连接时**，FileZilla 会提示接受 TLS 证书。选择 **接受**。

- - -

### **5. 检查 PureFTPd 配置文件**

确保以下配置在 `/etc/pure-ftpd/pure-ftpd.conf` 中已启用：

* **被动模式端口范围**：

  ```bash
  echo "30000 31000" > /etc/pure-ftpd/conf/PassivePortRange
  ```
* **VIRTUALCHROOT 设置**：
  检查 `/etc/default/pure-ftpd-common` 文件，确保：

  ```
  STANDALONE_OR_INETD=standalone
  VIRTUALCHROOT=true
  ```

重启服务：

```bash
sudo service pure-ftpd-mysql restart
```

### **8. 可能的错误与解决方案**

#### **无法列出目录内容**

* 检查防火墙是否开放了被动模式端口范围。
* 在 FileZilla 的设置中，启用被动模式： **站点管理器 > 传输设置 > 传输模式**，选择 **被动模式**。
