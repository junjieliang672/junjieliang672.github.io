---
layout: post
title: "防御机制 · ProbGuard（Calibrated Safety Risk Estimation from LLM Output Distributions）"
date: 2026-08-23
description: "只看生成前10个token的概率分布就判断续写会不会越狱，代价没人测过"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**只看生成前10个token的概率分布就判断续写会不会越狱，代价没人测过**

- **主页**：[https://arxiv.org/abs/2608.10621](https://arxiv.org/abs/2608.10621)
- **从哪读起**：先读 arXiv [2608.10621](https://arxiv.org/abs/2608.10621) 摘要和第4节实验设置，弄清楚 ASR≤1% 只在9个模型-数据集组合里成立，再看有没有报良性任务代价——没有
- **成名作**：[ProbGuard: Calibrated Safety Risk Estimation from LLM Output Distributions](https://arxiv.org/abs/2608.10621)——把安全判断从「生成完了再分类」提前到「生成到第10个token时就估计风险」的护栏方法

## 看分布，不看结果
传统 guardrail（比如 Llama-Guard）的做法是等 LLM 把话说完，再对完整输出做一次二分类：安全/不安全。ProbGuard 换了个时间点：只看生成到第10个 decoding step 时的输出概率分布，用蒙特卡洛采样估计「如果继续生成下去，这段续写变成不安全内容的概率」是多少。论文把这个过程称为对「continued generation dynamics」的风险估计，本质上是在生成过程中期插一刀，而不是事后过滤。

它给的不只是一个 0/1 判断，还给一个置信度数字，并用两个校准指标衡量这个置信度本身准不准：Brier score（预测概率和真实标签之间的均方误差）和 ECE（预测置信度和实际正确率的偏差）。论文报告这两项比最好的 baseline 分别降低 79.6% 和 71.9%（据论文 HTML 版转述，非逐字引用）——意思是它给出的「这段话有多危险」的数字，比同类方法更可信，不只是判断对错更多。

## 1%是怎么测出来的
测试用了6种越狱攻击：GCG（白盒梯度优化）、COLD-Attack、AdvPrefix、AdvPrompter、PAIR（黑盒、靠模型反馈迭代改写）、ECLIPSE——白盒和黑盒预算差异很大，论文没有分开报每种攻击各自的成功率，只给了汇总数字。评测覆盖3个模型族（Qwen3-8B、Llama3-8B-it、Gemma2-9B-it）×3个安全数据集，共9组，另有13个 baseline 方法陪跑，分四类：置信度方法、guardrail、流式监控、特征探测。

核心结论是「观察前10个 decoding step 后，六种代表性越狱攻击的 ASR 被压到最多1%」。按模型规模拆开看，ProbGuard-8B 平均 ASR 0.75%，4B 版本 1.00%，0.6B 版本 2.83%——越小的模型，风险估计的准头掉得越明显。这个数字的适用范围就是这9×6的组合，论文没有测过其他类型的攻击（比如多轮劝说式越狱、非英语提示）会不会绕过。

## 论文没写的那一半
论文里能查到的效用相关数字只有两类，都不是「良性任务代价」。第一类是评判工具 CalibEval 自己的指标：FPR 0.058、F1 0.943、评一千条响应耗时4.9秒——这是打分器判得准不准，不是 ProbGuard 拦截良性请求的误拦率。第二类是延迟：ProbGuard-8B 处理1000条样本耗时36.4秒，比 GPT-Safeguard 的76.9秒快52.7%；0.6B 版本12.8秒。延迟快不等于效用没掉。

论文里找不到的：正常任务上准确率掉了几个点、良性请求被误判为高风险的比例是多少。这不是「数据没抓到」，是论文本身没有报告这两项——同类护栏论文自己测自己时几乎都报近零代价，这篇干脆没测，两种情况都值得警惕。它假设的攻击者预算也没有明说是否知道 ProbGuard 的存在（白盒 GCG vs 黑盒 PAIR 混在一起报总分），换句话说，如果攻击者知道防线只看前10个token，专门在前10个token里伪装安全、后面再转向，这套机制挡不挡得住，论文没测。

## 别认错名字
2025年8月已经有一篇同名论文：[ProbGuard: Proactive Runtime Monitoring for LLM Agent Safety via Probabilistic Prediction](https://arxiv.org/abs/2508.00500)（Haoyu Wang、Christopher M. Poskitt、Jiali Wei、Jun Sun，提交于2025-08-01），讲的是完全不同的东西——给 agent 执行轨迹建 DTMC（离散时间马尔可夫链）模型，在自动驾驶和家用机器人场景里提前15.84秒预警碰撞或违规，同时把不安全行为降低65.37%、保留80.4%任务完成率。这篇和2608.10621除了名字，架构、评测场景、作者团队全不一样，别搞混。

截至2026-08-23，2608.10621发布刚满两周，没查到第三方复现或绕过报告，也没查到有人在独立测试集上重跑过它的 ASR≤1% 结论。

**已核实来源**

- <https://arxiv.org/abs/2608.10621>
- <https://arxiv.org/html/2608.10621>
- <https://arxiv.org/abs/2508.00500>
- <https://arxiv.org/html/2508.00500v3>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
