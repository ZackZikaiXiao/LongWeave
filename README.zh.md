# LongWeave：在真实相关性与可验证性之间搭桥的长文本生成基准

[![Paper](https://img.shields.io/badge/arXiv-2510.24345-b31b1b.svg)](https://arxiv.org/pdf/2510.24345)
[![License](https://img.shields.io/github/license/ZackZikaiXiao/LongWeave)](LICENSE)

切换语言： [English](README.md)

本仓库为 LongWeave 的官方实现（EMNLP 2025）。

LongWeave 是一个开源框架，用于评估大语言模型（LLMs）在长上下文生成任务上的能力。

## 功能特性

任务特性：

- 内置多种长文本生成任务（代码修复、销售报告生成等）
- 在保持真实世界相关性的同时确保任务可验证性
- 可配置的输入（128k）和输出（8k）长度
- 可无限生成任务样本

评测框架特性：
- 支持在 `core/tasks` 中添加新任务
- 支持多种 LLM 后端（DashScope、OpenAI、HuggingFace、DLC），在 `core/serve` 中配置推理端点
- 通过 `config` 目录下的 YAML 文件配置任务与模型参数
- 并发推理，并发 LLM-as-a-judge 评审
- 推理与评测过程中的中断恢复
- 结果汇总与报告生成

## 评测流程概览

下图展示了评测流水线：

![LongWeave Pipeline](img/LongWeave_pipeline.png)

## 环境搭建与运行

### 1. 环境准备

安装依赖：

```bash
pip install -r requirements.txt
```

### 2. 部署 LLM 服务

可使用 vLLM 部署本地 LLM 服务。以下给出两个模型的示例命令：

```bash
# 部署 Llama-3.1-8B-Instruct
vllm serve /mnt/data/models/Meta-Llama-3___1-8B-Instruct \
  --served-model-name Meta-Llama-3___1-8B-Instruct \
  --tensor-parallel-size 8 \
  --trust-remote-code \
  --port 8012 \
  --enable-prefix-caching \
  --max-model-len 131072

# 部署 Qwen3-8B
vllm serve /mnt/data/models/Qwen/Qwen3-8B \
  --served-model-name qwen3-8b \
  --tensor-parallel-size 8 \
  --trust-remote-code \
  --port 8012 \
  --enable-prefix-caching \
  --max-model-len 131072 \
  --rope-scaling '{"type":"yarn","factor":4.0, "original_max_position_embeddings":32768,"rope_type": "yarn"}' \
  --chat-template ./qwen3_nonthinking.jinja
```

### 3. 配置 LLM 端点

服务部署完成后，在 [core/serve/dlc.py](core/serve/dlc.py) 中将端点配置为你部署服务的 IP：

```python
MODEL_TO_API_BASE = {
    "Meta-Llama-3___1-8B-Instruct": "http://YOUR_IP_ADDRESS:8012/v1",
    "qwen3-8b": "http://YOUR_IP_ADDRESS:8012/v1",
}
```

请通过连通性测试确保内部网络可访问这些端点。

### 4. 解压数据集

解压数据压缩包，准备评测数据：

```bash
tar -xf data.tar.xz -C .
```

### 5. 配置任务与模型参数

编辑 [config/](config/) 目录下的配置文件：

- [global_config.yaml](config/global_config.yaml)：选择运行的任务并配置并发线程数
- [model_config.yaml](config/model_config.yaml)：选择要评测的模型
- [task_config.yaml](config/task_config.yaml)：配置任务相关参数

### 6. 运行评测

执行主脚本进行推理与评测：

```bash
python main.py
```

评测结果将按模型名称存放在 [results/](results/) 目录。

### 7. 生成汇总报告

在对多个模型完成评测后，可生成汇总报告：

```bash
python generate_summary.py
```

此命令将在当前目录生成 CSV 与 Excel 格式的汇总报告。

## 扩展新任务

如需新增任务，请在 [core/tasks/](core/tasks/) 目录中扩展基类以实现新任务。

## 项目结构

```
longweave_open/
├── config/                 # 配置文件
│   ├── global_config.yaml  # 全局设置
│   ├── model_config.yaml   # 模型设置
│   └── task_config.yaml    # 任务设置
├── core/                   # 核心框架组件
│   ├── metrics/            # 评测指标
│   ├── serve/              # LLM API 接口
│   ├── simulation/         # 数据生成工具
│   ├── tasks/              # 任务实现
│   ├── pipeline.py         # 执行流水线
│   ├── runner.py           # 任务运行器
│   └── seed.py             # 随机种子管理
├── data/                   #（解压后）评测数据集
├── results/                # 评测结果
├── main.py                 # 主执行脚本
└── generate_summary.py     # 结果汇总脚本
```

## 数据集

- 我们也在 Hugging Face 托管了评测数据：[LongWeave 数据集](https://huggingface.co/datasets/zikaixiao1/LongWeave)。
- 注意：本 GitHub 仓库是自包含的，能够从零生成全部评测数据；Hugging Face 上的数据用于方便查看或其他额外用途。

## 引用

```bibtex
@inproceedings{xiao-etal-2025-longweave,
    title = "{L}ong{W}eave: A Long-Form Generation Benchmark Bridging Real-World Relevance and Verifiability",
    author = "Xiao, Zikai and Huang, Fei and Tu, Jianhong and Wei, Jianhui  and Ma, Wen and Zhou, Yuxuan and Wu, Jian and Yu, Bowen and Liu, Zuozhu and Lin, Junyang",
    editor = "Christodoulopoulos, Christos and Chakraborty, Tanmoy and Rose, Carolyn and Peng, Violet",
    booktitle = "Findings of the Association for Computational Linguistics: EMNLP 2025",
    month = nov,
    year = "2025",
    address = "Suzhou, China",
    publisher = "Association for Computational Linguistics",
    url = "https://aclanthology.org/2025.findings-emnlp.549/",
    pages = "10386--10417",
    ISBN = "979-8-89176-335-7",
}
```

## 许可证

本项目基于 MIT 许可发布，详情见 [LICENSE](LICENSE) 文件。
