# 网络与服务

快照日期：2026-09-17。

## 网络接口与路由

| 接口 | 状态与地址 | 用途 |
|---|---|---|
| `usb0` | UP，`10.42.0.2/24` | 电脑 USB RNDIS，当前默认出口 |
| `ztxoogfbja` | UP，`10.67.218.100/24` | ZeroTier 虚拟网络 |
| `eth0`、`eth1` | DOWN / no carrier | 板载以太网口，当前未接线 |
| `usb1` | 未出现 | 预留给手机 USB 共享，连接配置已保留 |
| `lo` | `127.0.0.1/8`、`::1/128` | 本机回环 |

当前默认路由：

~~~text
default via 10.42.0.1 dev usb0 proto static metric 50
~~~

电脑 USB 配置 `LubanCat-PC` 使用固定地址 `10.42.0.2/24`，网关和 DNS 均为 `10.42.0.1`，路由与 DNS 优先级均为 50。手机配置 `Phone-USB` 绑定 `usb1`、使用 DHCP，路由与 DNS优先级均为 600，因此电脑和手机同时存在时优先走电脑。

## USB Gadget

`usbdevice.service` 开机启用。`/etc/init.d/.usb_config` 同时启用：

~~~text
usb_adb_en
usb_rndis_en
~~~

`/etc/usbdevice.d/rndis.sh` 固定两端 MAC：

- 电脑侧：`02:42:2a:00:00:01`；
- 板端 `usb0`：`02:42:2a:00:00:02`。

修改前的纯 ADB 配置保存在 `/etc/init.d/.usb_config.before-adb-rndis`。修改 Gadget 功能或 MAC 后应完整断电冷启动；仅热重启 Gadget 时，已有 ConfigFS RNDIS 功能可能继续保留旧 MAC。

## 远程入口与监听

以下是盘点时对外监听的主要端口：

| 端口 | 进程 / 服务 | 范围与说明 |
|---|---|---|
| TCP 21 | vsftpd | IPv6 wildcard，同时可接受 IPv4；本地用户登录，无 TLS |
| TCP 22 | OpenSSH | IPv4/IPv6 wildcard；允许密码和公钥认证 |
| TCP 4000、UDP 4000 | NoMachine `nxd` | IPv4/IPv6 wildcard，远程桌面入口 |
| TCP 5555 | `adbd` | IPv4 wildcard；板端 ADB 守护进程以 root 运行 |
| TCP/UDP 9993 | ZeroTier | IPv4/IPv6 wildcard |
| TCP 7001、UDP 4011 | NoMachine `nxnode` | 活动虚拟桌面会话相关端口 |
| UDP 5353 | Avahi / NoMachine | mDNS / 服务发现 |
| TCP/UDP 53 | systemd-resolved | 仅 `127.0.0.53` |
| TCP 631 | CUPS | 仅回环地址 |

NoMachine 和活动会话还使用若干仅绑定 `127.0.0.1` 的动态内部端口。

## 关键运行服务

当前确认运行且与板子用途直接相关的服务包括：

- `NetworkManager`、`systemd-resolved`、`ntp`；
- `usbdevice`、`adbd`；
- `ssh`、`vsftpd`、`zerotier-one`；
- `nxserver` / `nxd`；
- `avahi-daemon`；
- `lbc-test`、`rockchip`/`rkaiq_3A` 等板级组件中的已启用单元。

NoMachine 无头配置：

- `/usr/NX/etc/node.cfg` 使用 Ubuntu GNOME 默认会话；
- `VirtualDesktopVariables` 指向 `/usr/local/bin/nomachine-virtual-env.sh`；
- 虚拟桌面单独使用 Mesa llvmpipe，避免无物理显示器时 Mali EGL 初始化失败；
- 默认启动到 `multi-user.target`，不自动运行 GDM。

## 安全状态

这是状态记录，不代表已经执行加固：

- `ufw.service` 为 active/enabled，但 `ufw status` 为 inactive，因此当前没有 UFW 规则实际过滤上述监听端口。
- SSH 有效配置为 `PasswordAuthentication yes`、`PubkeyAuthentication yes`；root 只允许密钥方式登录（`PermitRootLogin without-password`）。
- vsftpd 禁止匿名登录，允许本地用户登录，但 `ssl_enable=NO`，FTP 凭据和数据不加密。
- ADB TCP 5555 对所有 IPv4 接口监听，且 `adbd` 以 root 运行。

在 USB 直连以外的网络使用此板子前，应按实际需求决定是否关闭 FTP/ADB TCP、限制监听地址并启用防火墙。本次盘点没有改变这些服务。

## 核对命令

~~~bash
ip -brief address
ip route
nmcli connection show LubanCat-PC
nmcli connection show Phone-USB
systemctl --type=service --state=running
ss -lntup
ufw status verbose
/usr/NX/bin/nxserver --status
~~~
