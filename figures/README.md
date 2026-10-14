# 精选图索引

[返回项目总览](../README.md)

这里的 13 张图是原实验预览的字节级副本。精确源路径和 SHA-256 见[归档清单](../results/source_manifest.json)，完整预览仍在各专题 results 目录中。

| 精选图 | 含义 | 显示与统计边界 |
|---|---|---|
| [input_view_00](input_view_00.png)、[input_view_01](input_view_01.png) | 双图 full 的处理后输入 | 350×518 观测 RGB，不是生成图像 |
| [depth_view_00](depth_view_00.png)、[depth_view_01](depth_view_01.png) | 预测 Z 深度 | 共同 2%/98% 分位数范围、近暗远亮、8 位显示；[显示记录](../results/inference/pair_full/inspection/inspection.json) |
| [geometry_scene](geometry_scene.png) | 深度反投影点与相机 | stride=8，每图 2,860 点；模型单位、未融合；[预览元数据](../results/geometry/preview/preview.json) |
| [point_maps_all](point_maps_all.png)、[point_maps_high50](point_maps_high50.png) | 两路线点图对照 | 相同坐标范围；每路线分别抽样 5,720 / 1,997 点；[预览元数据](../results/confidence/preview/preview.json) |
| [support_high50_view_00](support_high50_view_00.png)、[support_high50_view_01](support_high50_view_01.png) | high50 两路共同支持 mask | 白色为保留像素，不能解释为语义标签或真值可见性 |
| [cross_view_00_to_01](cross_view_00_to_01.png)、[cross_view_01_to_00](cross_view_01_to_00.png) | 源点在目标图中的诊断类别 | high75 / 5%，stride=12 仅供显示；[预览元数据](../results/cross_view/preview/high75_r05/preview.json) |
| [preprocess_crop](preprocess_crop.png)、[preprocess_pad](preprocess_pad.png) | 竖向诊断图的两种预处理 | 合成测试图，未用于真实场景推理 |

统计使用完整保存网格；PLY 和散点只是显示抽样。坐标距离为模型单位，相对深度差异为无量纲比值。
