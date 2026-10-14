# 真实推理与运行诊断

本项目在 RTX 4060 Laptop GPU 上完成 **单图 full、双图 camera-depth、双图 full** 三次推理。
最新双图 full 首次前向为 **1.731061 秒**，PyTorch 峰值 allocated/reserved 为 **5.237339 / 5.833984 GiB**。
三次均通过输出布局和有限值检查；这些结果说明执行链路可用，尚不提供相机、深度或点图的真值精度。

## 输入、源码与权重

| 项目 | 实际运行记录 |
|---|---|
| 图像 | 官方 VGGT kitchen 示例前两张照片；黄色积木推土机与桌面 |
| 原始 / 处理后 H×W | 520×779 / 350×518 |
| 输入顺序 | 单图使用 0；双图按 0、1 联合输入 |
| 预处理 | 官方 crop helper；本组仅缩放，无实际裁切或 padding |
| 源码 commit | `a288dd0f14786c93483e45524328726ab7b1b4ce` |
| 权重 | `facebook/VGGT-1B`，`model.safetensors` |
| 权重 revision | `860abec7937da0a4c03c41d3c269c366e82abdf9` |
| 权重文件大小 | 5,026,367,224 bytes；未随本仓库分发 |
| 环境 | Python 3.11.17；torch 2.3.1+cu121；torchvision 0.18.1+cu121；NumPy 1.26.1 |
| 设备 | NVIDIA GeForce RTX 4060 Laptop GPU，约 8 GiB 显存 |

输入顺序、尺寸和哈希见[输入清单](results/inference/input_manifest.json)，源码与权重核验见各运行报告。
权重 SHA-256 为 `f164acf60724910d8fe1578bb499d800850c7bb0948db7555c413f9fbe60467e`。
照片的主体与桌面明显重叠，但没有相机运动或场景静态性真值；`scene_geometry_verified=false` 不应改写成已标定场景。

## 三次成功运行的配置与耗时

三份报告的顶层均为 `status=success`、`stage=completed`。
所有耗时来自首次运行记录，没有 warm-up 后重复计时；完整流程不含权重下载与输入准备。

| 条件 | 首次 forward / s | 整流程 / s | peak allocated / GiB | peak reserved / GiB |
|---|---:|---:|---:|---:|
| [single_full](results/inference/single_full/run.json) | 1.593880 | 26.563233 | 5.117156 | 5.457031 |
| [pair_camera_depth](results/inference/pair_camera_depth/run.json) | 1.052363 | 30.140397 | 4.776446 | 5.125000 |
| [pair_full](results/inference/pair_full/run.json) | 1.731061 | 31.507797 | 5.237339 | 5.833984 |

显存取 `memory.forward_peak_and_end.max_allocated_bytes/max_reserved_bytes`，除以 `2^30`，峰值包含模型参数。
allocated 是 PyTorch 张量分配量，reserved 是缓存分配器保留量，均不等于全卡所有进程占用。
full 与 camera-depth 的配置不同，不能用表中时间差推导增加视图的普遍成本或稳定吞吐量。

| 精度与执行设置 | 三次运行的共同状态 |
|---|---|
| `eval()` / `inference_mode()` | 均启用；评估行为与关闭自动求导分别控制 |
| 参数存储 | FP32 |
| autocast | BF16，`cache_enabled=false` |
| 原始输出 / 导出数组 | torch.float32 / float32，原始输出不需要梯度 |
| 查询点 | 未提供，未产生跟踪结果 |

官方相机、深度与点图头在相应区域关闭 autocast，因此不能由 BF16 autocast 推断所有结果都是 BF16。
camera-depth 在严格加载全部权重后移除 point/track head，再转入 GPU；两张图仍共同经过 aggregator。
其 depth head `frames_chunk_size=1`，full 为 8。该后备配置没有把两图拆成两次独立单图预测。

## 保存了哪些实际输出

最新双图 full 的 `layout` 为 B=1、S=2、H=350、W=518；保留 batch 和 views 两轴。

| 字段 | 实际 shape | 保存位置 |
|---|---|---|
| `pose_enc` | `(1,2,9)` | `raw_predictions.npz` |
| `depth` | `(1,2,350,518,1)` | `raw_predictions.npz` |
| `depth_conf` | `(1,2,350,518)` | `raw_predictions.npz` |
| `world_points` | `(1,2,350,518,3)` | `raw_predictions.npz` |
| `world_points_conf` | `(1,2,350,518)` | `raw_predictions.npz` |
| `images` | `(1,2,3,350,518)` | `raw_predictions.npz` |
| `w2c` / `K` | `(1,2,3,4)` / `(1,2,3,3)` | `cameras.npz`，CPU 解码结果 |

全部字段有限。双图 full 的 362,600 个深度均为正，最小/最大/均值为 **0.527382 / 2.654784 / 1.129843**，单位未标定为米。
相机编码迭代共 4 项，每项 `(1,2,9)`；最后一项与 `pose_enc` 完全一致。
证据见[运行报告](results/inference/pair_full/run.json)与[CPU 输出检查](results/inference/pair_full/inspection/inspection.json)。

camera-depth 的 raw 输出只有 `pose_enc、depth、depth_conf、images`，缺少点图是预期行为。
单图 full 使用同类布局，将 S 改为 1；三次均没有 `track/vis/conf`。
w2c 满足 `Pc=R Pw+t`，K 对应 350×518 网格；第一相机近似 `[I|0]` 是预测结果，没有手工覆盖。

## 图像预览的含义

| 双图 full 的输入视图 0 | 对应深度预览 |
|---|---|
| ![处理后的输入 RGB](results/inference/pair_full/processed_rgb/00.png) | ![深度预览](results/inference/pair_full/inspection/depth_00.png) |

RGB 是处理后的观测图像，不是模型生成的新视图。
深度显示在同一 run 的全部视图上共同取 2%/98% 分位数，线性映射到灰度，近暗远亮；双图 full 的显示端点为 **0.606602 / 2.294410**。
PNG 经过截断与 8 位量化，后续几何计算必须读取浮点数组；不同 run 分别设定显示范围，灰度不能直接跨 run 比较。

## 权重加载与运行故障的工程处理

Windows 整文件权重加载曾触发页面文件相关错误。最终加载器在核验整文件哈希后，严格检查键集合、shape、dtype 与完整字节区间，再逐张量读取并复制到模型存储。
成功报告记录 `storage_backend=bounded_file_reads`、`tensors_loaded=1797`、`largest_tensor_bytes=67108864`，即最大的单张量缓冲为 64 MiB。
这避免了整文件映射和完整 checkpoint 字典的额外占用，没有量化或更改原始 FP32 权重。

[加载器测试记录](results/inference/evidence/checkpoint_loading_tests.txt)为 **3/3 通过**，覆盖与标准严格加载的数值一致性及非法字段/字节区间拒绝。
[当前 CPU 审计函数检查](results/inference/evidence/day3_check_recheck_20261012.json)为 **16/16 通过**，其范围是统计与布局函数，不是 GPU 推理精度。
历史运行还遇到过路径、原生进程、CUDA 与 cuDNN 故障；这些错误出现在不同阶段，不应全部归因为显存不足。

双图 full 的 Windows GPU 进程计数器采样共 97 次，专用内存峰值约 5.897 GiB、共享内存峰值约 0.135 GiB，见[采样摘要](results/inference/evidence/pair_full_gpu_memory_summary.json)。
这与 PyTorch 分配器是不同口径；初始化阶段已有共享内存，采样也可能遗漏短峰，因此不能据此证明系统内存回退导致运行成功。
本项目未做开关回退的同条件对照，也未独立核验驱动设置。

## 这组运行能支持什么结论

源码、输入和权重核验之后，三种配置完成了前向、导出、布局检查、有限值审计与相机解码。
报告明确 `numeric_contract_passed=true`、`geometry_accuracy_verified=false`：数值与布局检查不能替代真值误差、遮挡处理或米制尺度验证。
后续几何一致性实验应使用同一 run 的深度、内参与外参，并单独报告其指标与适用范围。

[返回项目总览](README.md)
