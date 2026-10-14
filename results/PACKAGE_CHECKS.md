# 整理与验证记录

[返回项目总览](../README.md) · [来源索引](../SOURCE_INDEX.md)

整理日期：2026-10-14（Asia/Shanghai）。本次验证分为历史运行证据、归档完整性和独立数值复算，结果见[verification.json](package_validation/verification.json)。新校验仅使用展示包内文件，不加载 VGGT、不读取模型权重，也不依赖原工作区绝对路径。

## 历史接口与运行证据

以下是原实验已保存的检查结果，本次整理未重跑这些教学检查器。其时间和源码指纹保留在对应报告中。

| 模块 | 通过 / 总数 | 证据 |
|---|---:|---|
| 输入输出契约 | 16 / 16 | [contracts/checks.json](contracts/checks.json) |
| 预处理、像素仿射与注意力分组 | 24 / 24 | [preprocessing/checks.json](preprocessing/checks.json) |
| 推理输出审计函数 | 16 / 16 | [推理检查复查](inference/evidence/day3_check_recheck_20261012.json) |
| 相机、反投影与回投 | 24 / 24 | [几何检查](geometry/evidence/day4_check.json) |
| 两路点图与筛选 | 24 / 24 | [筛选检查](confidence/evidence/day5_check.json) |
| 跨视图分类与汇总 | 24 / 24 | [跨视图检查](cross_view/evidence/day6_check.json) |
| 独立几何接口 | 24 / 24 | [接口检查](validation/day7_check.json) |
| **合计** | **152 / 152** | 各模块的解析或合成输入检查 |

这些检查不等于 152 个真实场景实验。正式推理由[单图 full](inference/single_full/run.json)、[双图 camera-depth](inference/pair_camera_depth/run.json)和[双图 full](inference/pair_full/run.json)分别记录；权重加载器另有 [3/3 测试记录](inference/evidence/checkpoint_loading_tests.txt)。

统一几何接口的[构建验收](validation/completion.json)记录与历史几何、mask 和 18 行方向汇总一致；[独立读取验收](validation/packet_check.json)记录隔离新进程的 6 类检查通过。独立读取验收的范围是导出完整性和内部关系，不包含未随 packet 导出的原始残差重算。

## 本次归档完整性检查

整理时使用本地 `verify_artifacts.py` 逐项检查。该脚本不随仓库分发，下文命令仅记录本地验证过程：

- 183 个复制文件的大小与 SHA-256，包括 13 张精选图；全部与归档清单一致。
- Markdown 和 HTML 图片的包内链接目标；全部存在。外部网络链接和标题锚点不属于该脚本检查范围。
- 所有包内 JSON 可解析；没有达到 50 MiB 的单文件。最大的跨视图数组约 22.46 MiB。
- `.gitattributes` 设置禁止自动行尾转换，使历史文本与二进制结果跨平台检出后仍能按原字节核对。

命令：

```powershell
python verify_artifacts.py
```

默认只读，不创建或更新文件。检查记录只证明当前包与声明清单一致；不能证明历史实验从未被修改。

## 独立数值复算

数值复算使用 NumPy，读取归档中的浮点数组，既不导入原始练习模块，也不使用 PNG 作为计算输入。

| 对象 | 独立复核内容 | 结果 |
|---|---|---|
| 统一几何接口 | 17 个字段的 shape/dtype、源预测身份、K/w2c、相机中心、c2w、世界点与有效性 | 通过 |
| 置信度支持 | 逐视图分位数、阈值并列、三档两路线 mask、交集与覆盖率 | 通过 |
| 点图差异 | 从原始点重新计算距离，核对 27 行单视图/合并汇总 | 通过 |
| 跨视图 | 重新投影世界点、判断边界、最近像素采样、重建支持、残差与七类标签，核对 18 行方向统计 | 通过 |
| pooled | 拼接两个方向的原始可比较查询，重算全部 9 组汇总，含默认 high75 / 5% | 通过 |

浮点摘要比较使用 `rtol=1e-8, atol=1e-10`；一般数组比较使用 `rtol=1e-8, atol=1e-9`；旋转关系另用 `atol=1e-4`，详见脚本及结果中的容差记录。

实际使用 Python 3.13 和 NumPy 2.4.0 完成 CPU 复算。这与历史模型推理环境不同；检查的是保存数组的数值关系，不是重放历史 GPU 算子。

```powershell
python verify_artifacts.py --recompute-metrics --output results/package_validation/verification.json
```

`--output` 显式保存本次 JSON，其余输入不修改。本次所有检查通过，详细计数、复算数值和空失败列表见[最终报告](package_validation/verification.json)。

## 结论的实际范围

复制证据的哈希、独立几何计算和统计复算分别验证文件身份、接口关系和汇总口径。它们不能单独验证真实相机位姿、米制尺度、真实表面深度、遮挡真值或跨场景泛化。

几何有效点、置信度支持点、两路共同点、跨视图可比较查询和全图像素拥有不同分母。正文按各自含义报告，未把集合内一致率写成整图准确率，也未把有向查询数写成唯一三维表面数。
