# 板子环境变更记录

本文件记录 LubanCat-5 V2 上会影响启动、网络、远程桌面和开发环境复现的持久变更。不得写入密码、令牌、私钥或临时运行日志。

## 2026-09-17

### USB 与电脑共享网络

- 将唯一的 USB Type-C 口由纯 ADB 调整为 ADB + RNDIS 复合设备；持久配置位于 `/etc/init.d/.usb_config`。
- 保留修改前备份 `/etc/init.d/.usb_config.before-adb-rndis`。
- 新增 `/etc/usbdevice.d/rndis.sh`，固定主机侧 MAC 为 `02:42:2a:00:00:01`、板端 MAC 为 `02:42:2a:00:00:02`，避免冷启动后主机接口名变化。
- 新增 NetworkManager 连接 `LubanCat-PC`：板端 `usb0` 使用 `10.42.0.2/24`，网关和 DNS 均为电脑 `10.42.0.1`，路由与 DNS 优先级均为 50。
- 保留 `Phone-USB` 手机共享网络连接：`usb1` 使用 DHCP，路由与 DNS 优先级均为 600，作为备用链路。
- 已完成断电冷启动验证：RNDIS、固定地址、ADB、默认路由和外部域名解析均能自动恢复。

### NoMachine 无头桌面

- NoMachine 服务端由 9.3.7-1 更新为 9.9.6-2；升级时保留现有 `server.cfg.sample`。
- 将默认启动目标设为 `multi-user.target` 并停用物理 `display-manager`，避免无 HDMI 时附着到失效的物理 framebuffer；NoMachine 改为按需创建虚拟桌面。
- 在 `/usr/NX/etc/node.cfg` 中将 `VirtualDesktopVariables` 指向 `/usr/local/bin/nomachine-virtual-env.sh`；原文件备份为 `/usr/NX/etc/node.cfg.before-mesa-headless`。
- 新增 `/usr/local/bin/nomachine-virtual-env.sh`，只在 NoMachine 虚拟桌面中使用 Mesa llvmpipe 和 X11，绕开 RK3588 Mali EGL 在无物理 DRM 显示器时导致 GNOME Shell 退出的问题。
- 已在拔掉外接显示器后验证：虚拟 GNOME 桌面可登录、显示正常，鼠标点击和基本操作正常；`nxd` 在 TCP 4000 监听。

### 环境笔记

- 创建 `/home/cat/Documents/env_notes`，后续用结构化文档记录板子自身环境和已构建内容。
- 完成板端环境快照：记录系统与硬件、USB 网络与开放服务、编译/Python/RKNN 工具链、源码快照、模型和已编译 ARM64 程序。
- 记录但未擅自修复以下既有风险：`/etc` 与 `/usr` 的异常属主/组写权限、UFW inactive、多个远程服务对外监听、部分模型/程序权限为 777，以及重复的 Python 环境路径。

### Git 版本管理

- 安装 Git 2.25.1，未执行系统全面升级。
- 以普通用户 `cat` 将 `/home/cat/Documents/env_notes` 初始化为 `main` 分支的独立仓库。
- 创建板子专用 SSH Deploy Key `LubanCat Ubuntu20 eMMC`；私钥保留在板端，GitHub 端仅授予私有仓库 `WHKLY/lubancat-env-notes` 写权限。
- 完成首次提交和 push；首次远端提交为 `0cecfe369d8ae89296a99bbca4afd897a8b74814`。
- 首次 push 后复核全部环境文档，修正“Git 未安装”和“本目录不是仓库”等过时状态；本次修订经用户确认后提交。
