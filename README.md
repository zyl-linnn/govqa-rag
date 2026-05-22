# GovQA-RAG：zw垂域大模型问答系统

> 基于 Qwen3-8B + QLoRA 三阶段微调的检索增强生成（RAG）zw政务问答系统

## 项目简介

本项目构建面向政务场景的检索增强生成系统，解决政务信息分散、术语晦涩、时效性强等痛点。系统支持口语化术语对齐、知识图谱增强检索、双重时效性校验与可追溯答案。
(具体代码和数据集6月下旬发，你懂得，哎呀，到时候直接发论文算了)

## 系统架构

```
用户交互层（CLI / Web）
    ↕
输入处理层：地域感知口语化术语对齐
    ↕
检索增强层：精确匹配 + 三元组检索 + 混合检索 + 重排序
    ↕
知识层：向量库 / 三元组问题库 / 元数据库 / 废止库
    ↕
模型层：Qwen3-8B + QLoRA (CPT→SFT→DPO)
    ↕
生成与后处理层：答案生成 + 时效性标注 + 参考文档列表
```

## 核心模块

| 模块 | 技术方案 | 说明 |
|------|----------|------|
| 多源数据处理 | pdfplumber / python-docx | 3,009 个文档对象（问答对+PDF+Word+表格） |
| 口语化术语对齐 | 规则匹配 + LLM 决策 | 支持地域方言→标准术语转换 |
| 三元组增强检索 | 知识图谱三元组 + 问题生成 | 191 个生成问题，BGE 向量匹配 |
| 混合检索 | BGE (0.6) + BM25 (0.4) | Top-30 召回 + bge-reranker-v2-m3 重排序 |
| 三阶段微调 | QLoRA (CPT→SFT→DPO) | 单卡可训练，最终加权得分 83.8% |
| 双重时效性 | 废止文件库 + 元数据库 | 自动标注"已废止"并附文件路径 |
| 多轮对话 | 环形缓冲区（8轮） | 支持指代消解与历史管理 |

## 三阶段微调

| 阶段 | 数据集 | 规模 | 损失变化 |
|------|--------|------|----------|
| CPT | BAAI/IndustryCorpus2 政务子集 | 20,000 条 | 2.27 → 2.08 |
| SFT | Government_Open_Data_Set | 7,535 条 | 2.53 → 1.56 |
| DPO | 人工+AI 联合标注偏好对 | 180 对 | 0.70 → 0.64 |

## 实验结果

| 实验组 | 全测试集加权得分 |
|--------|-----------------|
| Exp1: 基座模型（无RAG） | 28.5% |
| Exp2: 基座 + 基础RAG | 71.8% |
| Exp5: 三阶段训练 + 完整系统 | **83.8%** |

- RAG 是性能基石（+43.3%）
- 口语化对齐在口语测试集贡献 +9.2%
- 三元组增强在推理测试集贡献 +3.0%

## 技术栈

| 类别 | 技术 |
|------|------|
| 基座模型 | Qwen3-8B |
| 微调框架 | HuggingFace transformers + PEFT (QLoRA) |
| 检索 | BGE + BM25 + bge-reranker-v2-m3 |
| 向量库 | Chromadb / FAISS |
| 后端 | Python 3.10, Flask, Gradio |
| 训练硬件 | V100 32GB / A800 80GB |

## 快速开始

```bash
conda create -n govqa python=3.10
conda activate govqa
pip install torch transformers accelerate peft bitsandbytes
pip install chromadb rank_bm25 sentence-transformers flask gradio
```

> **注**：代码和数据集将在后续开放。当前仓库为设计文档先行。

## 文件结构

```
.
├── README.md        # 本文件
├── DESIGN.md        # 详细设计文档
└── .gitignore
```

## 许可证

本项目仅用于学术研究与学习参考。
