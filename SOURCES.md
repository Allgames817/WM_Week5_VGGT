# 方法、实现与图像来源

[返回项目总览](README.md)

实验结果以本仓库的本地运行记录为依据。论文和官方源码用于解释接口与方法，不以论文性能代替本机结果。

## 方法与固定源码

| 来源 | 本项目中的用途 |
|---|---|
| [VGGT 论文 arXiv v1](https://arxiv.org/abs/2503.11651v1) | Visual Geometry Grounded Transformer 的方法、交替注意力、多任务几何预测与共享参考系 |
| [官方 VGGT 固定 commit](https://github.com/facebookresearch/vggt/tree/a288dd0f14786c93483e45524328726ab7b1b4ce) | 本地实验锁定的源码版本；非随时间变化的 main |
| [VGGT.forward](https://github.com/facebookresearch/vggt/blob/a288dd0f14786c93483e45524328726ab7b1b4ce/vggt/models/vggt.py) | 输入布局、预测字段、query gate 与评估模式图像输出 |
| [Aggregator](https://github.com/facebookresearch/vggt/blob/a288dd0f14786c93483e45524328726ab7b1b4ce/vggt/models/aggregator.py) | 图像标准化、特殊 token、Frame/Global 交替与缓存特征 |
| [图像预处理](https://github.com/facebookresearch/vggt/blob/a288dd0f14786c93483e45524328726ab7b1b4ce/vggt/utils/load_fn.py) | resize、crop、pad、透明图像和组内统一尺寸 |
| [相机编码](https://github.com/facebookresearch/vggt/blob/a288dd0f14786c93483e45524328726ab7b1b4ce/vggt/utils/pose_enc.py) | 平移、xyzw 四元数、FoV 与 OpenCV w2c/K 解码 |
| [几何工具](https://github.com/facebookresearch/vggt/blob/a288dd0f14786c93483e45524328726ab7b1b4ce/vggt/utils/geometry.py) | Z 深度反投影与官方 helper 对照 |
| [预测头激活](https://github.com/facebookresearch/vggt/blob/a288dd0f14786c93483e45524328726ab7b1b4ce/vggt/heads/head_act.py) | 深度、点图及稠密置信度的实际激活定义 |

完整本地源码身份和静态接口指纹见[repo_lock.json](results/contracts/repo_lock.json)。本次整理核对了论文 v1 页面与固定 commit 的官方入口；接口细节以已经保存的锁定源码检查和本地内容为依据。

## 模型权重

使用原版 [facebook/VGGT-1B 固定 revision](https://huggingface.co/facebook/VGGT-1B/tree/860abec7937da0a4c03c41d3c269c366e82abdf9) 的 `model.safetensors`，未使用其他变体。

```text
revision: 860abec7937da0a4c03c41d3c269c366e82abdf9
bytes:    5026367224
SHA-256:  f164acf60724910d8fe1578bb499d800850c7bb0948db7555c413f9fbe60467e
```

核验记录见[weights_lock.json](results/inference/evidence/weights_lock.json)。展示包不分发权重，也不需要权重来复算已保存指标。

## 输入图像与展示图

两张照片来自固定源码中的 [kitchen 示例图像目录](https://github.com/facebookresearch/vggt/tree/a288dd0f14786c93483e45524328726ab7b1b4ce/examples/kitchen/images)，分别为 `00.png`、`01.png`。原工作区临时命名为 `input_raw/01.png`、`input_raw/02.png`，规范化后重新命名为 `inputs/00.png`、`inputs/01.png`；输入顺序和原始/规范化 SHA-256 见[输入清单](results/inference/input_manifest.json)。

展示包收录模型实际使用的处理后 RGB、预测深度、点图和诊断图。深度图经显示映射，几何预览经网格抽样；全部数值结论来自对应浮点数组或原始 JSON/CSV，不从 PNG 反推。图像出处和显示规则见[精选图索引](figures/README.md)。

上游照片、代码和权重的权利归各自权利人，本展示包未为第三方材料重新授予许可。

## 本实验与原论文的关系

本项目使用预训练模型，聚焦单场景双图输出的语义、工程执行和内部几何一致性。CPU 教学契约、均值注意力实验、置信度三档筛选、2%/5%/10% 相对容差和 geometry packet 均属于本项目的检查设计，不是论文全部 benchmark 的复现。

原始阅读材料、笔记和实现的指纹列在[source_manifest.json](results/source_manifest.json)中；按专题追溯产物请使用[来源索引](SOURCE_INDEX.md)。
