# AI 大模型私有化部署硬件调研

> **目标**：150万预算内部署大模型，支持50-100人使用
>
> **重点模型**：DeepSeek V4 (1.6T/284B)、Kimi K2.6 (1T)、MiniMax M2.7 (230B)、GLM-5.1 (744B)

---

## 文档索引

| 文档 | 内容 |
|------|------|
| [README.md](README.md) | 总体调研概览 |
| [nvidia-h200.md](nvidia-h200.md) | 英伟达 H200 方案详细分析 |
| [huawei-ascend.md](huawei-ascend.md) | 华为昇腾方案详细分析 |
| [domestic-gpu-analysis.md](domestic-gpu-analysis.md) | 国产 GPU 芯片方案分析 |

---

## 目录

- [模型硬件需求](#一模型硬件需求)
- [硬件价格参考](#二硬件价格参考)
- [预算可行性](#三预算可行性分析)
- [推荐方案](#四推荐方案)
- [参考链接](#参考链接)

---

## 一、模型硬件需求

### 1.1 目标模型

| 模型 | 总参数 | 激活参数 | 显存需求 | 来源 |
|------|:------:|:--------:|:--------:|------|
| **DeepSeek V4-Pro** | 1.6T | 49B | ~2.4TB (FP8) / ~700GB (INT4) | [DeepSeek官方](https://api-docs.deepseek.com/zh-cn/news/news260424) |
| **DeepSeek V4-Flash** | 284B | 13B | ~22GB (INT4) / ~160GB (FP16) | [51CTO](https://www.51cto.com/aigc/11729.html) |
| **Kimi K2.6** | 1T | 32B | ~2TB (FP16) / ~250GB (INT4) | [HuggingFace](https://huggingface.co/moonshotai/Kimi-K2.6) |
| **GLM-5.1** | 744B | 40B | ~1.5TB (完整) / 440GB (FP8/INT4) | [智谱官方](https://docs.bigmodel.cn/cn/guide/models/text/glm-5.1) |
| **GLM-5** | 744B | 40B | ~1.5TB (完整) / 241GB (2-bit) | [智谱官方](https://www.53ai.com/news/OpenSourceLLM/2026022239486.html) |
| **MiniMax M2.7** | 230B | 10B | ~220GB (FP16) / ~50GB (INT4) | [MiniMax官方](https://www.minimaxi.com/news/minimax-m27-zh) |
| **MiniMax M2.5** | 229B | 10B | ~220GB | [GitHub](https://github.com/MiniMax-AI/MiniMax-M2.5) |

> **DeepSeek V4 新特性**（2026年4月24日发布）：
> - 双版本发布：V4-Pro (1.6T/49B) 和 V4-Flash (284B/13B)
> - 全系标配 **100万 token 超长上下文**
> - V4-Pro 推理 FLOPs 仅为 V3.2 的 **27%**，KV 缓存降至 **10%**
> - V4-Flash INT4 单卡 RTX 5090 即可部署（~22GB）
> - 已开源，昇腾 910B 单机 8 卡可部署 V4-Flash
>
> 来源：[DeepSeek官方](https://api-docs.deepseek.com/zh-cn/news/news260424)、[知乎](https://zhuanlan.zhihu.com/p/2031132960632599659)

> **Kimi K2.6 新特性**（2026年4月20日发布）：
> - **1T 总参 / 32B 激活**，MoE 架构，15.5T tokens 训练
> - SWE-Bench Pro **58.6%**（并列 GPT-5.5）
> - 原生多模态智能体模型，长程编码能力突出
> - 支持多天自主任务执行 + Agent Swarm
> - 已开源，256K 上下文窗口
>
> 来源：[Kimi官方](https://www.kimi.com/ai-models/kimi-k2-6)、[MLQ.ai](https://mlq.ai/news/moonshot-ai-releases-kimi-k26-open-source-coding-model-with-autonomous-multi-day-task-execution/)

> **MiniMax M2.7 新特性**（2026年3月18日发布）：
> - **230B 总参 / 10B 激活**，MoE 架构，首个"自我进化"模型
> - Agent Harness 体系让模型参与自身训练，承担 **30-50%** 研发工作量
> - 编程与推理能力迈入第一梯队，与顶级模型比肩
> - 已开源，SGLang Day-0 支持，GGUF 量化版已发布
> - INT4 量化 ~50GB，双卡 RTX 4090 或单卡 RTX 6000 Ada 即可部署
>
> 来源：[MiniMax官方](https://www.minimaxi.com/news/minimax-m27-zh)、[腾讯云](https://cloud.tencent.com/developer/article/2653788)

> **GLM-5.1 新特性**（2026年4月发布）：
> - 编码能力暴涨，核心编码 **45.3 分**，达 Claude Opus 4.6 的 **94.6%**
> - 长程任务处理：单次任务可持续自主工作 **8 小时**
> - 高性能 Kernel 优化：**3.6 倍**几何平均加速比
> - AIME 2026：**95.30 分**（排名第 1）
> - SWE-Bench Pro：**58.40 分**（排名第 2）
>
> 来源：[量子位](https://www.qbitai.com/2026/04/397898.html)、[CSDN](https://aicoding.csdn.net/69d604970a2f6a37c59dc8e0.html)

### 1.2 其他常见模型

| 模型 | 参数 | FP16显存 | 4-bit显存 |
|------|:----:|:--------:|:---------:|
| GLM-4-9B | 9B | ~18GB | ~6GB |
| GLM-4-32B | 32B | ~64GB | ~20GB |
| DeepSeek-V3 | 671B | ~1.3TB | ~400GB |
| Qwen-72B | 72B | ~144GB | ~40GB |

---

## 二、硬件价格参考

### 2.1 英伟达 GPU

| 产品 | 显存 | 价格 | 来源 |
|------|:----:|------|------|
| RTX 4090 | 24GB | ~1.5万/张 | 市场价 |
| RTX 6000 Ada | 96GB | ~5万/张 | [中关村在线](https://vga.zol.com.cn/917/9176815.html) |
| **H200 8卡服务器** | **1128GB** | **150-230万** | [新浪财经](https://finance.sina.com.cn)、[cnBeta](https://www.cnbeta.com.tw) |
| H20 8卡服务器 | 768GB | **100-140万** | [36氪](https://m.36kr.com/p/2670933485451012) |
| H800 8卡服务器 | 640GB | **200-300万** | [东方财富](https://pdf.dfcfw.com/pdf/H3_AP202506121689781660_1.pdf) |

> **H200 亮点**：141GB HBM3e 显存 / 4.8TB/s 带宽 / FP8 支持详见 → [nvidia-h200.md](nvidia-h200.md)

### 2.2 华为昇腾

| 产品 | 规格 | 价格 | 来源 |
|------|------|------|------|
| Atlas 800 (8×910B) | 512GB, 2.24PFLOPS | **57.5万** | [中标公告](https://zcb.syuct.edu.cn/info/1007/2633.htm) |
| 月租参考 | - | 2-3万/月 | [财联社](https://caifuhao.eastmoney.com/news/20251121093242372531360) |

### 2.3 苹果 Mac Studio

| 配置 | 内存 | 价格 | 状态 |
|------|:----:|------|:----:|
| M3 Ultra 256GB | 256GB | ~8万 | ✅ 在售 |
| ~~M3 Ultra 512GB~~ | ~~512GB~~ | ~~10.9万~~ | ❌ **已下架** |

> ⚠️ **2026年3月**：苹果因全球 DRAM 短缺下架了 512GB 版本，目前最高仅支持 256GB

来源：[IT之家](https://www.ithome.com/0/926/348.htm)、[SMZDM](https://post.smzdm.com/p/a5r8ggvl)

### 2.4 国产 GPU

| 厂商 | 产品 | 备注 |
|------|------|------|
| 摩尔线程 | MTT S5000 | MiniMax M2.5 适配 |
| 昆仑芯 | P800 | DeepSeek/GLM 适配 |
| 沐曦 | 曦云 C500 | GLM-5 适配 |

---

## 三、预算可行性分析

### 3.1 DeepSeek V4

| 版本 / 精度 | 显存需求 | 150万预算 | 说明 |
|:-----------:|:--------:|:---------:|------|
| **V4-Flash INT4** | **~22GB** | ✅ **单卡即可** | RTX 5090 / 4090 单卡可跑 |
| V4-Flash FP16 | ~160GB | ✅ 可行 | 2-4× 企业级 GPU |
| **V4-Pro INT4** | **~700GB** | ✅ **H200 可行** | 8× H100/H200 |
| V4-Pro FP8 | ~2.4TB | ❌ 需 500万+ | 16× H100 级别 |

| 方案 (V4-Flash) | 显存 | 价格 | 结论 |
|------|:----:|------|:----:|
| **单卡 RTX 5090** | 32GB | ~2万 | ✅ INT4 可跑 |
| **H20 8卡服务器** | 768GB | 100-140万 | ✅ FP16 也行 |
| **Atlas 800 (8×910B)** | 512GB | 80-120万 | ✅ 已有部署方案 |

| 方案 (V4-Pro INT4) | 显存 | 价格 | 结论 |
|------|:----:|------|:----:|
| **H200 8卡服务器** | **1128GB** | **150-230万** | ✅ **最佳性能** |
| H20 8卡服务器 | 768GB | 100-140万 | ✅ 刚好够用 |
| Atlas 800 (8×910B) | 512GB | 80-120万 | ⚠️ INT4 勉强 |

### 3.2 Kimi K2.6 (1T总参 / 32B激活)

| 版本 | 显存需求 | 150万预算 |
|:----:|:--------:|:---------:|
| INT4 量化 | ~250GB | ✅ 多方案可行 |
| FP8 量化 | ~500GB | ✅ H200 可行 |
| FP16 完整版 | ~2TB | ❌ 需 400万+ |

| 方案 | 显存 | 价格 | 结论 |
|------|:----:|------|:----:|
| **H200 8卡服务器** | **1128GB** | **150-230万** | ✅ **FP8 丝滑** |
| H20 8卡服务器 | 768GB | 100-140万 | ✅ INT4 / FP8 可行 |
| Atlas 800 (8×910B) | 512GB | 80-120万 | ⚠️ INT4 需适配 |
| 4× RTX 6000 Ada | 384GB | ~80万 | ⚠️ INT4 勉强 |

### 3.3 MiniMax M2.7 / M2.5 (230B / 229B)

| 版本 | 显存需求 | 150万预算 |
|:----:|:--------:|:---------:|
| **M2.7 INT4 量化** | **~50GB** | ✅ **双卡 4090 即可** |
| M2.7 / M2.5 FP16 | ~220GB | ✅ 可行 |
| M2.7 200K 上下文 | 额外 ~48GB | ⚠️ 需更多显存 |

| 方案 | 显存 | 价格 | 结论 |
|------|:----:|------|:----:|
| **2× RTX 4090 / 单卡 RTX 6000 Ada** | 48-96GB | ~5万 | ✅ **M2.7 INT4** |
| 4× RTX 6000 Ada | 384GB | ~80万 | ✅ FP16 丝滑 |
| **H200 8卡服务器** | **1128GB** | **150-230万** | ✅ **最佳性能** |
| H20 8卡服务器 | 768GB | 100-140万 | ✅ |
| Atlas 800 (8×910B) | 512GB | 80-120万 | ⚠️ 需适配 |

### 3.4 GLM-5.1 / GLM-5

| 版本 | 显存需求 | 150万预算 |
|:----:|:--------:|:---------:|
| FP8/INT4 量化 | ~440GB | ✅ 多方案可行 |
| 2-bit 量化 | ~241GB | ✅ 多方案可行 |
| FP8 量化 | ~640GB | ⚠️ 需 200万+ / H200 可行 |
| 完整版 | ~1.5TB | ❌ 需 400万+ |

### 并发估算

| 配置 | 在线用户 | 高峰并发 | 适用模型 |
|------|:--------:|:--------:|----------|
| **8× H200** | **80-120人** | **30-50人** | V4-Pro、K2.6、GLM-5.1、M2.7 全部 |
| 8× H20/H800 | 50-80人 | 20-30人 | V4-Flash、K2.6 INT4、GLM-5.1、M2.7 |
| **Atlas 800T A3 (8×910B)** | **50-80人** | **15-25人** | V4-Flash、GLM-5.1 W4A8、M2.7 |
| 单卡 RTX 5090 | 5-10人 | 2-3人 | V4-Flash INT4、M2.7 INT4 |
| ~~4× Mac Studio 512GB~~ | ~~30-50人~~ | ~~15-20人~~ | ❌ 已下架 |

---

## 四、推荐方案

### 按场景选择

| 场景 | 推荐方案 | 价格 | 说明 |
|------|----------|------|------|
| **全模型通吃（V4-Pro + K2.6 + GLM-5.1）** | **H200 8卡** | **150-230万** | 1128GB大显存，全部模型FP8量化丝滑 |
| **DeepSeek V4-Flash 经济部署** | **单卡 RTX 5090** | **~2万** | INT4量化 ~22GB，性价比之王 |
| **Kimi K2.6 + GLM-5.1 + M2.7** | H20 8卡 | 100-140万 | INT4/FP8 均可，CUDA生态成熟 |
| **GLM-5.1 W4A8 单机部署** | **Atlas 800T A3** | **80-120万** | 单机 W4A8 量化，>45 tok/s |
| **V4-Flash 昇腾部署** | Atlas 800 | 80-120万 | 已有官方部署方案 |
| **MiniMax M2.7 INT4** | **2× RTX 4090** | **~5万** | INT4 ~50GB，超高性价比 |
| ~~性价比优先~~ | ~~Mac Studio~~ | ~~22万~~ | ❌ 512GB已下架 |

### 核心结论

```
┌──────────────────────────────────────────────────────────────────────┐
│  150万预算：                                                         │
│  • DeepSeek V4-Flash INT4 ✅ 单卡 RTX 5090 (~2万) 即可              │
│  • DeepSeek V4-Pro INT4 ✅ H200 可行 (~700GB)                       │
│  • Kimi K2.6 INT4 ✅ 可行，推荐 H200 或 H20                        │
│  • MiniMax M2.7 INT4 ✅ 双卡 4090 (~5万) 即可                       │
│  • MiniMax M2.7 FP16 ✅ 可行，推荐 H20 或 Atlas 800                 │
│  • GLM-5.1 W4A8 ✅ Atlas 800T A3 单机即可部署 (>45 tok/s)           │
│  • GLM-5.1 FP8/INT4 ✅ 可行，推荐 H200 或 Atlas 800                │
│  • GLM-5.1 完整版 / V4-Pro FP8 ❌ 需 400万+                        │
│  • H200 方案 ⚠️ 官方定价 150万，市场价 190-230万                    │
│                                                                      │
│  🆕 2026年3-5月新增：                                                │
│  • DeepSeek V4 (1.6T/284B MoE) — 全系百万token上下文                │
│  • Kimi K2.6 (1T/32B MoE) — SWE-Bench 并列 GPT-5.5                 │
│  • MiniMax M2.7 (230B/10B MoE) — 首个"自我进化"模型                 │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 参考链接

### 官方文档
- [MiniMax M2.5 部署指南](https://github.com/MiniMax-AI/MiniMax-M2.5)
- [智谱AI 开放平台](https://docs.bigmodel.cn/)
- [华为昇腾官网](https://www.hiascend.com/)

### 价格参考
- [中国移动智算中心集采](https://www.sohu.com/a/780970415_120740496)
- [Apple Mac Studio](https://www.apple.com.cn/shop/buy-mac/mac-studio)

### 技术分析
- [DeepSeek V4 官方发布](https://api-docs.deepseek.com/zh-cn/news/news260424)
- [DeepSeek V4 技术报告解读 - 知乎](https://zhuanlan.zhihu.com/p/2031132960632599659)
- [DeepSeek V4 本地部署方案 - 51CTO](https://www.51cto.com/aigc/11729.html)
- [DeepSeek V4-Flash 昇腾910B部署 - CSDN](https://deepseek.csdn.net/69ef80070a2f6a37c5a66b18.html)
- [Kimi K2.6 - HuggingFace](https://huggingface.co/moonshotai/Kimi-K2.6)
- [Kimi K2.6 官方介绍](https://www.kimi.com/ai-models/kimi-k2-6)
- [Kimi K2.6 技术分析 - MLQ.ai](https://mlq.ai/news/moonshot-ai-releases-kimi-k26-open-source-coding-model-with-autonomous-multi-day-task-execution/)
- [MiniMax M2.7 官方发布](https://www.minimaxi.com/news/minimax-m27-zh)
- [MiniMax M2.7 本地部署指南 - 腾讯云](https://cloud.tencent.com/developer/article/2653788)
- [MiniMax M2.7 量化版部署 - 知乎](https://zhuanlan.zhihu.com/p/2026750777763309036)
- [GLM-5.1 发布报道 - 量子位](https://www.qbitai.com/2026/04/397898.html)
- [GLM-5.1 详解 - CSDN](https://aicoding.csdn.net/69d604970a2f6a37c59dc8e0.html)
- [GLM-5 技术报告](https://www.53ai.com/news/OpenSourceLLM/2026022239486.html)
- [NPU vs GPU 参数对比](https://zhuanlan.zhihu.com/p/2004196636507789012)
- [H200 详细分析](https://www.nvidia.com/en-us/data-center/h200/)

---

## 更新记录

| 日期 | 内容 |
|------|------|
| 2026-03-20 | 初始版本 |
| 2026-03-20 | 添加华为昇腾详细分析 |
| 2026-03-23 | **Mac Studio 512GB 已下架**，移除该方案 |
| 2026-04-10 | **新增 GLM-5.1** 信息，**新增 H200 方案**，新增 nvidia-h200.md |
| 2026-05-08 | **新增 DeepSeek V4** (V4-Pro 1.6T + V4-Flash 284B)、**Kimi K2.6** (1T MoE)、**MiniMax M2.7** (230B MoE)，全面更新可行性分析和推荐方案 |
