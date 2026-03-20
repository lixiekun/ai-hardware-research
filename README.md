# AI 大模型私有化部署硬件调研

> 本项目用于整理企业内网部署大模型的硬件选型调研，预算 150 万人民币，目标支持 50-100 人使用。

## 目录

- [模型硬件需求](#模型硬件需求)
- [硬件价格参考](#硬件价格参考)
- [预算可行性分析](#预算可行性分析)
- [推荐方案](#推荐方案)
- [参考链接](#参考链接)

---

## 模型硬件需求

### MiniMax M2.5

| 指标 | 数值 | 来源 |
|------|------|------|
| 总参数 | 229B | [火山引擎官方文档](https://www.volcengine.com/docs/6419/2222496) |
| 激活参数 | 10B | 同上 |
| 权重显存需求 | **220GB** | [GitHub官方部署指南](https://github.com/MiniMax-AI/MiniMax-M2.5/blob/main/docs/transformers_deploy_guide_cn.md) |
| 最低运行配置 | 4× 96GB GPU | [SMZDM评测](https://post.smzdm.com/p/a650mw3z) |
| 128K上下文显存 | ~594GB | [apxml规格表](https://apxml.com/zh/models/minimax-m2) |

### GLM-5

| 指标 | 数值 | 来源 |
|------|------|------|
| 总参数 | 744B | [智谱官方技术报告](https://www.53ai.com/news/OpenSourceLLM/2026022239486.html) |
| 激活参数 | 40B | 同上 |
| 完整版显存需求 | **~1.5TB** | [知乎分析](https://zhuanlan.zhihu.com/p/2005395734573360144) |
| 2-bit量化后 | 241GB | [Unsloth AI官方](https://www.landiannews.com/archives/111838.html) |
| 推荐配置 | 8× H100 80GB (FP8量化) | [apxml规格表](https://apxml.com/zh/models/glm-5) |

### 其他常见模型

| 模型 | 参数量 | 显存需求 (FP16) | 显存需求 (4-bit) |
|------|--------|-----------------|------------------|
| GLM-4-9B | 9B | ~18GB | ~6GB |
| GLM-4-32B | 32B | ~64GB | ~20GB |
| DeepSeek-V3 | 671B (MoE) | ~1.3TB | ~400GB |
| Qwen-72B | 72B | ~144GB | ~40GB |

---

## 硬件价格参考

### 英伟达 GPU

| 产品 | 显存 | 价格 | 来源 |
|------|------|------|------|
| RTX 4090 | 24GB | ~1.5万/张 | 市场参考价 |
| RTX 6000 Ada | 96GB | ~5万/张 | [中关村在线](https://vga.zol.com.cn/917/9176815.html) |
| A100 80GB | 80GB | ~8-10万/张 | 市场参考价 |
| H20 八卡服务器 | 8×96GB | **100-140万/台** | [36氪](https://m.36kr.com/p/2670933485451012) |
| H800 服务器 | 8×80GB | **200-300万/台** | [东方财富研报](https://pdf.dfcfw.com/pdf/H3_AP202506121689781660_1.pdf) |

> ⚠️ 注：H800服务器在H20禁售后从200万涨至300万

### 华为昇腾

| 产品 | 说明 | 来源 |
|------|------|------|
| 昇腾910B | FP16算力313TFLOPS | [华为官方](https://www.hiascend.com/) |
| Atlas 800 9000 | 8×昇腾910B，FP16算力2.24PFLOPS | 华为官网 |
| 月租参考 | 2-3万元/月 | [财联社](https://caifuhao.eastmoney.com/news/20251121093242372531360) |

> ⚠️ 昇腾服务器价格需向华为或代理商询价，公开渠道无明确售价

### 苹果 Mac Studio

| 配置 | 统一内存 | 价格 | 来源 |
|------|----------|------|------|
| Mac Studio M3 Ultra 起售价 | 96GB | 32,999元 | [Apple中国官网](https://www.apple.com.cn/shop/buy-mac/mac-studio) |
| Mac Studio M3 Ultra 512GB | 512GB | **108,749元** | [新浪财经](https://finance.sina.com.cn/tech/roll/2025-03-07/doc-inenukkx0263306.shtml) |

### 国产 GPU 替代

| 厂商 | 产品 | 显存 | 备注 |
|------|------|------|------|
| 摩尔线程 | MTT S5000 | 80GB | MiniMax M2.5 Day-0适配 |
| 昆仑芯 | P800 | - | DeepSeek/GLM适配 |
| 天数智芯 | 天枢系列 | - | 多模型适配 |
| 沐曦 | 曦云 C500/C550 | - | GLM-5 Day-0适配 |

---

## 预算可行性分析

### 150万预算 - MiniMax M2.5 (需220GB显存)

| 方案 | 配置 | 显存总量 | 预估价格 | 可行性 |
|------|------|----------|----------|--------|
| RTX 6000 Ada | 4卡 | 384GB | ~80万 | ✅ 预算内 |
| H20 八卡服务器 | 8卡 | 768GB | 100-140万 | ✅ 预算内 |
| Mac Studio M3 Ultra 512GB | 2台 | 1024GB | ~22万 | ✅ 预算内 |

### 150万预算 - GLM-5

| 方案 | 配置 | 显存总量 | 预估价格 | 可行性 |
|------|------|----------|----------|--------|
| **量化版 (2-bit, 241GB)** | | | | |
| Mac Studio 512GB | 1台 | 512GB | ~11万 | ✅ 可运行 |
| H20 服务器 | 1台 | 768GB | 100-140万 | ✅ 可运行 |
| **完整版/FP8 (~640GB+)** | | | | |
| H800 8卡 | 1台 | 640GB | 200-300万 | ❌ 超预算 |
| H100 8卡 | 1台 | 640GB | ~400万+ | ❌ 超预算 |

### 并发能力估算

| 配置 | 同时在线用户 | 高峰并发 | 体验 |
|------|--------------|----------|------|
| 单张 RTX 4090 | 5-8人 | 2-3人 | 基础 |
| 单张 A100 80GB | 10-15人 | 5-8人 | 良好 |
| 8× H20/H800 | 50-80人 | 20-30人 | 优秀 |
| 4× Mac Studio 512GB | 30-50人 | 15-20人 | 良好 |

> 并发数受模型大小、上下文长度、响应速度要求等因素影响，仅供参考

---

## 推荐方案

### 场景一：MiniMax M2.5 为主

**推荐：H20 八卡服务器**

- 价格：100-140万（预算内）
- 显存：768GB，足够运行 + 长上下文
- 并发：支持 30-50 人同时使用
- 生态：CUDA 生态成熟

### 场景二：GLM-5 量化版

**推荐：Mac Studio M3 Ultra 512GB × 2**

- 价格：~22万（远低于预算）
- 显存：1024GB 统一内存
- 优点：性价比极高、功耗低
- 缺点：仅限推理，不适合训练

### 场景三：国产化合规要求

**推荐：昇腾910B 8卡服务器**

- 价格：需询价（预计 100-200万区间）
- 优点：GLM-5 原生适配、国产合规
- 缺点：生态需适配

### 场景四：需要完整版 GLM-5

**预算需提升至 400万+**

- 推荐：H100/H200 8卡服务器
- 或：昇腾集群方案

---

## 参考链接

### 官方文档
- [MiniMax M2.5 部署指南](https://github.com/MiniMax-AI/MiniMax-M2.5/blob/main/docs/transformers_deploy_guide_cn.md)
- [火山引擎 - MiniMax M2.5](https://www.volcengine.com/docs/6419/2222496)
- [智谱AI 开放平台](https://docs.bigmodel.cn/)
- [华为昇腾官网](https://www.hiascend.com/)

### 价格参考
- [36氪 - 智算中心招投标](https://m.36kr.com/p/2670933485451012)
- [Apple 中国官网 - Mac Studio](https://www.apple.com.cn/shop/buy-mac/mac-studio)
- [中国移动智算中心集采](https://www.sohu.com/a/780970415_120740496)

### 技术分析
- [GLM-5 技术报告解读](https://www.53ai.com/news/OpenSourceLLM/2026022239486.html)
- [Unsloth AI - GLM-5 量化](https://www.landiannews.com/archives/111838.html)
- [apxml - 模型规格](https://apxml.com/zh/models/)

---

## 更新记录

| 日期 | 内容 |
|------|------|
| 2026-03-20 | 初始版本，调研 MiniMax M2.5 和 GLM-5 部署方案 |
