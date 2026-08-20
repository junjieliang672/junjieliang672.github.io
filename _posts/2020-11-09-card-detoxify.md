---
layout: post
title: "防御机制 · Detoxify (unitaryai/detoxify)"
date: 2020-11-09
description: "一个 2020 年的 Kaggle 遗产分类器，被 157 篇论文当毒性裁判：对 GPT-4 写的仇恨内容 F1 只有 0.598"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**一个 2020 年的 Kaggle 遗产分类器，被 157 篇论文当毒性裁判：对 GPT-4 写的仇恨内容 F1 只有 0.598**

- **主页**：[https://github.com/unitaryai/detoxify](https://github.com/unitaryai/detoxify)
- **从哪读起**：先读 GitHub README 最下方的 Limitations 一段（它自己承认靠脏词判定），再看 HateBench（[2501.16750](https://arxiv.org/abs/2501.16750)）Table 3/4 的第三方成绩单——这两处放在一起就能解释为什么大家用它、以及它在什么时候会失灵。

## 一个 Kaggle 遗产：三个 checkpoint、七个分数、阈值 0.5，然后全领域拿它当尺子

Detoxify 是三个独立的文本分类 checkpoint，各自对应一届 Jigsaw 比赛：`toxic_bert`（BERT-base-uncased，2018 Toxic Comment Classification Challenge，训练语料是维基百科讨论页评论）、`unbiased_toxic_roberta`（RoBERTa-base，2019 Unintended Bias Challenge，Civil Comments）、`multilingual_toxic_xlm_r`（XLM-RoBERTa-base，2020 多语言赛）。输入一条文本，输出 toxicity / severe_toxicity / obscene / threat / insult / identity_attack / sexual_explicit 这几个 0–1 分数（具体几个随 checkpoint 变），绝大多数论文直接卡 0.5：≥0.5 判有毒。

它为什么被这么多人用，理由很俗：Apache-2.0，`pip install detoxify` 之后本地跑，没有配额。对照组是 Google 的 Perspective API——默认速率 1 QPS，要做 RealToxicityPrompts 那种上万条 continuation 的评测得申请提额、联网、还得对着别人的服务端排队。Detoxify 是一个 500MB 的本地前向。

于是它在本地语料的 6160 篇论文里被 157 篇提到，同时干三份完全不同的活：

- **输出过滤器**：模型生成完，Detoxify 打分，超阈值就拦掉或重采。
- **评测指标**：去毒方法报「毒性下降了 X%」，那个 X 常常就是 Detoxify 分数的均值或超阈值比例。Visual Adversarial Examples（[2306.13213](https://arxiv.org/abs/2306.13213)）用它统计 MiniGPT-4 在 RealToxicityPrompts 挑战子集上的中毒率；Breaking Bad Tokens（[2505.14536](https://arxiv.org/abs/2505.14536)）用它衡量 SAE steering 的去毒强度。
- **奖励信号 / 训练标签**：RLHF 或知识编辑里的 reward model，比如 Adaptive Detoxification（[2505.22298](https://arxiv.org/abs/2505.22298)）那一类毒性感知的权重编辑。

这三件事对分类器的要求完全不同——当过滤器只要求它在对抗输入下不漏；当指标要求它跨方法可比；当奖励要求它的决策边界经得起被梯度直接优化。Detoxify 是为第一届 Kaggle 比赛的 AUC 排行榜训练的，三件事都不是它的设计目标。README 自己把话说得很白：

> "If words that are associated with swearing, insults or profanity are present in a comment, it is likely that it will be classified as toxic, regardless of the tone or the intent of the author e.g. humorous/self-deprecating. This could present some biases towards already vulnerable minority groups."

这句话是 2020 年写的，写的时候还没有 GPT-4 帮人把仇恨内容写得一个脏字都不带。

## HateBench 的账：GPT-4 写的仇恨内容 F1 掉到 0.598，非仇恨样本误杀约 28%

HateBench（[2501.16750](https://arxiv.org/abs/2501.16750)，USENIX Security '25）是目前唯一一份把 Detoxify 和另外七个仇恨言论检测器放在同一批数据上、由第三方评的成绩单。数据是 HateBenchSet：六个 LLM 生成、覆盖 34 个 identity group，人工标注 11,016 条，去掉拒答后剩 7,838 条——3,641 条 hateful、4,197 条 non-hateful。

整体（Table 3），Detoxify (Original)：precision 0.724、recall 0.858、F1 0.782、accuracy 0.782。Detoxify (Unbiased) 更差：0.691 / 0.760 / 0.730 / 0.731。榜首是 TweetHate（F1 0.864）、OpenAI Moderation（0.852）、LFTW（0.825）。也就是说 Detoxify 在八个检测器里排中下游，但它是被引用得最多的那个。

按生成模型拆开（Table 4）才是要害：

| 检测器 | Vicuna | GPT-3.5 | GPT-4 | 人写 |
|---|---|---|---|---|
| Detoxify (Original) | 0.835 | 0.782 | **0.598** | 0.595 |
| Detoxify (Unbiased) | 0.784 | 0.700 | 0.584 | 0.543 |

从 Vicuna 到 GPT-4 掉了 23.7 个百分点。掉的原因和 README 那句自白严丝合缝：Vicuna 写仇恨内容会带脏词，GPT-4 不带。Perspective 也一样，GPT-3.5 上 F1 0.878，GPT-4 上 0.621。检测器不是被攻击打垮的，是被生成端的文风换代甩掉的。

**代价那一栏。**HateBench 没有直接给 false positive rate，下面这个数是本卡从 P/R 和类别计数反推的推算：TP = 0.858 × 3641 ≈ 3124；FP = TP/0.724 − TP ≈ 4315 − 3124 ≈ 1191；1191 / 4197 ≈ **28.4%**。即在这个数据集上，每 100 条无害样本大约有 28 条被 Detoxify 判成仇恨。这个数据集的 non-hateful 侧本身就偏难（都是围绕 identity group 的讨论），换成日常语料 FPR 会低不少——但方向是明确的：它不是一个能直接挂上线做硬拦截的分类器，它自己的 README 也写了 "content moderation assistance rather than autonomous decision-making"。

这一节的数不是厂商自测。Unitary 从来没报过对 LLM 生成内容的成绩，报成绩的是把它当基线的第三方。

## 攻击者预算：白盒权重、无限次查询、连续分数反馈

上面 0.782 的 F1 有一个隐含前提：**输入是自然写出来的，作者没有针对这个分类器优化过。**真实预算比这宽得多：

- **权重全公开**（Apache-2.0，HuggingFace 上 `unitary/toxic-bert`），可以直接对输入做梯度攻击，不需要估计器。
- **查询次数无上限**：本地推理，没有速率限制、没有封号，一张消费级卡一秒能跑几百条。
- **反馈是连续分数不是二值**：模型返回 0.4831 而不是「拦/不拦」，这给了攻击者一条可爬的坡。对比之下，攻击一个只回「blocked」的线上 moderation 端点要难一个量级。
- **无状态**：单条文本单次前向，没有会话历史、没有跨轮累积。攻击者不需要考虑「试第 11 次会不会被限流」。

第三方在这个预算下测过的结果是 TaeBench（[2410.05573](https://arxiv.org/abs/2410.05573)，NAACL 2025 Industry Track，Amazon）。它从 20 多种 TAE（toxic adversarial example，即对有毒文本做小扰动让检测器判成无害，比如替换同义词、改写句式）攻击配方的 94 万条原始生成里，用模型标注 + 人工核验筛出 26.4 万条「仍然有毒、语法通顺、看起来像人写的」样本。detoxify-unbiased 的成绩：

- 干净 Jigsaw 测试集 AUC **97.78%**
- FNR（漏检率）：Jigsaw 种子样本 **9.14%** → TaeBench 样本 **36.20%**
- OffensiveTweet：种子 **17.84%** → TaeBench **36.13%**

注意这些是**迁移攻击**——TaeBench 的样本不是针对 detoxify 生成的，是别的检测器上生成后转过来的。也就是说，攻击者连白盒都没用上，只用别人现成的对抗样本，漏检率就从 9% 涨到 36%。

对抗训练能救回一部分：用 Jigsaw + TaeBench 增强重训后，detoxify 在 TaeBench 测试集上 FNR 降到 **23.25%**。仍然是四条里漏一条，而且 unitaryai/detoxify 官方发布的权重**没有**做过这个对抗训练——你 pip 装下来的是 36% 那个版本。

**必须明写的空白**：HateBench 那组 ASR 0.966 的规避实验（TextFooler 做词级扰动）打的是 Perspective（0.966）、Moderation（0.974）、TweetHate（0.975）——**没有针对 Detoxify 的对应数字**。语料里提及 Detoxify 最多的两篇（[2512.19297](https://arxiv.org/abs/2512.19297) 的 LoRA 后门、[2603.23509](https://arxiv.org/abs/2603.23509) 的 internal safety collapse）我没能在公开渠道核实其实验细节，这里不引用它们的数字。

## 当裁判也是被优化的目标

最后一个坑和攻击者无关，是这 157 篇论文自己造的。

[2306.13213](https://arxiv.org/abs/2306.13213) 在同一批 MiniGPT-4 输出（RealToxicityPrompts 挑战子集，1225 条 prompt，ε=16/255 的对抗图像）上同时跑了两个裁判：Perspective API 判 **53.6%** 含有毒属性，Detoxify 判 **46.4%**。同一批文本，换个裁判差 7.2 个百分点。放宽到无约束对抗图像是 66.0% vs 61.0%。论文自己记了一句："Perspective API flags a larger ratio of content as toxic than that of detoxify." 这意味着任何一篇只报单一裁判分数的去毒论文，它的绝对数值不能和另一篇跨着比。

更麻烦的是把它当被优化目标。像 [2505.14536](https://arxiv.org/abs/2505.14536) 那样调 SAE steering 强度、或者 [2505.22298](https://arxiv.org/abs/2505.22298) 那样按毒性分数做知识编辑，如果唯一的信号是 Detoxify 分数，那做的就是对一个 2020 年 RoBERTa 的决策边界做梯度下降。它已知靠脏词判定——把脏词磨掉，分数就下去了，语义有没有变是另一回事。[2505.14536](https://arxiv.org/abs/2505.14536) 报的是「毒性下降最多 20%」且标准 NLP benchmark 分数稳定，但同时承认 GPT-2 Small 上 "fluency can degrade noticeably depending on the aggressiveness"，流畅度这一侧是有代价的。

可用的做法很朴素：Detoxify 当粗筛（AUC 97.78% 的干净集性能足够挡住明着骂人的那一批）、当跨论文的可比坐标（大家都用同一个 checkpoint、同一个 0.5 阈值，至少数字之间能对齐），但报「近零毒性」的时候得换一个没参与优化的裁判复核一遍。这一步在语料里那 157 篇论文中做的是少数。

**已核实来源**

- <https://github.com/unitaryai/detoxify>
- <https://github.com/unitaryai/detoxify/releases>
- <https://pypi.org/pypi/detoxify/json>
- <https://arxiv.org/abs/2501.16750>
- <https://arxiv.org/html/2501.16750v1>
- <https://arxiv.org/abs/2410.05573>
- <https://arxiv.org/html/2410.05573v1>
- <https://aclanthology.org/2025.naacl-industry.21/>
- <https://arxiv.org/html/2306.13213v2>
- <https://arxiv.org/abs/2505.14536>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
