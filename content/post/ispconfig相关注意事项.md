---
title: ispconfig相关注意事项
date: 2025-02-22T23:04:00+08:00
---
ispconfig后台更改server域名后，需要把服务器的/etc/hosts /etc/hostname更改，然后在ispconfig文件夹运行php -q update.php，成功后会继续使用https

ispconfig部署的网站域名使用cloudflare的DNS，域名的ssl/tls选择完全，不要使用完全(严格).不使用cloudflare后台的http->https，在ispconfig里设置

ispconfig部署多php版本

https://www.howtoforge.com/ispconfig-php-ubuntu/

https://shape.host/resources/how-to-install-multiple-versions-of-php-on-debian-with-ispconfig
