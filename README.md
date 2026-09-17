# LubanCat-5 V2 板端环境笔记

快照日期：2026-09-17。

本目录记录这块 LubanCat-5 V2 自身的系统、网络、服务、RKNN 开发运行环境和现有工程产物。内容来自板端实测，用于后续维护和 AI 会话恢复上下文；它不是源码或模型的备份。

## 当前定位

这块板子目前主要承担 RK3588 NPU 上的视觉模型部署与推理：系统安装了 RKNN Runtime/Lite、RGA、Mali、OpenCV，并留有 YOLOv10、YOLOv8 Pose、分类和车牌识别相关模型与脚本。

板子具备 GCC、G++、CMake 和 Make，可编译普通 ARM64/C++ 项目；但当前没有 Git、Ninja、Java、Docker、ROS、PyTorch、ONNX Runtime 或可导入的 Ultralytics 包。现状更接近“板端推理与部署环境”，不是完整的训练/模型转换工作站。

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

## 维护约定

- 系统版本、驱动、RKNN Runtime、网络、启动目标或模型产物变化后更新对应文档和 `CHANGELOG.md`。
- 不把 `.nx` 会话缓存、运行日志、图片结果、模型文件或大型 SDK 副本复制进本目录。
- 当前板子没有 Git 命令，`env_notes` 也不是 Git 仓库；需要版本管理时应先明确安装和远端策略。
- 对 `/etc`、`/usr`、网络服务或模型目录执行批量权限修复前，先阅读 [05-maintenance.md](05-maintenance.md)。
