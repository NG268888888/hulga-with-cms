---
title: uni-app使用图标
date: 2024-12-20T15:31:00+08:00
---
在iconfont.cn把图标加入购物车，在购物车-添加至项目-在我的项目页面/项目设置/字体格式：Base64勾选上，返回点击下载至本地

把下载包的iconfont.css文件单独解压出来放进uniapp项目的static文件夹，并修改iconfont.css

```
 src: 
       url('data:application ...
```

只留下url那行

在App.vue文件的style块加入

```
@import '@/static/iconfont.css';
```

之后就可以在页面中使用了

```
<text class="iconfont" style="font-size: 32px;">&#xe616;</text>
```

*其他用法参考：https://uniapp.dcloud.net.cn/component/uniui/uni-icons.html*
