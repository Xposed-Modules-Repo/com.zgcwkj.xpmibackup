# XPoser_MiBackup - 小米云备份助手

[![Android](https://img.shields.io/badge/Android-9.0+-blue)](https://www.android.com)
[![Github](https://img.shields.io/badge/Github-repo-blue)](https://github.com/zgcwkjOpenProject/XPoser_MiBackup)
[![LSPosed](https://img.shields.io/badge/LSPosed-supported-green)](https://modules.lsposed.org)
[![XposedModule](https://img.shields.io/badge/XposedModule-repo-green)](https://github.com/Xposed-Modules-Repo/com.zgcwkj.xpmibackup)

通过 Xposed 模块虚拟小米智能存储设备，将小米备份 App 的 DFS 存储流程重定向到自建 SMB、WebDAV 或自定义 HTTP 服务，实现备份与恢复数据的云端存储

## 原理

小米备份 App 会通过 DFS 服务连接小米智能存储设备，并通过 AIDL 接口执行目录查询、文件上传、文件下载和进度回调；本模块注入 `com.android.settings` 与 `com.miui.backup` 进程，在设置页展示配置入口，并在备份 App 的 DFS/AIDL 边界将文件操作改由 SMB/WebDAV/自定义 HTTP 完成

```text
小米备份 App
  -> 查询智能存储设备：返回虚拟设备
  -> 连接 DFS 服务：模拟在线和已连接
  -> DFS AIDL 上传：写入 SMB/WebDAV/自定义 HTTP
  -> DFS AIDL 下载：从 SMB/WebDAV/自定义 HTTP 读取
  -> 进度与完成回调：回传给小米备份原流程
```

主要 Hook 边界在 DFS AIDL、设置页入口和明确的备份 UI/服务事件，尽量避免直接依赖混淆业务函数

## 功能

- 在 系统设置中注入「云备份助手」配置入口
- 拦截 DFS 连接，模拟小米智能存储设备在线状态
- 支持 SMB/CIFS、WebDAV 和自定义 HTTP 脚本三种传输协议
- 备份和恢复时将文件重定向到 SMB/WebDAV/自定义 HTTP 服务器
- 顶部“备份”按钮点击进入小米备份的智能存储备份页，长按进入小米应用商店的备份升级页
- 大文件在 Cloud 层统一切片上传，SMB、WebDAV 和自定义 HTTP 共用同一套切片逻辑
- 自动清理超出数量限制的旧备份

# 图片预览

![001](imgs/001.jpg?20260711)

![002](imgs/002.jpg?20260711)

更多图片：[imgs](imgs/README.md)

## 环境要求

- Android 9.0+（minSdk 28）
- 已安装 Xposed 框架（LSPatch / LSPosed / EdXposed 等）
- 支持的 Xposed 作用域：`com.android.settings`、`com.miui.backup`

## 配置

配置入口：系统设置 -> 云备份助手

配置文件：

```text
/sdcard/MIUI/backup/config.ini
```

常用配置：

| 配置项 | 说明 |
| --- | --- |
| `protocol` | 存储协议，`smb`、`webdav` 或 `custom` |
| `backup_path` | 云端备份根目录，默认 `MIUI/backup` |
| `upload_threads` | 并发上传线程数，默认 `3` |
| `chunk_size_mb` | 上传切片大小，默认 `64`；设为 `0` 表示不切片 |
| `backup_max` | 最大保留备份数，`0` 表示不自动清理 |
| `device_name` | 设置页中展示的虚拟设备名称 |
| `device_describe` | 设置页中展示的虚拟设备描述 |

SMB 配置：

| 配置项 | 说明 |
| --- | --- |
| `smb_server` | SMB 服务器地址 |
| `smb_port` | SMB 端口，默认 `445` |
| `smb_share` | SMB 共享名 |
| `smb_user` | SMB 用户名 |
| `smb_pass` | SMB 密码 |

WebDAV 配置：

| 配置项 | 说明 |
| --- | --- |
| `webdav_url` | WebDAV 根地址 |
| `webdav_user` | WebDAV 用户名 |
| `webdav_pass` | WebDAV 密码 |

## 许可证

[Apache License 2.0](LICENSE)
