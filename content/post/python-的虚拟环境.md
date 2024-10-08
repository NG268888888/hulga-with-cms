---
title: Python 的虚拟环境
date: 2024-10-08T16:59:00+08:00
---
使用 Python 的虚拟环境。在虚拟环境中，所有的包安装和配置都是在用户目录下进行，不需要提升权限。

1. 创建一个虚拟环境：

```bash
python3.11 -m venv myenv
```

2. 激活虚拟环境：
  
  ```bash
  source myenv/bin/activate
  ```
  
3. 在虚拟环境中安装所需的包：
  
  ```bash
  pip install uvicorn
  ```
  
4. 运行应用程序：
  
  ```bash
  python -m uvicorn conversation:app --host 127.0.0.1 --port 8001 --reload
  ```
  
  由于虚拟环境位于用户目录下，您无需担心权限问题。
