---
title: macOS 配置终端代理来使用 HTTP 或 SOCKS5 代理服务
date: 2024-12-28T15:58:00+08:00
---
```
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890
export all_proxy=socks5://127.0.0.1:7890
```

检查是否生效：
```
echo $http_proxy
curl -I https://www.google.com
```
取消代理：
```
unset http_proxy
unset https_proxy
unset all_proxy
```
