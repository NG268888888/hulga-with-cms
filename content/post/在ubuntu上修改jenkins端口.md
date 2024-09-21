---
title: 在Ubuntu上修改Jenkins端口
date: 2024-09-21T09:01:00+08:00
---
/etc/init.d/jenkins

/etc/defaut/jenkins

/lib/systemd/system/jenkins.service

修改这3处的默认端口8080为其他
```
systemctl daemon-reload
sudo systemctl restart jenkins
```
