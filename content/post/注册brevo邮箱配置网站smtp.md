---
title: 注册Brevo邮箱配置网站SMTP
date: 2024-10-15T08:56:00+08:00
---
注册Brevo账号
https://onboarding.brevo.com/account/register

手机号任意填写，并不会接收短信来验证。

每天免费发送 300 封邮件 一般足够用了

绑定域名
https://app.brevo.com/senders/domain/list

添加发送者邮箱
https://app.brevo.com/senders/add

SMTP验证

首先获取SMTP相关配置信息

进入smtp控制页面
https://app.brevo.com/settings/keys/smtp

选择stmp
配置信息如下:
```
SMTP Server:   smtp-relay.sendinblue.com
Port:     587
Login:    brevo@你的域名.com
Protocol:     tls
Value：    JcrM5LHXXXXXXXX
```
