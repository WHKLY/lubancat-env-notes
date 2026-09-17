# 项目与构建产物

快照日期：2026-09-17。

## 目录总览

| 路径 | 大小 | 识别结果 |
|---|---:|---|
| `/home/cat/rknn-toolkit2_backup/rknn-toolkit2-master` | 2.1 GB | RKNN Toolkit2 2.3.2 完整快照，含 Lite wheels、rknpu2、文档和多平台示例 |
| `/home/cat/hit` | 48 MB | 自定义视觉推理脚本、模型、测试图和输出结果 |
| `/home/cat/rknn_yolov10_demo` | 23 MB | 可直接运行的 YOLOv10 C++ 部署包 |
| `/home/cat/Desktop/rknn_yolov10_demo` | 22 MB | 上述部署包副本，不含根目录的 `out.png` |
| `/home/cat/rknpu2` | 16 MB | 独立的 RKNN Runtime 与 C/C++ 示例快照 |
| `/home/cat/rknn_model/rknn_model_zoo` | 15 MB | RKNN Model Zoo 部分快照与公共工具 |
| `/home/cat/yolov10-main(airochip)/yolov10-main` | 5.6 MB | Ultralytics/YOLOv10 源码快照，内部版本 8.1.34 |
| `/home/cat/0718_final.rknn` | 15.7 MiB | 自定义 RKNN 模型，由 `hit/yolov10_rknn_integrated.py` 默认加载 |

`/home/cat`、`/opt` 和 `/usr/local/src` 中没有发现 `.git` 目录；板上也没有 Git 命令。因此这些目录是无版本历史的文件快照，无法仅凭板端确认上游提交、分支或本地修改范围。

## RKNN Toolkit 与 Runtime

`rknn-toolkit2_backup/rknn-toolkit2-master` 的 `CHANGELOG.md` 最高版本为 2.3.2，且包含适配 CPython 3.8/aarch64 的：

~~~text
rknn-toolkit-lite2/packages/
  rknn_toolkit_lite2-2.3.2-cp38-cp38-manylinux_2_17_aarch64.manylinux2014_aarch64.whl
~~~

同一快照也包含面向其他 Python 版本、x86_64 转换工具、Android/Linux 多架构库和大量示例模型。这解释了其 2.1 GB 占用；板端当前实际安装的只有 Lite 2.3.2 推理包。

`/home/cat/rknpu2` 包含：

- Linux/Android RKNN Runtime；
- `rknn_server` 与 `librknn_api`；
- API、benchmark、custom op、dynamic shape、matmul、MobileNet、YOLOv5 和 zero-copy 等示例；
- 一个简短的 `rknpu.mk`。

## Model Zoo 与 YOLO 源码

`/home/cat/rknn_model/rknn_model_zoo` 是部分 Model Zoo 快照，当前顶层示例只有：

- `yolov8_obb`；
- `yolov8_pose`；
- `yolov8_seg`；
- `yolox`；
- `zipformer`。

目录还带有 `build-linux.sh`、`build-android.sh`、公共 C/C++ 图像/音频工具、Python executor，以及一份 Ultralytics 8.1.34 源码树。

`/home/cat/yolov10-main(airochip)/yolov10-main` 也是 Ultralytics 8.1.34 源码快照，包含 pyproject、测试、Dockerfile 和 RKNN 说明。它没有安装进当前 Python 环境；从普通工作目录执行 `import ultralytics` 会失败。板端也没有 PyTorch，不能按完整 Ultralytics 训练/导出流程直接运行。

## YOLOv10 部署包

`/home/cat/rknn_yolov10_demo` 包含：

- ARM64 可执行程序 `rknn_yolov10_demo`；
- RKNN Runtime 与 RGA 动态库；
- `yolov10n.rknn`、`0717.rknn`；
- COCO 80 类标签与测试图 `bus.jpg`；
- 一次输出图 `out.png`。

桌面副本中的程序、模型和库与主目录对应文件一致；两目录唯一检测到的差异是主目录额外存在 `out.png`。建议将 `/home/cat/rknn_yolov10_demo` 视为主副本，避免两边分别修改。

## 自定义 `hit` 工程

根据脚本名、导入项和默认模型路径推断，`/home/cat/hit` 当前包含：

| 入口 | 默认模型 / 用途 |
|---|---|
| `yolov10-rknn.py` | YOLOv10 RKNN 推理，复用 `py_utils/rknn_executor.py` |
| `yolov10_rknn_integrated.py` | 加载 `/home/cat/0718_final.rknn` 的集成版检测流程 |
| `yolov8_pose.py` | 加载 `hit/yolov8_pose.rknn` 的姿态估计 |
| `yolov8_pose_integrated.py` | 姿态估计集成流程 |
| `doctor.py` | 同样加载 `yolov8_pose.rknn` 的自定义处理脚本 |
| `resnet.py` | 加载 `my_weight/resnet18_0508_31.rknn` 的分类流程 |
| `lprnet.py` | 默认加载 `my_weight/lenet.rknn` 的车牌字符识别流程 |

目录中的 `1/`、`2/`、`test/` 是输入/测试图片，`result/` 是历史输出，`lib/` 包含与系统相同版本的 RKNN Runtime 以及 RGA 库。本次只识别入口和依赖，没有验证每个脚本的业务正确性。

## 主要产物指纹

| 文件 | 大小 | 修改时间 | SHA-256 |
|---|---:|---|---|
| `0718_final.rknn` | 16,508,233 B | 2026-07-19 | `16a29e9d154873b46a537afdfec6b17cca81b3c1bacc42d3cd299281eb815b37` |
| `rknn_yolov10_demo/model/0717.rknn` | 9,840,526 B | 2026-07-18 | `aeb775edb37723ae737a4981171cbd8703184b792631dac844a2520f7fe45772` |
| `rknn_yolov10_demo/model/yolov10n.rknn` | 3,786,503 B | 2026-07-21 | `089375973f5f6566f77ff34517be84883ce59ebe85e320d55fe960ec67854a3d` |
| `hit/yolov8_pose.rknn` | 8,538,486 B | 2026-07-24 | `0be75b459c7e69886d25624e33e3a46f973dc7a71665b2fb22ec66aa522c0c8b` |
| `hit/my_weight/lenet.rknn` | 181,805 B | 2026-08-04 | `b9f9d21ea13816b2bcf9657c4c10ae00f4eb0bb7e81656a798c8cc804aef5ab3` |
| `hit/my_weight/resnet18_0508_31.rknn` | 22,555,245 B | 2026-08-04 | `983afeb91b911ee9a0a26263f598a90c2bf32dd804700ec5b5df563828c9fbdb` |
| `rknn_yolov10_demo/rknn_yolov10_demo` | 1,000,448 B | 2026-07-21 | `4d9c5421c88d5e5066550d35a36e783a96e9bbbade9830c066e5557281657477` |

`0718_final.rknn`、YOLOv10 两个模型和演示程序当前权限为 777，即所有本地用户均可修改；`hit` 下其他模型多为 664。状态已记录但未改动。

## 完整性核对示例

~~~bash
sha256sum \
  /home/cat/0718_final.rknn \
  /home/cat/hit/yolov8_pose.rknn \
  /home/cat/hit/my_weight/lenet.rknn \
  /home/cat/hit/my_weight/resnet18_0508_31.rknn \
  /home/cat/rknn_yolov10_demo/model/*.rknn \
  /home/cat/rknn_yolov10_demo/rknn_yolov10_demo
~~~
