+++
title = 'PDM使用小结'
date = 2024-11-04T13:49:57+08:00
draft = false
tags = ["python", "pdm"]
summary = 'PDM 是一个 Python 包管理工具，本文是一个关于如何安装、设置的小结'
+++

# PDM 使用小结

# 安装

安装 PDM 只需要安装官网上的说明即可，但有时因为 [raw.githubcontent.com](https://raw.githubcontent.com/) 域名会被 DNS 污染，所以需要单独下载安装脚本进行安装。

从[pdm/install-pdm.py at main · pdm-project/pdm (github.com)](https://github.com/pdm-project/pdm/blob/main/install-pdm.py) 下载该文件后，执行

```bash
python install-pdm.py
```

就应该能正常安装了，如果是 Windows 系统，需要 PowerShell 窗口执行，但默认 PS 没有执行外来脚本的权限，需要执行

```bash
Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser
```

# 设置使用国内源

```bash
pdm config pypi.url https://pypi.tuna.tsinghua.edu.cn/simple
```

# 安装 Python

PDM 包管理器支持多版本 Python 的选择，可以使用

```bash
pdm python install 3.11
```

来安装 Python 版本。这里 3.11 可以替换成其他的版本。可以使用

```bash
pdm python install -l
```

来打印所有支持安装的版本。

```bash
pdm python list
```

则可以显示当前系统已经安装的 Python 版本。

# 选用 Python 版本

使用

```bash
pdm use 3.11
```

即可
