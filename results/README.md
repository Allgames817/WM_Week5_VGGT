# 实验结果归档

[返回项目总览](../README.md) · [完整来源映射](../SOURCE_INDEX.md) · [本次校验](PACKAGE_CHECKS.md)

| 目录 | 保存内容 |
|---|---|
| `contracts/` | 环境探测、固定源码、输入输出检查 |
| `preprocessing/` | crop/pad 计划与像素对照、均值传播实验、合成图预览 |
| `inference/` | 三次成功运行的原始浮点预测、解码相机、相机编码迭代、图像和运行记录 |
| `geometry/` | c2w、相机中心、深度反投影、同视图回投、官方 helper 对照与反例 |
| `confidence/` | 三档两路 mask、同像素点图差异、27 行汇总与完整预览 |
| `cross_view/` | 投影中间量、标签、原始有向残差、18 行方向统计和显示图 |
| `geometry_packet/` | 统一的 17 字段 NPZ、来源 manifest、9 组双方向与 pooled 汇总 |
| `validation/` | 历史独立接口、同源构建与 packet 读取验收 |
| `package_validation/` | 本次展示包的独立完整性与数值复核 |

复制产物的字段、时间、文件名和绝对路径保持原值。阅读导航使用包内相对路径；原记录中的历史路径不要求在其他机器存在。

[source_manifest.json](source_manifest.json)记录复制文件的字节身份；[excluded_files.json](excluded_files.json)记录省略范围。完整推理实现、模型权重和原始输入图片副本未包含。本仓库可离线审阅和复算保存结果，不能直接复现模型推理或生成新场景预测。
