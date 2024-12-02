---
title: fedora39 kde fcitx5漏字、ibus中文输入法无输入框的解决办法
date: 2024-12-02T21:30:00+08:00
---
fedora39 kde fcitx5漏字、ibus中文输入法无输入框的解决办法

```
sudo dnf remove ibus
sudo dnf remove fcitx
sudo dnf autoremove
sudo dnf install fcitx5
sudo dnf install fcitx5-chinese-addons
```

之后fcitx5就不会出现漏字问题了
