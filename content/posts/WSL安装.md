---
title: "Windows WSL安装"
date: 2020-06-27
draft: false
tags: ["Linux"]
# categories: ["ca1"]

---


### 第 1 步，启用 WSL

以管理员身份打开 PowerShell 工具并运行以下命令

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

### 第 2 步，启用“虚拟机平台”

WSL 2 需要启用 Windows 10 的 “虚拟机平台” 特性。它独立于 Hyper-V，并提供了一些在 Linux 的 Windows 子系统新版本中可用的更有趣的平台集成。

要在 Windows 10（2004）上启用虚拟机平台，以管理员身份打开 PowerShell 并运行：

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

要在 Windows 10（1903，1909）上启用虚拟机平台，请以管理员身份打开 PowerShell 并运行：

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName VirtualMachinePlatform -NoRestart
```

重启系统

### 第 3 步，更新内核组件

wsl_update_x64.exe


### 第 4步，设置 WSL 2 为默认值

以管理员身份打开 PowerShell，然后运行以下命令以将 WSL 2 设置为 WSL 的默认版本：

```powershell
wsl --set-default-version 2
```


### 第 5 步，安装 Linux 发行版

从 Microsoft Store 中选择并安装 Linux 发行版

有几种不同的发行版可供选择，包括 OpenSUSE、Pengwin、Fedora Remix 和 Alpine Linux。

个人推荐Ubuntu 20.04 LTS（尽管也有 18.04 LTS 和 16.04 LTS 可用）。

要在 Windows 10 上安装 Ubuntu，请打开 Microsoft Store 应用，搜索 “Ubuntu 20.04”。

`wsl.exe --set-version Ubuntu 2`