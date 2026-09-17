# 维护、风险与核对

快照日期：2026-09-17。

## 目前需要保留的配置

- `/etc/init.d/.usb_config`：ADB + RNDIS Gadget 模式；
- `/etc/usbdevice.d/rndis.sh`：固定 RNDIS 两端 MAC；
- NetworkManager 的 `LubanCat-PC` 与 `Phone-USB`；
- `/usr/NX/etc/node.cfg` 中的 `VirtualDesktopVariables`；
- `/usr/local/bin/nomachine-virtual-env.sh`；
- 默认启动目标 `multi-user.target`；
- `/lib/librknnrt.so` 与 `cat` 用户的 RKNN Lite 2.3.2。

升级系统、NoMachine、NetworkManager 或 RKNN Runtime 后，应逐项复核这些配置。

## 已发现但未处理的风险

### 1. 核心系统目录属主与权限异常

实测：

~~~text
/etc  uid 1001，group nx(1001)，mode 775
/usr  uid 1001，group nx(1001)，mode 775
~~~

UID 1001 没有对应的 passwd 用户条目，GID 1001 是 NoMachine 的 `nx` 组。NoMachine 因此持续报告 `/etc`、`/usr` 以及指向 `/usr/lib` 的 `/lib` 存在异常属主或组可写权限。

这不是标准 Ubuntu 权限，具有较高风险，但本次没有修复。禁止直接执行 `chown -R root:root /etc /usr`：递归 chown 可能破坏包内特定属主、能力和 setuid/setgid 位。后续修复前应先：

1. 备份或制作可恢复镜像；
2. 确认该状态来自镜像制作、历史操作还是 NoMachine 安装过程；
3. 用 dpkg 文件清单、`dpkg-statoverride`、文件 capabilities 和同版本干净镜像生成差异；
4. 制定精确到路径的修复清单并分批验证。

### 2. 网络服务暴露面较大

UFW 当前 inactive，而 SSH 22、FTP 21、NoMachine 4000、ADB TCP 5555 和 ZeroTier 9993 对非回环地址监听。FTP 未启用 TLS，SSH 允许密码认证，ADB 守护进程以 root 运行。

USB 直连实验环境中这些入口可能是有意的；接入不受信网络前应重新评估。不要在不了解远程依赖时一次性关闭所有入口，以免失去管理通道。

### 3. 部分部署产物权限为 777

`0718_final.rknn`、YOLOv10 模型和演示程序允许任何本地用户修改。若没有多用户共享写入需求，后续可在备份并核对哈希后改为由 `cat:cat` 持有的 644（模型）与 755（程序）。本次没有调整。

### 4. Python 环境路径重复

`.bashrc` 重复三次加入 Python 3.8 user site，并加入不存在的 Python 3.9 user site；还全局把 `/usr/lib` 前置到 `LD_LIBRARY_PATH`。当前推理可用，但长期可能造成包或动态库来源混淆。

### 5. 源码和产物缺少版本历史

板端没有 Git，也没有 `.git` 元数据；主要目录中未发现保留的 CMake build tree。模型、源码快照和二进制的精确来源无法从板端独立还原。当前文档中的哈希只能确认之后是否变化，不能替代上游提交信息和构建配方。

### 6. 无 swap

当前 7.7 GiB 内存通常足以运行板端推理，但大模型转换、并行编译或内存泄漏时没有 swap 缓冲。不要因此默认在板上执行完整训练或大规模转换任务。

## 快速健康检查

~~~bash
# 系统和存储
uname -a
free -h
df -hT / /boot

# 温度与 NPU
for z in /sys/class/thermal/thermal_zone*; do
  printf '%s ' "$(cat "$z/type")"
  cat "$z/temp"
done
cat /sys/kernel/debug/rknpu/version
cat /sys/kernel/debug/rknpu/load

# 网络与远程入口
ip -brief address
ip route
ss -lntup
systemctl --failed
/usr/NX/bin/nxserver --status

# RKNN 与 Python
python3 -m pip show rknn-toolkit-lite2 opencv-python numpy scipy
strings /lib/librknnrt.so | grep 'librknnrt version'
ldd /home/cat/rknn_yolov10_demo/rknn_yolov10_demo

# 核心目录异常是否仍存在
stat -c '%n uid=%u gid=%g mode=%a' /etc /usr
~~~

温度文件单位是毫摄氏度，例如 `33307` 表示约 33.3°C。

## 文档更新规则

发生以下任一事件时更新 `CHANGELOG.md` 和对应章节：

- 更换镜像、内核、设备树或启动目标；
- 修改 USB Gadget、NetworkManager、监听服务或防火墙；
- 升级 NoMachine、RKNN Runtime/Lite、NPU 驱动、RGA、Mali 或 OpenCV；
- 新增/替换模型、二进制、源码快照或构建方法；
- 修复 `/etc`、`/usr` 权限或清理 `.bashrc`；
- 实际完成一次可复现推理验证。

记录版本、路径、命令和验证结果，不记录密码、令牌、私钥、Wi-Fi 凭据、ZeroTier 网络 ID、图片数据或 NoMachine 会话缓存。
