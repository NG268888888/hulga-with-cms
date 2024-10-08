---
title: 逐步完成 Jenkins CICD与GitHub集成
date: 2024-09-13T10:35:00+08:00
---
## 123
部署项目的的vps安装jenkins后，每当本地推送代码到github，jenkins会自动部署最新的项目代码到vps。

### 在vps上安装jenkins

**开始使用jenkins**

1. 在 Jenkins 仪表板上，单击“New Item”以创建您的第一个作业或项目。
2. 输入您的第一个项目的名称并选择“Freestyle project”，然后单击确定。
3. 现在我们需要在 Jenkins 中配置第一个作业的各种参数。
![](/images/imag.webp)

GitHub project - Project url一定填写https://开头的git仓库地址

Source Code Managment - Git - Reponstory URL 填写刚才一样的git仓库地址

在vps上使用命令：ssh-keygen生成2个文件，将生成的XXX.pub内容复制到
Github - Settings - SSH 爱and GPG keys

在Jenkins中的Credentials点击add，选择jenkins，Username填写vps的ssh用户名，Private Key粘贴vps刚才生成的XXX文件内容。

选择要持续集成的Github仓库的分支，一半都是main或者master

最后点击Build Now，看看是否成功

如果有问题，在Console Output中查看日志解决

### 如果是Git私有仓库并且用ssh验证

GitHub project - Project url填git@github.com:XXX/XXX.git

Source Code Management - Git - Repository URL填git@github.com:XXX/XXX.git

在vps上使用命令：ssh-keygen -t ed25519 将生成的XXX.pub内容复制到Github对应的私有仓库- Settings - Deploy keys

在Jenkins中的Credentials点击add，选择jenkins，Username试试github项目的名字vthily（例如：git@github.com:vthily/<your-git-repo>.git），Private Key粘贴vps刚才生成的XXX文件内容。

Github仓库的分支，一般都是main或者master

提示错误：

> stderr: No ED25519 host key is known for github.com and you have requested strict > checking.
> Host key verification failed.

在vps上输入git ls-remote -h git@github.com:XXX/XXX.git HEAD

会在~/.ssh/known_hosts把私钥存入known_hosts文件

修改~/.ssh和known_hosts的权限

然后就不会提示错误了（实际测试依旧报错No ED25519 host key is known）

终极解决办法：go to 'Manage Jenkins' -> 'Security' -> 'Git Host Key Verification Configuration' and configure host key verification，选择不验证

参考：https://medium.com/@mudasirhaji/complete-step-by-step-jenkins-cicd-with-github-integration-aae3961b6e33
