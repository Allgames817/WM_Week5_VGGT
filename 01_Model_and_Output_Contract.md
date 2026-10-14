# VGGT 架构与输出契约

VGGT 从同一场景的一组 RGB 图像联合预测相机、深度和点图；给定查询点时还可预测二维跟踪。
本项目先用 **16/16 项 NumPy 检查**明确输入与张量布局，再完成单图和双图真实推理。
接口检查、源码阅读和实际模型输出是三类证据，分别说明“怎样接入”“代码怎样组织”和“本机运行得到了什么”。

## 从观测图像到几何预测

```text
同一场景的 RGB 图像
        │
resize / crop / pad → [0,1] 浮点输入
        │
patch 特征 + camera / register token
        │
Frame / Global 交替注意力
        │
        ├── camera head → pose_enc → w2c、K
        ├── depth head  → depth、depth_conf
        ├── point head  → world_points、world_points_conf
        └── track head  → track、vis、conf（需要 query_points）
```

本项目锁定的官方源码 commit 为 `a288dd0f14786c93483e45524328726ab7b1b4ce`，见[版本锁](results/contracts/repo_lock.json)。
网络内部执行图像通道标准化；外部输入只需处于 `[0,1]`，不应重复减均值、除标准差。
注意力的形状追踪与局限见[图像预处理与注意力](02_Image_Preprocessing_and_Attention.md)。

## 输入的两个层级

| 输入形式 | 解释 | 模型中的统一形状 |
|---|---|---|
| `(S,3,H,W)` | 一个场景的 S 张图像 | 自动补成 `(1,S,3,H,W)` |
| `(B,S,3,H,W)` | B 个独立图像组，每组 S 张图像 | 保留 B 与 S 两个维度 |

B 是独立样本维度，S 是同一样本的视图维度。把两张照片分别推理后拼接，不等于双图联合推理。
本地教学契约要求浮点 NumPy 数组、有限数值、RGB 通道、非空轴和可被 14 整除的 H/W；拒绝非法输入而不自动裁剪数值。
这些是教学函数的检查规则，不能将其全部视为官方 `forward()` 已实现的输入验证。

## 原始输出与出现条件

以下按默认各预测头启用描述。相机矩阵是后处理结果，不属于原始 `forward()` 字典。

| 字段 | 形状 | 含义与条件 |
|---|---|---|
| `pose_enc` | `(B,S,9)` | 最后一轮相机编码；不是外参矩阵 |
| `pose_enc_list` | 列表，每项 `(B,S,9)` | 相机头各轮预测；本项目真实运行保存了 4 项 |
| `depth` | `(B,S,H,W,1)` | 各视图相机系的 Z 深度 |
| `depth_conf` | `(B,S,H,W)` | 深度分支的稠密置信度分数 |
| `world_points` | `(B,S,H,W,3)` | 点图头直接预测的共享参考系 XYZ |
| `world_points_conf` | `(B,S,H,W)` | 直接点图的稠密置信度分数 |
| `track` | `(B,S,N,2)` | N 个查询点在各视图的二维像素坐标；需要查询点 |
| `vis` / `conf` | 各 `(B,S,N)` | 跟踪的可见性与置信度；需要查询点 |
| `images` | `(B,S,3,H,W)` | 评估模式保留的预处理后输入图像 |

教学 shape 函数只返回单个张量的 shape，因此排除了列表 `pose_enc_list`；这不表示模型没有该字段。
关闭某个预测头时，对应字段也不出现。`camera-depth` 配置没有直接点图，而三次实际推理均未提供查询点，因此没有跟踪输出。
真实字段清单见[推理与工程诊断](03_Inference_and_Runtime_Diagnostics.md)和[双图 full 报告](results/inference/pair_full/run.json)。

## 相机编码怎样成为矩阵

源码按 `t_x,t_y,t_z,q_x,q_y,q_z,q_w,fov_h,fov_w` 排列 9 维相机编码，四元数实部在最后。
不能根据论文中的概括性符号顺序直接切片。
解码得到 OpenCV world-to-camera 外参 `(B,S,3,4)` 与像素单位内参 `(B,S,3,3)`：

```math
\mathbf P_c=R\mathbf P_w+\mathbf t,\qquad
\mathbf C_w=-R^\top\mathbf t.
```

平移列 t 不等于世界系相机中心。需要 c2w 时应显式求逆，旋转为 R 的转置，平移为上述相机中心。
相机轴采用 x 向右、y 向下、z 向前；内参解码使用实际进入模型的 H/W：

```math
f_x=\frac{W/2}{\tan(fov_w/2)},\qquad
f_y=\frac{H/2}{\tan(fov_h/2)},\qquad
(c_x,c_y)=(W/2,H/2).
```

双图实际输入为 350 × 518，因此 K 的主点为 `(259,175)`。不能把原照片 520 × 779 的高宽直接代入这份输出。
解码契约见[双图报告的 camera_contract](results/inference/pair_full/run.json)，像素坐标变化见[预处理专题](02_Image_Preprocessing_and_Attention.md)。

## 深度、点图和置信度的语义

对零 skew 针孔相机，深度反投影与直接点图是不同计算路径：

```math
\mathbf P_c=ZK^{-1}[u,v,1]^\top,\qquad
\mathbf P_w=R^\top(\mathbf P_c-\mathbf t).
```

这里 Z 是相机轴向深度；欧氏 range 为 `Z × ||K^{-1}[u,v,1]ᵀ||`，离开主光轴后通常大于 Z。
`world_points` 则由 point head 直接预测，两条路径不保证逐点相同；它们的后续一致性需单独核对。
共享参考系也不等于已标定的米制世界，不同输入批次不能仅凭字段同名就拼成同一张地图。

稠密深度/点图置信度默认使用 `1 + exp(raw)`，因而可以大于 1，不能解释为百分比正确率或直接裁到 `[0,1]`。
这一激活结论不应直接套到 track head 的 `vis/conf`。

## 已完成的契约检查与能力边界

[检查报告](results/contracts/checks.json)记录 `passed=16`、`total=16`：8 项输入检查覆盖维度、数值、类型与 patch 对齐，8 项 schema 检查覆盖字段集合、布局、查询点与训练模式条件。
报告同时明确 `official_model_executed=false`、`model_accuracy_verified=false`；这些合成输入检查没有加载权重。
真实推理在独立报告中保存，不能把“shape 符合预期”当作相机或三维点真值准确的证明。

VGGT 在本项目中的作用是估计观测图像的几何关系。可控新视图渲染仍需要场景表示和渲染器；长期空间记忆还需要配准、存储与更新；动作条件交互还需要状态转移模型和闭环评估。
一次前向返回的几何字典本身不提供这些能力。

[返回项目总览](README.md)
