---
title: 在 Linux 系统中，为 `.AppImage` 文件创建快捷方式
date: 2024-11-30T14:42:00+08:00
---
`~/.local/share/applications/your-app.desktop`

`/usr/share/applications/cursor.desktop`

添加以下内容并保存：

```
[Desktop Entry]
Name=Cursor
Exec=/opt/cursor.appimage
Icon=/opt/cursor.png
Type=Application
Categories=Development;
```

- **`Name`**：显示的名称。
- **`Exec`**：`.AppImage` 文件的绝对路径。
- **`Icon`**：应用图标的路径，可以使用 `.png` 或 `.svg` 图标。
- **`Terminal`**：如果需要终端运行，改为 `true`。
- **`Categories`**：分类标签，可根据应用类型调整。

**设置权限**

`chmod +x /path/to/your-app.AppImage`

**刷新菜单**

update-desktop-database ~/.local/share/applications
