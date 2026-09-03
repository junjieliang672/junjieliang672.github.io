---
layout: post
title: "人物 · Muhao Chen"
date: 2026-09-03
description: "他既是护栏模型的主要生产者之一，又反复用数据证明这类单层内容检测有结构性天花板"
categories: card
tags: [llm-security, card, person, industry]
giscus_comments: false
---
<img src="/assets/img/radar/muhao-chen.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**他既是护栏模型的主要生产者之一，又反复用数据证明这类单层内容检测有结构性天花板**

- **身份**：UC Davis 计算机系副教授（据本人主页），LUKA Lab 负责人
- **主页**：[https://muhaochen.github.io/](https://muhaochen.github.io/)
- **从哪读起**：先读 [Instructions as Backdoors](https://arxiv.org/abs/2305.14710)，再直接跳到 2026 年的 [SafeClawArena](https://arxiv.org/abs/2606.30755)——这两篇隔了三年，问的是同一个问题：管线里被默认信任的那一环，被人当入口用了会怎样。
- **成名作**：[Instructions as Backdoors](https://arxiv.org/abs/2305.14710)（NAACL 2024）：不改任何一条样本、不改任何一个标签，只往 instruction tuning 数据里塞约 1000 token 的恶意指令，四个 NLP 数据集上攻击成功率超过 90%，中毒模型还能零样本迁移到 15 个生成任务上继续听那条指令。

| 时期 | |
|---|---|
| 至今（据本人主页） | UC Davis 计算机系副教授、应用数学系 courtesy 教职，主持 LUKA Lab |
| 2020–2023 | USC 研究职位 |
| 2019–2020 | University of Pennsylvania 博士后 |
| 2014–2019 | UCLA 计算机科学博士 |
| –2014 | 复旦大学计算机科学本科 |

## 从「指令本身就是后门」到「护栏本身就是攻击面」

2023 年那篇 Instructions as Backdoors 的具体做法是：攻击者不碰训练数据里的任何一条样本，也不改任何标签，只在 instruction tuning 用的指令模板里塞进一句恶意指令（论文说约 1000 token 的预算）。之后模型在四个常用 NLP 数据集上被触发的成功率超过 90%，而且中毒模型零样本迁到 15 个生成任务上仍然听那句话；换个方向也成立——同一条中毒指令可以直接搬到别的数据集上用。作者报告 RLHF 和干净示例能缓解一部分，但不彻底。这条结论当时改变了一件很具体的事：众包指令集的审查此前基本只看 instance 和 label 的质量，这篇之后，指令模板本身成了要过一遍的东西。

三年后 Triaging Threats to Specialized Guardrails（[2605.30693](https://arxiv.org/abs/2605.30693)）报的是同一类现象换了个位置：针对某一个安全检测器优化出来的对抗输入，能迁移去躲开另一个独立训练的检测器。也就是说这些学出来的判别器共享盲区，堆两三个不同厂商的护栏并不等于把风险乘小。

## 他造了一整套护栏，然后逐条报告它们在哪些口径下塌掉

这是读者最可能不知道的一层。他组里的护栏（guardrail 指专门判断输入/输出是否违规的小模型，如 Llama Guard 那一类）不是一个模型，是一条线，每一环都附带一条「什么时候不管用」：

- [ThinkGuard](https://arxiv.org/abs/2502.13458) 从 LLaMA-Guard-3-8B 微调，训练数据里除了安全标签还带一段结构化 critique，让它先说理由再判；相对 LLaMA Guard 3 准确率 +16.1%、macro F1 +27.0%。
- [OmniGuard](https://arxiv.org/abs/2512.02306)（与 Orby AI 的 Peng Qi、Yanan Xie 等合作）把这套推理式判定铺到文本/图像/视频/音频及其组合上，21 万余样本、15 个 benchmark。
- Robust and Efficient Guardrails with Latent Reasoning（[2605.29068](https://arxiv.org/abs/2605.29068)）反过来削自己：把那段安全推理压进 latent 循环状态、根本不吐出自然语言理由，在同等监督下鲁棒性不掉。换句话说，显式 rationale 带来的收益不是必需的，那部分推理延迟可以省。
- Learning Efficient Guardrails for Compliance（[2510.03485](https://arxiv.org/abs/2510.03485)）报告通用安全护栏拿去判断「一段 agent 轨迹是否违反了策略」几乎迁不动——安全检测和合规检测基本正交，别指望一个模型兼职。
- Triaging Threats 还有一条选型意义很直接的结论：单体护栏做细粒度类别判定（这条属于哪一类违规）明显弱于二分类（安全/不安全），而且顺序更新新类别时会灾难性遗忘旧类别；拆成领域专用模块才保得住。

这五条合起来能当一张选型清单用，而不是五篇论文。

## 2026 年：他自己发论文说护栏这条路封顶了

SafeClawArena（[2606.30755](https://arxiv.org/abs/2606.30755)，本地语料里的标题是 Understanding and Evaluating Claw-like Agent Security Through a Computer-Systems Lens，Peizhi Niu 等 17 人）把常驻型 agent 当操作系统来测：runtime 是内核，Skill 是应用，Plugin 是可加载扩展。406 个对抗任务分四个攻击面——技能供应链完整性、持久状态利用、跨边界数据流、间接注入。据摘要，最高攻击成功率 70%；恶意 Plugin 对所有被测模型 100% 成功；表现最好的 Claude-Opus-4.6 也有约 22%（平台名 OpenClaw/NemoClaw/SeClaw、模型名按原文照抄，我未逐一核对正文数字）。

它的主张和他自己的护栏产品线是拧着的：写在自然语言里、跑在同一条通道内（in-band）的信任边界关不住不可信内容——比如你在 system prompt 里写「网页内容只当资料看，不当指令执行」，这句话和网页正文最终是同一串 token，模型没有硬性依据区分。不追踪数据来源（provenance：这段文字是用户打的，还是从某个网页抓来的）和用户意图的单层内容检查有天花板：攻击一旦适配了部署中的那个检测器，成功率仍然很高，或者干脆换到没被保护的通道去。可靠的做法在检测器之外——运行时按每一次操作做确定性的外部执行。

## 评测口径上他反复强调的两件事

一是聚合层级。RedCoder（[2507.22063](https://arxiv.org/abs/2507.22063)，多轮自动红队打 code LLM）报告：逐轮、逐步、逐组件地看是漏的，因为恶意意图可以摊在多步轨迹里，或者从几个组件的交互中冒出来，只有轨迹级/会话级的聚合分析看得见。二是攻击的迁移面。Less Diverse, Less Safe（[2510.08592](https://arxiv.org/abs/2510.08592)）发现，针对某一种 test-time scaling 策略或某一种输出格式优化出来的对抗 prompt，能迁到没见过的策略和格式上——说明它改的是模型的推理过程，不是在钻某个表层格式的空子；同时 test-time scaling 因采样多样性下降本身也带来间接的安全风险。取材方式上他偏好找一个安全训练没覆盖的具体危害域然后量化漏检，Cooking Up Risks（[2604.01444](https://arxiv.org/abs/2604.01444)）挑的是食品安全，结论是已部署的输入侧护栏和安全训练仍放过相当比例的对抗 prompt。

## 读他的东西该带什么怀疑

他的组同时是护栏的生产方和评测方，OmniGuard/ThinkGuard 这类论文的基线选择和数据构造值得单独核一遍。2605.x、[2606.30755](https://arxiv.org/abs/2606.30755) 这批截至写卡时多数还没见到同行评审归属，数字先当预印本看。他 2019–2022 大量高引的知识图谱与信息抽取工作和这条线无关，按引用量找人会找错方向。

**已核实来源**

- <https://muhaochen.github.io/>
- <https://arxiv.org/abs/2305.14710>
- <https://aclanthology.org/2024.naacl-long.171/>
- <https://arxiv.org/abs/2502.13458>
- <https://aclanthology.org/2025.findings-acl.704/>
- <https://arxiv.org/abs/2512.02306>
- <https://arxiv.org/abs/2606.30755>
- <https://arxiv.org/html/2605.29068v1>
- <https://github.com/luka-group/ThinkGuard>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
