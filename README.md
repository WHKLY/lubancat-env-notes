# LubanCat-5 V2 板端环境笔记

快照日期：2026-09-17。

本目录记录这块 LubanCat-5 V2 自身的系统、网络、服务、RKNN 开发运行环境和现有工程产物。内容来自板端实测，用于后续维护和 AI 会话恢复上下文；它不是源码或模型的备份。

## 当前定位

这块板子目前主要承担 RK3588 NPU 上的视觉模型部署与推理：系统安装了 RKNN Runtime/Lite、RGA、Mali、OpenCV，并留有 YOLOv10、YOLOv8 Pose、分类和车牌识别相关模型与脚本。

板子具备 Git 2.25.1、GCC、G++、CMake 和 Make，可维护环境文档并编译普通 ARM64/C++ 项目；但当前没有 Ninja、Java、Docker、ROS、PyTorch、ONNX Runtime 或可导入的 Ultralytics 包。现状更接近“板端推理与部署环境”，不是完整的训练/模型转换工作站。

## 文档索引

- [CHANGELOG.md](CHANGELOG.md)：影响启动、网络、远程桌面和开发复现的持久变更。
- [01-system-hardware.md](01-system-hardware.md)：板型、系统、CPU、内存、存储和硬件加速栈。
- [02-network-services.md](02-network-services.md)：USB 网络、路由、远程入口、运行服务和监听端口。
- [03-development-stack.md](03-development-stack.md)：编译工具、Python 包、RKNN Runtime 和 shell 环境。
- [04-projects-artifacts.md](04-projects-artifacts.md)：已有源码快照、演示目录、模型和已编译程序。
- [05-maintenance.md](05-maintenance.md)：快速核对、已知风险和后续维护规则。

## 快速入口

~~~bash
# 电脑通过 USB 进入 root ADB shell
adb shell

# 从电脑连接板端网络
ping 10.42.0.2
ssh cat@10.42.0.2

# 板端读取当前环境笔记
less /home/cat/Documents/env_notes/README.md
~~~

NoMachine 使用 `10.42.0.2:4000` 和系统用户 `cat`。本目录不记录密码、私钥、令牌、Wi-Fi 凭据、ZeroTier 网络 ID 或 NoMachine 会话认证字段。

## 版本管理

本目录已经初始化为独立 Git 仓库，由普通用户 `cat` 维护：

| 项目 | 当前值 |
|---|---|
| 本地路径 | `/home/cat/Documents/env_notes` |
| 分支 | `main` |
| 远端 | `git@github-lubancat-env:WHKLY/lubancat-env-notes.git` |
| GitHub 可见性 | Private |
| 首次远端提交 | `0cecfe369d8ae89296a99bbca4afd897a8b74814` |
| Deploy Key | `LubanCat Ubuntu20 eMMC`，仅限该仓库并启用写权限 |

SSH 私钥只保存在 `/home/cat/.ssh/id_ed25519_github_lubancat_u20_emmc`，不属于本仓库。该密钥不可复制到文档、聊天、其他系统或未来的 Ubuntu 24 环境；新系统应生成自己的 Deploy Key。

## 维护约定

- 系统版本、驱动、RKNN Runtime、网络、启动目标或模型产物变化后更新对应文档和 `CHANGELOG.md`。
- 不把 `.nx` 会话缓存、运行日志、图片结果、模型文件或大型 SDK 副本复制进本目录。
- 修改文档后先检查 `git diff`，只添加本次明确修改的 Markdown 文件，再 commit 和 push；不要习惯性使用 `git add .`。
- 对 `/etc`、`/usr`、网络服务或模型目录执行批量权限修复前，先阅读 [05-maintenance.md](05-maintenance.md)。
