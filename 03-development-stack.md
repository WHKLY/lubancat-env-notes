# 开发与 RKNN 工具链

快照日期：2026-09-17。

## 基础工具

| 工具 | 当前版本 / 状态 |
|---|---|
| Bash | 5.0.17 |
| Python | 3.8.10，`/usr/bin/python3` |
| pip | 20.0.2 |
| GCC / G++ | 9.4.0 |
| CMake | 3.16.3 |
| GNU Make | 4.2.1 |
| build-essential | 12.8ubuntu1.1 |
| FFmpeg | 4.2.7 |
| GStreamer tools | 1.18.5 |
| Git | 未安装 |
| Ninja / Meson / Clang | 未检测到 |
| Java / Node.js / Go / Rust | 未检测到 |
| Docker / Podman | 未检测到 |
| ROS / colcon / catkin | 未检测到 |

板上没有发现位于 `rknn-toolkit2_backup` 之外的 `CMakeCache.txt`，因此当前保留的是部署产物和源码快照，没有可直接增量重建的本地 CMake build tree。

## Python 推理环境

`cat` 用户实际从 `/home/cat/.local/lib/python3.8/site-packages` 加载以下关键包：

| 包 | 版本 |
|---|---|
| rknn-toolkit-lite2 | 2.3.2 |
| opencv-python | 5.0.0.93；导入时报告 5.0.0 |
| NumPy | 1.24.4 |
| SciPy | 1.10.1 |
| Pillow | 7.0.0 |
| PyYAML | 5.3.1 |
| protobuf | 3.6.1 |
| psutil | 7.2.2 |

PyTorch、可导入的 `ultralytics` 和 ONNX Runtime 当前均不存在。系统 apt 同时安装了 Python OpenCV 4.2 和 NumPy 1.17.4，但用户 site-packages 中的 OpenCV 5.0.0 与 NumPy 1.24.4 优先被导入。

`/home/cat/.bashrc` 当前追加了：

~~~bash
export PYTHONPATH=$HOME/.local/lib/python3.9/site-packages:$PYTHONPATH
export PYTHONPATH=$HOME/.local/lib/python3.8/site-packages:$PYTHONPATH
export PYTHONPATH=$HOME/.local/lib/python3.8/site-packages:$PYTHONPATH
export PYTHONPATH=$HOME/.local/lib/python3.8/site-packages:$PYTHONPATH
export LD_LIBRARY_PATH=/usr/lib:$LD_LIBRARY_PATH
~~~

Python 3.8 路径重复三次，且板上没有对应的 `.local/lib/python3.9` 目录。这不影响当前导入，但会让环境来源不清晰；后续可在单独维护操作中去重，当前未修改。

## RKNN 运行栈

| 组件 | 当前值 |
|---|---|
| RKNN Lite Python 包 | 2.3.2 |
| `librknnrt.so` | 2.3.2，构建 `429f97ae6b@2025-04-09T09:09:27` |
| `rknn_server` | 2.3.2，构建日期 2025-03-30 |
| NPU Transfer | 2.2.2，构建日期 2024-06-18 |
| RKNPU 内核驱动 | 0.9.8，构建日期 2024-08-28 |
| RGA | 内核模块 1.2.27；用户包 2.2.0 |

系统库与工程目录内的 aarch64 RKNN Runtime 副本完全相同，SHA-256 均为：

~~~text
d31fc19c85b85f6091b2bd0f6af9d962d5264a4e410bfb536402ec92bac738e8
~~~

已核对的相同副本：

- `/lib/librknnrt.so`；
- `/home/cat/rknpu2/runtime/Linux/librknn_api/aarch64/librknnrt.so`；
- `/home/cat/rknn_yolov10_demo/lib/librknnrt.so`；
- `/home/cat/hit/lib/librknnrt.so`。

## YOLOv10 C++ 演示入口

`/home/cat/rknn_yolov10_demo/rknn_yolov10_demo` 是 ARM64、动态链接、未 strip 的 ELF 程序，直接依赖 `librknnrt.so`、libstdc++、libm、libgcc、pthread、libc 和 libdl；`ldd` 未发现缺失依赖。

随目录提供的 `requirements_cp38-2.3.2.txt` 实际不是 pip requirements，而是一段运行说明，其中路径仍写成 `/home/user`。按当前目录运行应使用：

~~~bash
cd /home/cat/rknn_yolov10_demo
export LD_LIBRARY_PATH="$PWD/lib${LD_LIBRARY_PATH:+:$LD_LIBRARY_PATH}"
./rknn_yolov10_demo model/yolov10n.rknn model/bus.jpg
~~~

上面仅记录入口；本次环境盘点没有再次执行模型推理。

## 版本核对

~~~bash
python3 --version
python3 -m pip show rknn-toolkit-lite2 opencv-python numpy scipy
gcc --version
cmake --version
cat /sys/kernel/debug/rknpu/version
strings /lib/librknnrt.so | grep 'librknnrt version'
ldd /home/cat/rknn_yolov10_demo/rknn_yolov10_demo
~~~

不要使用 `rknn_server --version` 探测版本：该程序会忽略该参数并直接启动传输服务。版本信息应从启动日志或二进制字符串读取。
