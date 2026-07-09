# XPoser_MiBackup - 小米云备份助手

[![Android](https://img.shields.io/badge/Android-9.0+-blue)](https://www.android.com)
[![LSPosed](https://img.shields.io/badge/LSPosed-supported-green)](https://modules.lsposed.org)
[![Github](https://img.shields.io/badge/Github-repo-blue)](https://github.com/zgcwkjOpenProject/XPoser_MiBackup)

源码: https://github.com/zgcwkjOpenProject/XPoser_MiBackup

## 原理

小米备份 APP 通过 DFS（Distributed File System）协议连接小米智能存储设备进行备份。本模块通过 Xposed Hook 注入 `com.miui.backup` 进程，拦截 DFS 连接流程，将其重定向到用户自建的 SMB 或 WebDAV 服务。

```
小米备份 APP → DFS 连接（被拦截）→ Mock CONNECTED 状态
            → 文件上传（被拦截）→ SMB/WebDAV 上传
            → 文件下载（被拦截）→ SMB/WebDAV 下载
```

## 环境要求

- Android 9.0+（minSdk 28）
- 已安装 Xposed 框架（LSPatch / LSPosed / EdXposed 等）
- 支持的 Xposed 作用域：`com.android.settings`、`com.miui.backup`

## 功能

- 在 MIUI 设置中注入「云备份助手」配置入口
- 拦截 DFS 连接，模拟智能存储设备已连接状态
- 备份时将文件上传到 SMB/WebDAV 服务器
- 恢复时从云端下载备份文件
- 自动清理超出数量限制的旧备份
- 支持 SMB/CIFS 和 WebDAV 两种传输协议

# 图片预览

![001](imgs/001.jpg?20260709)

![002](imgs/002.jpg?20260709)

![003](imgs/003.jpg?20260709)

## 配置

安装后在 MIUI 设置页面找到「云备份助手」入口，进入配置界面：

**设备配置**

| 配置项 | 说明 |
|---|---|
| 设备名称 | 虚拟设备名称，显示在设置页面 |
| 设备描述 | 设置页面中的描述文字 |
| 上传线程数 | 并发上传线程数，默认 3 |
| 备份路径 | 云端存储的根路径，如 `MIUI/backup` |
| 最大备份数 | 超出时自动清理旧备份，设 0 不清理 |

**服务配置**

| 协议 | 参数 |
|---|---|
| SMB | 服务器地址、端口、共享文件夹、用户名、密码 |
| WebDAV | 服务器地址（需 HTTPS）、用户名、密码 |

配置文件存储在 `/sdcard/MIUI/backup/config.ini`，可手动编辑。

## 许可证

[Apache License 2.0](LICENSE)
