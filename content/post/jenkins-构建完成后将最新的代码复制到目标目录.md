---
title: Jenkins 构建完成后将最新的代码复制到目标目录
date: 2024-09-30T10:00:00+08:00
---
本想使用软连接Jenkins 的工作区和 ISPConfig 生成的网站目录同步，错误提示“Too many levels of symbolic links”，导致符号链接解析过程中产生无限循环。

\### 使用部署脚本

考虑使用 Jenkins 构建完成后的“部署步骤”将最新的代码复制到目标目录，而不是直接使用符号链接。你可以在 Jenkins 构建任务完成后，使用以下命令复制文件：

\`\``

cp -r /var/lib/jenkins/workspace/laravel/* /var/www/clients/client0/web1/web/

\`\``



可能遇到的错误：

\> cp: cannot create regular file '/var/www/clients/client0/web1/web/README.md': Permission denied

\> cp: cannot create directory '/var/www/clients/client0/web1/web/app': Permission denied



Jenkins 在尝试将文件复制到 /var/www/clients/client0/web1/web/ 目录时，遇到了权限问题。这通常是由于 Jenkins 运行的用户没有足够的权限在目标目录中写入文件导致的。默认情况下，Jenkins 可能是以 jenkins 用户的身份运行，而 /var/www/clients/client0/web1/web/ 目录通常是由 www-data 或 nginx 用户拥有，权限不足会导致操作失败。



\*\*解决步骤:\*\*

1. 检查目标目录的权限

\`\``

ls -ld /var/www/clients/client0/web1/web

\`\``

如果目录由 www-data 或其他用户拥有，而 jenkins 用户没有写入权限，你就会遇到类似的权限问题。



2. 为 Jenkins 用户授予适当权限



有几种方法可以解决这个问题：



方法 1: 修改目录所有者或权限 如果你确定 Jenkins 用户需要有写入权限，你可以使用以下命令将目录的所有者修改为 Jenkins 用户：

\`\``

sudo chown -R jenkins:jenkins /var/www/clients/client0/web1/web

\`\``

这样做会使 Jenkins 用户成为该目录的所有者，能够在该目录中写入文件。



或者，你可以直接授予目录写入权限，而不更改所有者（这种方法可能稍微不安全，建议仅限于开发或受控环境）：

\`\``

sudo chmod -R 775 /var/www/clients/client0/web1/web

\`\``

这会允许目录的拥有者和组成员都有写入权限。



方法 2: 将 Jenkins 用户添加到 www-data 组 如果不想修改目录的所有者，你可以将 Jenkins 用户添加到 www-data 组（假设 www-data 是 /var/www/clients/client0/web1/web/ 的组所有者）：

\`\``

sudo usermod -aG www-data jenkins

\`\``

然后，确保该目录及其子目录的组所有者有写入权限：

\`\``

sudo chmod -R g+w /var/www/clients/client0/web1/web

\`\``

\`为了确保 Jenkins 用户组更新生效，重启 Jenkins 服务：\`

\`\``

sudo systemctl restart jenkins

\`\``

这样 Jenkins 用户就可以通过 www-data 组访问并写入该目录了。



3. 在 Jenkins Pipeline 中使用 sudo

如果你不想修改目录权限，还可以在 Jenkins 部署过程中使用 sudo 命令来提高权限。但这需要 Jenkins 用户能够在 sudoers 文件中配置无密码执行 sudo 命令：



编辑 sudoers 文件：

\`\``

sudo visudo

\`\``

添加如下规则，允许 Jenkins 用户在没有密码的情况下运行 cp 和其他部署相关的命令：

\`\``

jenkins ALL=(ALL) NOPASSWD: /bin/cp, /bin/mkdir, /bin/chown, /bin/chmod

\`\``

然后在 Jenkins 部署脚本中，使用 sudo 运行 cp 命令：

\`\``

sudo cp -r /var/lib/jenkins/workspace/laravel-chat/* /var/www/clients/client0/web1/web/

\`\``

结论：

最常见的做法是将 Jenkins 用户添加到适当的组，并调整目录的组权限或直接更改文件的所有者，确保 Jenkins 有权限进行文件操作。
