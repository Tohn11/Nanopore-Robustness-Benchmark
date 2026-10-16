# Nanopore Robustness Benchmark

纳米孔测序电信号鲁棒性评测基准（Nanopore Robustness Benchmark），用于评估和比较**基座模型（Foundation Model）** 与 **端到端模型（End-to-End Model）** 在不同电信号扰动条件下的鲁棒性与适应能力。

## 1. 项目简介

本项目围绕纳米孔测序电信号中的典型扰动，构建统一的扰动生成与模型评测流程，比较不同类型 Basecaller 在扰动前后的表现。

> 当前文档为 Benchmark 设计草案，具体实验流程、参数和模型清单将在实现过程中逐步完善。

## 2. 评测设计

| 项目 | 设置 |
| --- | --- |
| 评测结果汇总粒度 | **Chunk-level**（按 chunk 统计与报告） |
| 扰动施加粒度 | **Read-level**（在 read 层面选择和组织扰动） |
| 数据来源 | S1 体系，HG002 |
| 对比模型 | 端到端 Basecaller、基座模型 |
| 主要指标 | ACC、Coverage、是否比对到参考基因组 |

**粒度说明：** Read-level 指扰动的选取和应用以 read 为单位管理；具体扰动操作可只作用于 read 内的局部区间或 chunk。Chunk-level 指最终指标按 chunk 汇总。需要在评测实现中明确 chunk 切分长度、步长与 read/chunk 对应关系，以保证结果可复现。

## 3. 信号扰动类型

1. **时间轴拉伸（Time-axis stretching）**：在 read 内选定的 chunk 中进行局部时间轴拉伸，模拟信号时序变化。
2. **基线漂移（Baseline drift）**：不对整条 read 施加统一偏移；从 read 中随机抽取样本，并在所选 read 的局部区间施加基线漂移，以降低全 read 归一化消除扰动的可能性。需进一步明确与实际使用的标准化流程之间的关系。
3. **随机噪声（Random noise）**：向电信号加入随机噪声，噪声分布与强度由配置文件指定。
4. **复合扰动（Combined perturbations）**：组合上述三类扰动，评估模型在多因素同时作用下的鲁棒性。

扰动强度、随机种子、抽样比例、作用区间及组合顺序将在 `configs/` 中统一管理。

## 4. 评价指标

| 指标 | 目的 | 待明确事项 |
| --- | --- | --- |
| **ACC（准确率）** | 评估碱基识别结果的准确程度 | chunk 的参考序列、对齐方式及 ACC 计算公式 |
| **Coverage（覆盖度）** | 评估预测结果对目标序列或参考区域的覆盖情况 | 覆盖度的分母、统计范围与汇总方式 |
| **参考基因组比对状态** | 判断输出序列能否成功比对到参考基因组 | 比对工具、参考基因组版本、成功比对阈值 |

所有指标最终按 **chunk-level** 报告；如涉及 read-level 的中间统计，应另外标注，不与 chunk-level 指标混用。

## 5. 数据来源

- **体系**：S1
- **样本**：HG002  
- **原始数据路径**：/mnt/zzbnew/poregpt/datasets/DNA_S1_HG00200_MIX_250F701901011
- **数据提取后保存路径**：/mnt/zzbnew/rnamodel/likaifeng/Nanopore-Robustness-Benchmark/data
- **数据版本、测序平台、数据规模、训练/验证/测试划分**：待补充
- **参考基因组版本**：/mnt/zzbnew/rnamodel/data/ref/chm13v2.0.fa

## 6. 模型设置

### 6.1 端到端模型

- **Bonito** 权重参数：/mnt/zzbnew/rnamodel/model/bonito/dna_basic_0121

### 6.2 基座模型

- 模型名称及 checkpoint：
- 1：/mnt/zzbnew/poregpt/models/HF_RSQ741C12V523_DNABERT_V543
- 2：/mnt/zzbnew/poregpt/models/HF_RSQ741C12V523_DNADLLM_V011
## 7. 项目结构

```text
Nanopore-Robustness-Benchmark/
├── README.md                 # 项目说明
├── pyproject.toml            # Python 项目与依赖配置
├── configs/                  # 模型、扰动和实验配置
├── data/                     # 数据及划分（大型数据不提交到 Git）
├── src/                      # 核心实现：预处理、扰动、模型、指标
├── scripts/                  # 实验执行脚本
└── results/                  # 结果、图表和日志
```

## 8. 安装

```bash
git clone https://github.com/Tohn11/Nanopore-Robustness-Benchmark.git
cd Nanopore-Robustness-Benchmark
python -m pip install -e .
```

> 安装命令以项目后续提供有效的 `pyproject.toml` 为前提。CUDA、PyTorch 和 Basecaller 等依赖可能需要根据服务器环境单独安装。

## 9. 实验流程（规划中）

1. 加载原始 read 电信号及对应参考信息。
2. 根据配置抽取 read，并在选定区间施加单一或复合扰动。
3. 使用端到端模型与基座模型执行 Basecalling。
4. 将预测结果按规定的 chunk 映射、对齐及汇总。
5. 统计 ACC、Coverage 和参考基因组比对状态。
6. 比较不同模型、扰动类型与强度下的鲁棒性。

## 10. 可复现性

后续将公开或记录实验配置、数据划分、模型版本、随机种子、扰动参数、对齐工具及其版本，并提供可复现的运行命令。

## 11. License

开源许可证待确定。
