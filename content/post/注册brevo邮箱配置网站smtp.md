---
title: 注册Brevo配置laravel网站SMTP
date: 2024-10-15T08:56:00+08:00
---
注册Brevo账号
https://onboarding.brevo.com/account/register

手机号可用国内。

每天免费发送 300 封邮件 一般足够用了

绑定域名
https://app.brevo.com/senders/domain/list

添加发送者邮箱
https://app.brevo.com/senders/add

SMTP验证

首先获取SMTP相关配置信息

进入smtp控制页面
https://app.brevo.com/settings/keys/smtp


配置laravel .env信息:
```
MAIL_MAILER=smtp
MAIL_HOST=SMTP Server
MAIL_PORT=Port #587大部分服务器商都封，使用2525代替
MAIL_USERNAME=Login
MAIL_PASSWORD=SMTP key value
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=都可以
MAIL_FROM_NAME="${APP_NAME}"
```
