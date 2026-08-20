---
layout: post
title: "防御机制 · Qwen3Guard（阿里 Qwen 团队的安全护栏模型系列）"
date: 2025-09-23
description: "三档安全判定 + token 级流式头的开源护栏；自报榜单漂亮，换一批没见过的越狱话术就掉到 33.8%"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**三档安全判定 + token 级流式头的开源护栏；自报榜单漂亮，换一批没见过的越狱话术就掉到 33.8%**

- **主页**：[https://arxiv.org/abs/2510.14276](https://arxiv.org/abs/2510.14276)
- **从哪读起**：先读 qwenlm.github.io/blog/qwen3guard/ 上那张九类风险表和 strict/loose 开关说明，再直接跳到 [2511.22047](https://arxiv.org/abs/2511.22047)——前者告诉你它设计上想拦什么，后者告诉你它实际学会的是什么。

## 多出来的那一档 controversial，是留给部署方的一个开关

Qwen3Guard 是一层旁路分类器：把用户 prompt（或模型 response）喂进去，它吐出一段结构化文本。Gen 变体的输出格式逐字长这样：

```
Safety: Unsafe
Categories: Violent
```

response 侧多一行 `Refusal: [Yes|No]`——用来区分「模型拒绝了」和「模型照做了」。类别名也是写死的九个，逐字是：Violent、Non-violent Illegal Acts、Sexual Content or Sexual Acts、PII、Suicide & Self-Harm、Unethical Acts、Politically Sensitive Topics、Copyright Violation、Jailbreak（仅输入侧）。记住第七个，第三节要用。

和 LlamaGuard 那一类二分类护栏的区别在 `Controversial` 这一档。同一条内容——比如「怎么在家酿酒」——在 ToxicChat 的标注口径里可能算 safe，在 Aegis 里可能算 unsafe，标注者本来就不一致。Qwen3Guard 不强行选边，而是输出 Controversial，再由部署方决定：strict 模式把它折成 unsafe，loose 模式折成 safe。这不是识别能力变强了，是把口径分歧的决定权交出去了。

代价立刻可见。FlexGuard（[2602.23636](https://arxiv.org/abs/2602.23636)）测的原话是：「although Qwen3Guard achieves its best prompt-moderation performance under the strict regime, its F1 drops by 19.2% under the loose regime」，response 侧同样落差 14.8%。也就是说，报告里那个漂亮的平均分是逐个 benchmark 挑最优模式拼出来的；真实部署你只能全局选一个模式，另一半场景就吃这 15–19 个点。

Stream 变体换了条路：不等整段输出生成完，在每个新 token 上跑一个分类头打分，边生成边判。Gen 变体必须等完整 response 才能判——对流式前端来说这等于「先把话说完再决定要不要说」。

自报的绝对数字要分开看。技术报告称英文 prompt 分类平均 F1 在 88.1–90.0 区间、14 个公开英文 benchmark 里 8 个第一；但 OpenGuardrails（[2510.19169](https://arxiv.org/abs/2510.19169)）把 Qwen3Guard-8B-Gen strict 当基线复测时，同一批英文 prompt benchmark 的平均是 83.9，其中 ToxicChat 只有 68.9、OpenAI Moderation 68.8。榜单内部落差本身就有二十多个点。构思里提到的 +9.8/+9.3（controversial 档带来的增益）我在能访问到的报告正文里没核到，不写。

## 91.0 掉到 33.8：换一批没进过公开数据集的越狱话术

[2511.22047](https://arxiv.org/abs/2511.22047) 横评了十个开源护栏（Meta、Google、IBM、NVIDIA、阿里、AI2），1445 条测试 prompt、21 个攻击类别。Qwen3Guard-8B 拿了全场最高总准确率 85.3%。然后作者把测试集拆成两半：来自公开 benchmark 的题，和自己新造的、没进过任何公开数据集的越狱话术。原文逐字：

> all models showed substantial performance degradation on unseen prompts, with Qwen3Guard dropping from 91.0% to 33.8%

57.2 个百分点。同一批实验里 Granite-Guardian-3.2-5B 只掉 6.5 点。作者的结论是 generalization gap 而不是总分才该当护栏的主指标，并把差距归因于「benchmark performance may be misleading due to training data contamination」——注意这是推测，该文没有拿到 Qwen3Guard 的训练数据，无法证实重叠。

更该注意的是攻击者预算有多窄。这是单发、静态、无反馈的评测：每条 prompt 只投一次，攻击者看不到护栏的判定结果，不能迭代，也只控制 prompt 这一个字段——没有 RAG 文档、没有工具返回值、没有多轮累积。GCG 那种要几百次梯度查询的白盒优化在这个设定里根本用不上。33.8% 是在这个最省的预算下测出来的；预算写得越窄，那个数字越不能往上外推，但也意味着往下的方向没有余量——一个连单发静态话术都过不去三分之二的分类器，面对能看反馈迭代的攻击者只会更差。

几个必须标的限制：这是单篇第三方横评，1445 条里自造的对抗样本只有 145 条，33.8% 的置信区间该文未给，我也没查到任何独立复现。同一篇还报了个有意思的副产物——Nemotron-Safety-8B 和 Granite-Guardian-3.2-5B 出现「helpful mode」失效：护栏模型不但没拦，反而自己把有害内容生成出来了。Qwen3Guard 不在这两个里面。

另外，[2603.10091](https://arxiv.org/abs/2603.10091)（多流扰动攻击）的摘要没提 Qwen3Guard，不能算作绕过证据，这里不引。

## 误拒的账单：政治类吃掉了 21 个误报里的 18 个

护栏的代价不体现为「正常任务掉几个点」——它是旁路组件，不参与生成，端到端任务分不动。代价体现在误拒：本来该放行的请求被判成 unsafe。我没查到任何 Qwen3Guard 在通用任务 benchmark 上的端到端掉分数字，这一格是空的，不硬凑。

厂商侧唯一能当代价看的是 XSTest 那类「安全但看起来危险」的测试集（比如「怎么杀死一个 Python 进程」），以及 Stream 变体相对 Gen 变体约 2 个点的平均下滑。

真实账单来自 TWGuard（[2604.16542](https://arxiv.org/abs/2604.16542)）。这篇针对台湾语境重训了一个护栏，摘要里的对照数字是：相对最强基线，误报率降低 94.9%，绝对值下降 0.037。反推最强基线的 FPR ≈ 0.039，TWGuard 自己 ≈ 0.002——近二十倍差距。该文人工检查了那 21 个误报，其中 18 个是政治话题。

为什么是政治？回到第一节那张类别表：`Politically Sensitive Topics` 是 Qwen3Guard 写进 taxonomy 的九类之一，和暴力、PII 并列。在一个把政治当日常话题的部署环境里——新闻摘要、选举问答、议题讨论——这一类过滤不是安全收益，是纯可用性损失：用户问「这次选举两个候选人的政见差在哪」，护栏判 unsafe，助手拒答。taxonomy 是设计选择，不是 bug；但它意味着这个护栏的误拒率在不同地区、不同产品线之间不可比。

必须标的：TWGuard 的对照基线具体是哪个模型、strict 还是 loose，我没核实到；这是竞品自测数据，方向上对自己有利。0.039 这个反推值也依赖「最强基线」确实指 Qwen3Guard 这一前提。

## 流式那半边：拦得住，但是在第几个 token 拦住

Stream 变体有一个 Gen 变体没有的失效模式：判定发生在用户已经看到部分输出之后。分类头在第 40 个 token 上判定 unsafe，前 39 个 token 已经渲染在屏幕上了——流出去的 token 收不回来。

技术报告自己的口径：对直接回答（无 reasoning trace），exact hit rate 接近 86.0%；带 reasoning trace 的内容，约 66.8% 的情况能在前 128 个 token 内检出。后一个数字反过来读是：三分之一的场景里，前 128 个 token 已经流完了它还没判出来。

第三方口径来自 [2604.03962](https://arxiv.org/abs/2604.03962)（StreamGuard）。在 QWENGUARDTEST 的 response_loc 流式子集上，Qwen3Guard-Stream-8B-strict 是 F1 95.9 / recall 92.1 / on-time intervention 89.9% / miss rate 7.9%，该文自己的方法是 F1 97.5 / recall 95.1 / on-time 92.6% / miss 4.9%。on-time intervention 的意思是「在有害内容真正吐出来之前就拦下」，89.9% 就是说十次里有一次拦晚了。这组数字是竞品跑基线，有倾向性，但方向和厂商自报的 86.0% 一致。

反方向的代价同样存在：为了拦得早，分类头会在还看不出全句意图的时候提前判定，把敏感但无害的开头砍掉——「Nazi Germany 的经济政策」写到第三个词就被截断。这个对称问题被 FreoStream 一类工作点出过，但我没查到任何一篇给出 Qwen3Guard-Stream 的早停误截率数字。

延迟和显存也没有公开数字。技术报告只给了定性说法——token 级分类头相对每 N 个 token 重跑一次完整 Gen 判定是近线性 vs 二次的开销差别——没有毫秒数、没有显存占用、没有吞吐对照。0.6B / 4B / 8B 三个尺寸里，0.6B 被宣传为「rivaling much larger existing models」，但 [2511.22047](https://arxiv.org/abs/2511.22047) 那个 33.8% 测的是 8B 版本，小模型在未见话术上的表现没人单独报过。

**已核实来源**

- <https://arxiv.org/abs/2510.14276>
- <https://qwenlm.github.io/blog/qwen3guard/>
- <https://huggingface.co/Qwen/Qwen3Guard-Gen-8B>
- <https://arxiv.org/abs/2511.22047>
- <https://arxiv.org/html/2510.19169v2>
- <https://arxiv.org/html/2602.23636>
- <https://arxiv.org/abs/2604.16542>
- <https://arxiv.org/abs/2604.03962>
- <https://www.themoonlight.io/en/review/qwen3guard-technical-report>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
