---
title: hugo安装hugo-blog-awesome主题和设置
date: 2025-02-25T16:13:00+08:00
---
https://github.com/hugo-sid/hugo-blog-awesome/tree/main

### Using the theme as Git submodule（不要使用Hugo module模式安装）

```
hugo new site myblog
cd myblog
git clone https://github.com/hugo-sid/hugo-blog-awesome.git themes/hugo-blog-awesome
```

在本地运行的话

```
cd themes/hugo-blog-awesome/exampleSite
hugo server --themesDir ../..
```

这样部署到netlify会有问题，或是直接把exampleSite的hugo.toml复制到根目录下访问项目，主页会在浏览器tab上显示有问题，类似这样："J Blog | J Blog"，复制exampleSite文件夹下的所有到hugo根目录

```
cp -r themes/hugo-blog-awesome/exampleSite/* ./
hugo server
```

在本地访问localhost:1313可以访问，同时也不影响之后部署到netlify

## 部署hugo到netlify

首先在hugo根目录下

```
rm -rf themes/hugo-blog-awesome/.git*
hugo
```

在github创建仓库，根据提示上传hugo根目录所有文件

在netlify部署界面选项 Environment variables下添加

- 找到 **Environment** 部分，点击 **Edit variables**。
- 添加一个新的环境变量：
  - **Key:** `HUGO_VERSION`
  - **Value:** `0.142.0`（在本地根目录hugo version）

## ## [Netlify+Cloudflare配置网站和DNS](https://www.yesmiracle.net/post/netlify-webserver-cloudflare-dns/)

自定义域名，记住**Proxied关闭**

## 在hugo博客集成waline评论

[Get Started | Waline](https://waline.js.org/en/guide/get-started/#leancloud-settings-database)

跟着教程走，到html引入这里，以hugo-blog-awesome主题为例

To use another comments system, provide your own `comments.html` partial in `layouts\partials\comments.html`.

waline的一些设置参数[Component Props | Waline](https://waline.js.org/en/reference/client/props.html)

参考：[在博客中使用 waline 评论 &#183; 瞳のBlog](https://www.hetong-re4per.com/posts/use-waline-comment-on-hugo/#html%E5%BC%95%E5%85%A5)

1. 部署完成后，请访问 /ui/register 进行注册。首个注册的人会被设定成管理员。
2. 管理员登陆后，即可看到评论管理界面。在这里可以修改、标记或删除评论。
3. 用户也可通过评论框注册账号，登陆后会跳转到自己的档案页。

## 使用PicList简化上传github图床

创建一个public github仓库，依次点击Settings -> Developer settings -> Personal access tokens -> Tokens(classic)

在新界面中`Note`项随意填写；`Expiration`项我比较懒，一般选没有期限；而`Select scopes`项，我们一般选取`repo`即可，其他默认。然后下滑到底，点击`Generate token`。

不用使用picgo，无法同步删除github的图片，插件无法使用。piclist默认集成的有同步删除远端图片。

配置路径：在piclist->图床->github->设定储存路径,images/,然后在设置->上传设置->高级重命名填写为：{Y}{m}/{filename}，这样就在github生成类似路径 images/202502/xxx.jpg
