# 系统与硬件

快照日期：2026-09-17。

## 身份与系统

| 项目 | 当前值 |
|---|---|
| 板型 | Embedfire LubanCat-5 V2 |
| Device Tree | `embedfire,rk3588-lubancat-5-v2`、`rockchip,rk3588` |
| SoC / 架构 | Rockchip RK3588 / aarch64 |
| 主机名 | `lubancat` |
| 系统 | Ubuntu 20.04.6 LTS（Focal） |
| 镜像构建信息 | `root@dev120.embedfire.local Sun Jan 4 18:14:58 CST 2026` |
| 内核 | 5.10.160，`#15 SMP Mon Jan 5 08:40:00 CST 2026` |
| 时区 | Asia/Shanghai；系统时钟已同步，RTC 使用 UTC |
| 普通用户 | `cat`，UID/GID 1000；属于 `sudo`、`audio`、`video` |
| ADB shell | root |

## CPU、内存与温度

RK3588 共 8 个 CPU，当前 cpufreq 策略分为三组：

| CPU | 频率范围 |
|---|---|
| 0–3 | 408 MHz–1.8 GHz |
| 4–5 | 408 MHz–2.352 GHz |
| 6–7 | 408 MHz–2.352 GHz |

内存总量 7.7 GiB。盘点时约使用 1.0 GiB、可用 6.6 GiB；没有交换分区或 swapfile。

冷启动后空闲温度约为 32–34°C：SoC、小核、大核、GPU 和 NPU 温区均在该范围。这只是一次空闲快照，不代表持续推理负载温度。

## 存储

系统位于 58.7 GB eMMC：

| 设备 | 文件系统 | 挂载点 | 容量 | 盘点时占用 |
|---|---|---|---|---|
| `/dev/mmcblk0p1` | 未标注 | 未挂载 | 8 MB | 启动保留分区 |
| `/dev/mmcblk0p2` | ext2，标签 `boot` | `/boot` | 128 MB | 72 MB，61% |
| `/dev/mmcblk0p3` | ext4 | `/` | 58.5 GB | 9.8 GB，18% |

`/home/cat` 盘点时约 3.4 GB，其中约 2.1 GB 来自 RKNN Toolkit 备份目录。`/tmp` 是内存文件系统。

## NPU、GPU 与 RGA

| 组件 | 当前状态 |
|---|---|
| RKNPU 内核驱动 | 0.9.8，IOMMU 模式，3 个 NPU 核心；空闲快照负载均为 0% |
| RKNN Runtime | 2.3.2，构建 `429f97ae6b@2025-04-09T09:09:27` |
| GPU | Mali，内核 DDK `g18p0-01eac0`；用户库 `libmali` 1.9 |
| RGA 内核模块 | 1.2.27；发现两个 RGA3 核心和一个 RGA2 |
| RGA 用户包 | `librga2` / `librga-dev` 2.2.0 |
| OpenCL | Mali OpenCL 库已安装 |
| DRM 节点 | `/dev/dri/card0`、`card1`、`renderD128`、`renderD129` |

`cat` 属于 `video` 组，可访问组为 `video` 的 card 节点；它不属于 `render` 组。现有 RKNN Lite 推理依赖板端 `/lib/librknnrt.so`。

## 图形系统与启动方式

| 项目 | 当前值 |
|---|---|
| GNOME Shell | 3.36.9 |
| X.Org | 1.20.11 |
| GDM | 3.36.3 |
| 默认 systemd target | `multi-user.target` |
| `display-manager` | inactive，按当前无头方案不自动启动 |
| NoMachine | 9.9.6，按需创建虚拟 GNOME X11 桌面 |

板子仍安装完整本地图形栈，但默认不启动物理显示管理器。这是无 HDMI 时保持 NoMachine 输入和显示正常的有意配置，不代表 GNOME/GDM 已卸载。

## 快速核对

~~~bash
cat /etc/os-release
uname -a
tr '\0' '\n' </proc/device-tree/compatible
free -h
lsblk -o NAME,SIZE,TYPE,FSTYPE,LABEL,MOUNTPOINT
df -hT / /boot
cat /sys/kernel/debug/rknpu/version
cat /sys/kernel/debug/rknpu/load
systemctl get-default
systemctl is-active display-manager
~~~
