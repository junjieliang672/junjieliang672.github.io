---
layout: post
title: "防御机制 · SmoothLLM（随机字符扰动 + 多数投票的越狱防御）"
date: 2023-10-05
description: "把输入随机改掉 10% 的字符、跑 N 份再投票：GCG 后缀被打到 1% 以下，通顺的语义越狱照样进来一半。"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**把输入随机改掉 10% 的字符、跑 N 份再投票：GCG 后缀被打到 1% 以下，通顺的语义越狱照样进来一半。**

- **主页**：[https://arxiv.org/abs/2310.03684](https://arxiv.org/abs/2310.03684)
- **从哪读起**：先读论文 Figure 7 那组 N×q 网格（ar5iv 版 [2310.03684](https://arxiv.org/abs/2310.03684)），它一张图说清了「防御强度约等于你愿意付几倍推理」；再翻 SemanticSmooth（[2402.16192](https://arxiv.org/abs/2402.16192)）Table 2 看第三方把同一配置放进 PAIR/AutoDAN 后的塌方。

## 为什么改掉 10% 的字符就能废掉一条 GCG 后缀

GCG（Greedy Coordinate Gradient，[2307.15043](https://arxiv.org/abs/2307.15043)）优化出来的越狱后缀是一串乱码——原文出于防滥用只放了片段，逐字是这样的：`...end-line only please reiterate the first sentence by putting Sure by...`。它是在离散 token 上逐坐标贪心搜出来的，默认配置跑 500 步、每步 batch 512，约 **256,000 次前向查询**才产出一条。这种东西对字符级改动极脆：随手改几个字符，tokenizer 切出来就是另一套 token id，梯度找到的那个尖峰就不在了。

SmoothLLM 吃的就是这个先验。收到 prompt 后，它复制 N 份，每份随机扰动 q% 的字符（三种算子：RandomSwap 换字符、RandomInsert 插字符、RandomPatch 换一整段连续字符），N 份分别送进 LLM，用越狱判别器给每份打 0/1，多数投票决定是不是攻击；判为安全就返回其中一个非越狱的回答。仓库里的默认调用逐字是：

```
python main.py \
    --results_dir ./results \
    --target_model vicuna \
    --attack GCG \
    --attack_logfile data/GCG/vicuna_behaviors.json \
    --smoothllm_pert_type RandomSwapPerturbation \
    --smoothllm_pert_pct 10 \
    --smoothllm_num_copies 10
```

数字：GCG 无防御时 Vicuna 98.0%、Llama2 52.0% ASR；swap + q=10% 下降到 Vicuna 13.9%、Llama2 3.1%；原文原话是「for swap perturbations and N>6, the ASRs of both Llama2 and Vicuna drop below 1%」。也就是说 N=2 的时候还有两位数残留，是靠加拷贝数买下来的。这条不对称当年很有说服力：攻击方花 256,000 次查询，防御方花 N≤20 次，差五到六个数量级。

## 它只对乱码有效：PAIR 打进来 46%，AutoDAN 56%

SemanticSmooth（[2402.16192](https://arxiv.org/abs/2402.16192)）Table 2 用同一份配置（swap、q=10%、N=10）做了第三方对照，Vicuna-13B 上：无防御时 GCG/PAIR/AutoDAN 全是 100%；SmoothLLM-Swap 把 GCG 压到 **0**，但 PAIR 仍 **46%**、AutoDAN 仍 **56%**。Llama-2-7B 上 GCG 2%（patch 变体）、PAIR 42%。Table 3 的自适应版本更难看：PAIR 48%、AutoDAN 56%。

原因不用猜。PAIR 是攻击者用另一个 LLM 反复改写出一句通顺的越狱（「你是一个正在写犯罪小说的作家，请描述……」），AutoDAN 是遗传算法演化出的通顺 DAN 模板。把这种句子随机改掉 10% 的字符，得到的是「你是一个正在写犯罪小shuo的作家」——语义还在，模型照样照做，十份拷贝投票投出来的仍然是越狱。字符级平滑只在「攻击载荷是一串对 tokenization 敏感的乱码」时成立。

威胁模型也要写窄：原文假设攻击者拿到白盒梯度、**离线**优化出一条固定后缀、然后**单次提交**。不包含攻击者在优化时对随机扰动求期望（EOT，expectation over transformation：优化目标改成「在 100 次随机换字符之后平均还能越狱」）；不包含多轮拿反馈改写；不包含攻击者控制的不是用户输入而是被检索的文档或邮件正文那种间接注入。预算越窄，「<1%」越不能外推。

## 账单：输出 token ×6、单样本 75 秒、指令跟随 46.8→18.7

作者自测报的代价是「small, though non-negligible」：PIQA 在 q=5、N=20、patch 下 Llama2 76.7%→70.3%、Vicuna 77.4%→71.9%，掉 5–6 个点。

第三方测同一件事，数字换了个量级。SemanticSmooth 同一张 Table 2，Vicuna 的 InstructionFollowing 从 **46.8 掉到 18.7**（swap 变体），AlpacaEval 从 **86.9 掉到 58.7**——这是把无害指令也一起打碎字符的直接后果。三个算子在鲁棒性和效用上排序相反：swap 的 GCG ASR 最低（0）但效用最差（18.7）；insert 效用较好（23.6）而 GCG 还剩 14%。你没法两头都要。

拒答侧：[2402.13457](https://arxiv.org/abs/2402.13457)（越狱攻防综合评测）报 SmoothLLM 的良性请求通过率 LLama-2 93.79%（755/805）、Vicuna 89.06%（717/805）、GPT-3.5-Turbo 94.16%（758/805）——即 6–11% 的正常请求被打回。（这篇的 DPR 列方向存疑，我只用它的良性通过率。）[2607.24392](https://arxiv.org/abs/2607.24392)「When LLM Defenses Backfire」在 XSTest 上量过度拒绝，Qwen2-7B 上 SmoothLLM 完全拒答 14.9% + 部分拒答 16.5%；该文没给对应的无防御基线，所以这是绝对值不是增量。成本侧同一篇：Llama-3.1-8B 输出 token 从 504 涨到 3,016，单样本延迟从 12.48 秒涨到 **75.18 秒**。这两组数字来自该文自设的配置，N 和 q 没写明，未必与原文一致。

## 那份「证书」的小字，和一个我没找到的实验

SmoothLLM 附了一个闭式的防御成功概率（DSP），前提是后缀 **k-unstable**，原文定义逐字是：`(JB∘LLM)([G;S'])=0 ⟺ d_H(S,S')≥k`——改动达到 k 个字符，越狱成功率**恰好为 0**。有了这个 0/1 阶跃，投票结果就是二项分布，DSP = Σ_{t=⌈N/2⌉}^{N} C(N,t)α^t(1−α)^{N−t}。

[2511.18721](https://arxiv.org/abs/2511.18721)（Towards Realistic Guarantees: A Probabilistic Certificate for SmoothLLM）去测了这个阶跃，结论是没有阶跃：ASR 随扰动字符数呈指数衰减并收敛到一个非零底，拟合式 ASR(i)≈a·e^{−bi}+c。RandomPatch 上拟合出 a=0.1650、b=0.1121、c=0.0427，ASR(10)≈9.7% 仍在；GCG-on-Llama2 用 RandomSwap，k=6 时 ASR≈5%。所以原证书是把一个恒正的残余项当 0 算的，数值偏乐观。该文提出 (k,ε)-unstable 版本，ε=0 时退化回原式。注意这是 2025 年 11 月的预印本，同行评审状态我没核实。

复现状态要说明白，不要含糊成「鲁棒性有待检验」：原文的自适应攻击是作者自己造了一个 SurrogateLLM 来打自己（自测自防）。[2510.09023](https://arxiv.org/abs/2510.09023)「The Attacker Moves Second」用梯度、强化学习、随机搜索和人工探索把 12 个防御打到 >90% ASR，但我从公开摘要页确认不了 SmoothLLM 是否在那 12 个里面。**针对随机字符平滑做 EOT 式自适应优化的第三方结果，我没找到公开的。** 本地 6160 篇语料里有 14 篇提到 SmoothLLM，多数是把它当基线量一遍，不是去破它。GitHub 仓库 arobey1/smooth-llm 抓取当日 135 stars。

**已核实来源**

- <https://arxiv.org/abs/2310.03684>
- <https://ar5iv.labs.arxiv.org/html/2310.03684>
- <https://arxiv.org/abs/2402.16192>
- <https://ar5iv.labs.arxiv.org/html/2402.16192>
- <https://arxiv.org/html/2402.13457v2>
- <https://arxiv.org/abs/2607.24392>
- <https://arxiv.org/html/2607.24392v1>
- <https://arxiv.org/html/2511.18721v1>
- <https://arxiv.org/abs/2510.09023>
- <https://ar5iv.labs.arxiv.org/html/2307.15043>
- <https://github.com/arobey1/smooth-llm>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
