# 来源与证据索引

[返回项目总览](README.md) · [归档校验](results/PACKAGE_CHECKS.md)

公开文档与目录按技术主题组织。原始实验来自本地 `Daily/Week5_Day1` 至 `Daily/Week5_Day7`；复制的 JSON、CSV、日志、PNG 和 NPZ 保留原字节。原始文件名中的 Day 仅用于追溯。

| 展示主题 | 原始目录 | 核心证据 |
|---|---|---|
| 模型与接口 | `Week5_Day1` | [源码锁](results/contracts/repo_lock.json)、[环境](results/contracts/environment.json)、[16 项契约检查](results/contracts/checks.json) |
| 预处理与注意力 | `Week5_Day2` | [crop](results/preprocessing/preprocess_crop.json)、[pad](results/preprocessing/preprocess_pad.json)、[均值注意力实验](results/preprocessing/attention_demo.json)、[24 项检查](results/preprocessing/checks.json) |
| 正式推理 | `Week5_Day3` | [输入清单](results/inference/input_manifest.json)、[single full](results/inference/single_full/run.json)、[pair camera-depth](results/inference/pair_camera_depth/run.json)、[pair full](results/inference/pair_full/run.json) |
| 相机与深度 | `Week5_Day4` | [几何报告](results/geometry/analysis.json)、[官方 helper 对照](results/geometry/official_comparison.json)、[相机 CSV](results/geometry/cameras.csv)、[反例](results/geometry/evidence/counterexamples.json) |
| 点图与置信度 | `Week5_Day5` | [比较报告](results/confidence/comparison.json)、[27 行筛选统计](results/confidence/selection_summary.csv)、[筛选实验](results/confidence/evidence/mask_experiments.json) |
| 跨视图诊断 | `Week5_Day6` | [一致性报告](results/cross_view/consistency.json)、[18 行方向统计](results/cross_view/direction_summary.csv)、[可见性反例](results/cross_view/evidence/visibility_cases.json) |
| 统一接口与验收 | `Week5_Day7` | [packet manifest](results/geometry_packet/geometry_packet_manifest.json)、[双方向与 pooled](results/geometry_packet/cross_view_summary.json)、[构建验收](results/validation/completion.json)、[独立读取验收](results/validation/packet_check.json) |

## 目录映射与运行选择

| 原始相对路径 | 本仓库路径 |
|---|---|
| `Week5_Day1/evidence/` | `results/contracts/`，其中 `day1_check.json` 改名为 `checks.json` |
| `Week5_Day2/evidence/`、`artifacts/` | `results/preprocessing/`，其中 `day2_check.json` 改名为 `checks.json` |
| `Week5_Day3/artifacts/` | `results/inference/`；三次成功运行完整保留 |
| `Week5_Day3/evidence/` 的 JSON/TXT/JSONL | `results/inference/evidence/` |
| `Week5_Day4/artifacts/pair_geometry/` | `results/geometry/` |
| `Week5_Day5/artifacts/pair_point_comparison/` | `results/confidence/` |
| `Week5_Day6/artifacts/pair_cross_view/` | `results/cross_view/` |
| `Week5_Day4/5/6/evidence/` | 对应专题的 `evidence/` |
| `Week5_Day7/artifacts/geometry_packet/`、`evidence/` | `results/geometry_packet/`、`results/validation/` |

几何、置信度、跨视图和 packet 均以同一个成功 `pair_full` 为源，没有将不同 run 的预测拼接。`single_full` 和 `pair_camera_depth` 只用于展示实际运行与配置边界。

推理审计采用后续 [day3_check_recheck_20261012.json](results/inference/evidence/day3_check_recheck_20261012.json) 的 16/16 结果，同时保留[早期检查](results/inference/evidence/day3_check.json)和[恢复报告](results/inference/evidence/recovery_report.json)。详细失败运行目录未重复收录。

原始 `templates/week5_summary.md` 与 `templates/week6_bridge.md` 已填写真实结果，是本次专题改写的输入。整理依据其实际内容和独立产物，而非根据 templates 目录名判断完成状态。

## 文件身份与复核边界

[source_manifest.json](results/source_manifest.json) 的 `copied_files` 逐项记录源路径、目标路径、字节数、SHA-256 和用途；精选图也是原预览的字节级副本。`documentation_inputs_not_distributed` 保存原工作区说明、笔记、模板与代码的文件身份，供需要原实现时追溯。

[excluded_files.json](results/excluded_files.json) 说明未收录的原始源码、权重、输入图片副本、环境、缓存和详细失败目录。历史 JSON/TXT 中保留的本机绝对路径是当时的来源记录，本仓库的独立校验不依赖这些路径。

历史 `model_executed=false` 要按记录阶段解释：静态源码检查、CPU 后处理或 packet 验收没有运行模型，不否定[成功推理报告](results/inference/pair_full/run.json)的顶层完成状态。`geometry_accuracy_verified=false` 则保持其含义：本项目没有外部几何真值验证。

本次新增校验见[verification.json](results/package_validation/verification.json)。哈希支持文件身份核对；独立数值复算支持保存数组与汇总一致。二者均不证明预测与真实场景一致，也不替代完整模型复现。
