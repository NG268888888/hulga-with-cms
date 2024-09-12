---
title: Cloudflare加速访问hugo部署在netlify的博客，“Site Not Found”
date: 2024-09-12T15:57:00+08:00
---
## 1. 检查Cloudflare的DNS设置

在Cloudflare的DNS设置中，确保你已经正确配置了指向Netlify二级域名的CNAME记录。
你的CNAME记录应该如下：
Type: CNAME
Name: @（表示你的根域名，一级域名）
Target: Netlify提供的二级域名（比如 example.netlify.app）
在Cloudflare设置ssl为 full
![](/images/截屏2024-09-12-16.06.35.png)

## 2. 确保Netlify的域名配置正确

登录Netlify，进入你的网站设置页面。
在“Domain settings”中，确保你的一级域名已经正确添加到Netlify，并且Netlify已经完成了域名验证。
如果你的域名在Netlify中没有正确配置，Netlify将无法正确解析请求，因此会出现“Site Not Found”的错误。

## 3. 等待DNS解析生效

DNS修改后可能需要一些时间才能生效，通常为几分钟到几小时。如果刚刚做了修改，稍等一段时间再尝试访问。

## 4. 确保Netlify站点已成功部署

确保你的博客已经成功部署在Netlify，并且可以通过Netlify的二级域名直接访问。如果二级域名访问时也出现“Site Not Found”，那说明问题出在Netlify的部署上，而不是域名配置。
