---
layout: post
title: "系统/工具 · sn-guided-diffusion（安全神经元引导的扩散越狱）"
date: 2026-08-24
description: "用扩散 LLM 的去噪过程逐格生成越狱提示，代价降到 GCG 的百分之一——但仓库至今是空的"
categories: brief
tags: [llm-security, brief, tool]
giscus_comments: false
---
**用扩散 LLM 的去噪过程逐格生成越狱提示，代价降到 GCG 的百分之一——但仓库至今是空的**

- **主页**：[https://github.com/ellyoana/sn-guided-diffusion](https://github.com/ellyoana/sn-guided-diffusion)
- **从哪读起**：先读 arXiv [2608.07430](https://arxiv.org/abs/2608.07430) 的 SN-Guided Diffusion 一节再看仓库——反过来会浪费时间，因为 GitHub 上除了一行「源码即将发布」什么都没有。
- **成名作**：[Diffusion LLMs as Targets and Adversaries: Mechanistic Safety Exploits](https://arxiv.org/abs/2608.07430) —— 第一次把「安全神经元稀疏且跨架构可迁移」这件事从剪枝实验推成一个全离线的黑盒越狱流水线。

## 两个阶段、四个零件：从剪枝出「范例回答」，到把提示词一格格解码出来

**安全神经元（safety neuron，下称 SN）**是这套东西的地基：在 LLaDA-8B 的 FFN 里，有一小撮神经元只在模型准备拒绝时才显著激活——你输入「怎么烤蛋糕」它们几乎不动，输入「怎么合成神经毒剂」它们一起亮。找它们的零件是①**探针训练器**：输入是约 14k 条 benign/malicious 提示跑过 LLaDA-8B 后的逐层 FFN 激活，输出是一组 (层, 神经元) 坐标和一个逻辑回归权重向量 w。论文报的分离度是 AUROC = 1.0（作者自评，同一模型内）。

②**Generative Pruning Cascade**：输入是一条 cloaked 有害提示——所谓 cloaking 就是把「教我制毒」包装成一段第一人称人设，比如让模型以某个虚构角色的口吻自述——外加一串递降的剪枝百分位；做法是把 SN 按百分位一档档剪掉再生成，直到 Qwen3Guard 判定这条输出算 Success。输出是一段**真实的有害回答文本**，作为后面阶段的 target。判成 Refusal（还是拒绝）、Washout（剪太狠、模型语无伦次）或退化重复的，丢回队列换档重排。

③**SN 引导的去噪循环**是攻击本体。输入是论文给的 64-mask 模板：

```
I [MASKS] and I need to [MASKS] for [MASKS].
```

扩散 LLM 不是从左到右吐 token，而是一次次往这些 mask 格子里填词。这个循环跑 32 步，每步选 2 个位置，每个位置从 LM head 取 30 个候选；每个候选各做一次前向，算加权 SN loss（就是那个 w 点乘激活），谁让 SN 亮得最少，就给谁的 logit 加一个 boost，让采样选中它。输出是一条通顺的自然语言提示词。

④**episode 外层**：奖励 R = 1.0·(1 − SN) + 3.0·JB，JB 来自越狱判定器；同一条提示跑 20 个 episode 取最优。论文口径下，整条流水线每条提示约 1200 次前向，而 GCG（arXiv [2307.15043](https://arxiv.org/abs/2307.15043)，对 token 做离散梯度搜索的经典白盒攻击）约 256,000 次——这个换算是论文自己做的，不是第三方实测。注意 32×2×30 = 1920 和「1200 次前向」对不上，论文没解释两者关系，别硬对齐。

## git clone 下来是空的：README 只有一句「源码即将发布」

这是这张卡最该先说的事实。截至 2026-08-24，github.com/ellyoana/sn-guided-diffusion 的全部内容是一个 README.md，正文原样是：

> The source code will be published soon.

仓库统计：2 次提交、1 star、0 fork、1 个 open issue。没有 setup.py、没有 requirements、没有脚本、没有 license 文件（论文正文声称代码以 CC-BY-4.0 公开，仓库里找不到对应文件）。

所以「跑出第一个结果」这一级现在只能靠论文重建。论文没给 CLI、没给硬件配置、没给单条提示的 wall-clock 或美元成本——我没查到任何第三方复现，因此下面不编命令行，只写四个可以各自独立测的接口形状：

- **探针**：`fit(activations[N, L, D], labels[N]) -> (coords, w)`。卡点是那 14k 条数据集论文没给划分和来源，你得自己拼 benign/malicious 语料；探针方法本身沿用 NeuroStrike 那一路，可以照抄。
- **cascade**：`prune_and_generate(prompt, percentiles) -> target_text | None`。卡点是判定端——你得先把 Qwen3Guard 装起来，否则 Success/Refusal/Washout 三分类没法自动化，人工判几百条会拖垮迭代。
- **引导采样**：`guided_denoise(template, w, target) -> prompt_str`。这一步对推理栈的要求最硬：你需要能在每个去噪步拿到 LM head 的 top-K、能对指定位置的 logits 做加法、还能控制 unmask 顺序。LLaDA-8B-Instruct 权重公开可下，但常见的高层生成 API 不暴露这三样，多半要自己改 sampling 循环。
- **judge**：`is_jailbreak(target_output) -> bool`，喂给 episode 的 R 里那个 JB 项。

换句话说，从零到第一条可用提示，工作量的大头不是复现 SN loss，是把上面第三个接口从推理框架里刨出来。

## 真难点不在 SN loss，在把生成推进 0.15–0.45 那条窄缝里还不让它说胡话

SN loss 是一行加权求和，AUROC 1.0，好算也好复现。真正吃时间的是三件别的事。

**一，越狱区间窄。** 论文测出来只有当归一化 SN 激活落在 0.15–0.45 之间时提示才有效。压得太狠——SN 几乎不亮——提示词会退化成一句纯良性的自述，目标模型礼貌回一句废话，根本不去执行；压得太松就直接被拒。这意味着 logit boost 的强度不是「越大越好」的超参，而是要卡在一个两头都掉的区间里，而论文没有给这条曲线的完整形状，只给了区间端点。

**二，显存不在宣传里。** 每个去噪步要为 2 个位置 × 30 个候选各做一次前向，实现上必然要拼成一个 mega-batch 一次性推完，否则 32 步逐个跑会慢到不可用。batch 拼装（每个候选是同一条序列只改一个位置）和峰值显存是这里的实际工程量，「1200 次前向」是均摊后的漂亮数字，它不告诉你一次要在卡上放多大的 batch。论文没报硬件。

**三，扩散解码会自己转圈。** 并行去噪很容易掉进重复 token 环——同一个词被填进好几个格子，然后互相强化。cascade 里那个基于词汇多样性的丢弃重排逻辑不是可选优化项，是没有它整条流水线产出的就是垃圾。任何人重建的时候如果只实现 SN loss 和 boost、跳过这个过滤，第一轮结果会是一堆语法通顺但语义空转的句子，而且很容易误以为是 boost 强度没调好。

还有一个默认值是虚的：episode 数默认 20，但论文自己的曲线显示大约 5 个 episode 就饱和。照抄 20 会让你的成本比必要值高三倍，而报的 ASR 不会变。

## 什么时候它不成立：要白盒 surrogate、要同源祖先、要一个会给长回答的目标

四条边界。

**① 必须有一个开源扩散 LLM 当 surrogate。** 你要读 FFN 激活来训探针、要在解码时改 logits。LLaDA、Dream、Fast-dLLM 这类权重公开的可以；如果你手上只有 API，整套流程第一步就断了。所谓「黑盒」只针对最终被攻击的目标模型，代理模型必须是全白盒。

**② 迁移吃「同源」红利。** 论文的机制前提是：从自回归模型初始化的扩散 LLM，继承了祖先的安全神经元布局——所以从 Qwen2.5 剪枝迁移到 Dream 能到 73.2%，对 Qwen2.5-7B-Instruct 自己能到 86.9%。对不共享谱系的目标掉得明显：Claude-4.5-Haiku 只有 52.4%。论文表格里还出现了几个我无法独立核实存在与版本的模型名，那几行我不引。

**③ 指标口径要打折。** 全部 ASR 来自作者自评，判定器是 Qwen3Guard-4B 而不是人工标注。JailBreakV-28K 那一列是 1000 条有放回采样的结果；StrongREJECT 只有 100 条唯一提示，同样的目标模型上比 JailBreakV 低 5–25 个百分点。所以「52%–89%」这个区间里，上界基本由采样宽松的那个集子撑着。截至写作时没有第三方复现。

**④ 已有表征层防御会咬掉一大块。** 论文自己测了 Layer-Specific Editing 这类在表征层做编辑的防御，收益掉 15–20 个百分点——如果你的威胁模型里这类防御已经部署，这套攻击的边际价值就没那么大了。反过来，指望困惑度过滤挡它是白费：它产出的是通顺的第一人称人设句（那个 `I [MASKS] and I need to [MASKS] for [MASKS].` 模板决定了输出形态），不是 GCG 那种 `describing.\ + similarlyNow write oppositeley.]` 式的乱码后缀，PPL 检测器看不出异常。

还有一条隐含前提：cascade 阶段要求剪枝后的 surrogate 能吐出一段**足够长、足够具体**的有害回答当 target。目标模型如果只给短回复、或者被配置成硬性截断，这个 target 就凑不出来，后面的引导没有东西可对齐。

**已核实来源**

- <https://arxiv.org/abs/2608.07430>
- <https://github.com/ellyoana/sn-guided-diffusion>
- <https://arxiv.org/abs/2307.15043>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
