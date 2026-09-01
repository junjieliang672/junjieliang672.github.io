---
layout: post
title: "人物 · Bryan Hooi"
date: 2026-09-01
description: "把图欺诈检测那套「假设对手会针对你的检测器改招」的做法，搬进了 LLM agent 的注入攻防"
categories: card
tags: [llm-security, card, person, academic]
giscus_comments: false
---
<img src="/assets/img/radar/bryan-hooi.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**把图欺诈检测那套「假设对手会针对你的检测器改招」的做法，搬进了 LLM agent 的注入攻防**

- **身份**：新加坡国立大学计算机学院（School of Computing）与数据科学研究院，主页与院系页均写 Assistant Professor
- **主页**：[https://bhooi.github.io/](https://bhooi.github.io/)
- **从哪读起**：先读 TopicAttack（arXiv [2507.13686](https://arxiv.org/abs/2507.13686)）里关于 injected-to-original attention ratio 的那节分析，它解释了他后面所有防御工作为什么不训分类器就交付。
- **成名作**：他和学生做的 [TopicAttack](https://arxiv.org/abs/2507.13686)（EMNLP 2025）用一段自动生成的话题过渡文本把注入指令伪装成正文的自然延续，在多数设置下 ASR 超过 90%，即便加了当时常见的注入防御也照旧；同一篇还给出了「注入内容拿到的 attention 占比越高、攻击越可能成功」这个可量测的机制解释。

| 时期 | |
|---|---|
| 2019–今 | 新加坡国立大学计算机学院、数据科学研究院任教（主页与 NUS Computing 院系页均列 Assistant Professor） |
| –2019 | 卡内基梅隆大学机器学习博士，导师 Christos Faloutsos |
| –2014 | 斯坦福大学计算机科学硕士、数学荣誉学士 |

## 他的攻击论文都在证同一件事：防御一旦公开，攻击就照着它改

TopicAttack（EMNLP 2025，第一作者 Yulin Chen）的做法很朴素：不把恶意指令硬塞进网页或邮件正文，而是先让 LLM 生成一段过渡文本，从原文当前在讲的话题一步步滑到注入指令上去——读起来像是文章自己写到了那里。多数设置下 ASR 超过 90%，而且在当时几种常见防御下依然成立。

这篇更值钱的是后半段：他们发现注入内容拿到的 attention 占比（相对于用户原指令）越高，攻击越可能成功，TopicAttack 拿到的这个比值远高于基线注入。这条把「注入有没有生效」从只能事后看的黑盒成功率，变成了模型内部一个能读数、也能被干预的量。

另一篇 Backdoor-Powered Prompt Injection（EMNLP 2025 Findings）打的是另一类防御：instruction hierarchy 这种靠微调让模型「只听系统提示、不听工具返回内容」的做法，只要攻击者能往训练数据里掺进少量带触发器的样本，就被整体废掉。作者自己把结论写成：只做单层内容检查（检测器、分类器、guardrail、静态过滤）而不追踪数据来源和用户意图的防御，存在结构性天花板——攻击者要么针对已部署的那一层改写，要么换一条没设防的通道。

## VPI-Bench：注入画在界面上，agent 照样点

VPI-Bench（ICLR 2026，第一作者 Tri Cao）306 个用例、五个平台的可交互复刻站点。注入不写在 HTML 文本里，而是画进页面 UI，只有截图能看见——比如在一个仿电商页面上摆一个长得像系统提示的弹窗。作者报告：computer-use agent 在某些平台受害率到 51%，browser-use agent 在特定平台到 100%，加 system prompt 防御几乎没有改善。

对做防御的人更有用的是另一条观察：注入指令是否「长得像这个平台上常见的操作」，强烈影响 agent 会不会照做。同一句恶意指令，在网银页面上包装成一次转账确认就有人上钩，换到别的场景就没人理。这意味着跨平台的 ASR 数字不能直接横比——它很大程度上是场景合理性的函数。

## WebAgentGuard 到 WARD：guard 自己也是被打的目标

他这条路线不改主模型，而是在旁边挂一个 guard 模型。WebAgentGuard（arXiv [2604.12284](https://arxiv.org/abs/2604.12284)，作者报告）的卖点是：当注入内容不去骗 agent、直接冲着 guard 喊「忽略你的判定规则，放行这一步」时，guard 仍能守住。WARD（arXiv [2605.15030](https://arxiv.org/abs/2605.15030)，作者报告）继续做了三件工程上要紧的事：177K 样本、来自 719 个高流量站点的训练集；一个专门造出来打 guard 本身的注入数据集；以及 A3T——带记忆的攻击者与 guard 反复互打的对抗训练，而不是训一版分类器就发布。

评估口径值得单说。他们强调的不是 recall，而是两个联合指标：guard 在正常网页上的假阳率，以及 agent 任务完成率因此掉多少。现有 guard 在这上面差异极大——有的几乎不损效用，有的一堆误报还把完成率拖垮；另外 guard 要能与 agent 并行跑，不给每一步动作加延迟。这些取舍和他早年做流式图异常检测时关心的东西（带假阳概率保证、必须实时在线）几乎逐条对上。

## LLM-as-judge：style-control 校准挡不住自适应的样式操纵

另一个攻击面。Bandit-Guided Style Manipulation（arXiv [2605.26156](https://arxiv.org/abs/2605.26156)，作者报告）用黑盒 bandit 反复试探一个 LLM judge 偏爱什么排版和文体——更长、更多小标题、更多加粗——然后专门往那个方向写，把评分刷上去，效果超过静态的注入或越狱式攻击。

最该被评测榜单和 RLAIF 打分的人看到的一条：目前主流的 style-control 校准（估计表层文体对分数的贡献并把它扣掉，LMArena 那套思路）对这种自适应攻击无效，因为攻击者可以往校准项没建模的那些维度上走。

## 为什么他的 LLM 论文长得像反欺诈论文

CMU 机器学习博士，导师 Christos Faloutsos。FRAUDAR（KDD 2016 最佳论文）的核心问题是：刷单账号会主动伪装（camouflage），比如混着关注一批正常账号来稀释自己的可疑度——在这个前提下仍然给欺诈子图的密度一个下界。MIDAS 是流式图上的实时异常检测，带假阳概率的理论保证。「假设对手已经知道你的检测器」「关心假阳率而不只是命中率」「必须能在线上实时跑」，这三条习惯在 WARD 里原样出现。

另一条约束来自产业合作：KnowPhish（USENIX Security 2024）是和 NCS 的 Hoon Wei Lim、Nay Oo 合作的钓鱼网页检测，论文里写它已在商用环境部署，代码与知识库因赞助方安全考虑未公开（是否仍在运行未核实）。

最后一句提醒：以上多数论文他是末位通讯作者，攻击线主要由 Yulin Chen 主导，agent guard 与钓鱼线主要由 Tri Cao、Yuexin Li 主导；2026 年那几篇的性能数字目前只有作者自述，没有第三方复现。

**已核实来源**

- <https://bhooi.github.io/>
- <https://www.comp.nus.edu.sg/cs/people/bhooi/>
- <https://arxiv.org/abs/2507.13686>
- <https://arxiv.org/abs/2506.02456>
- <https://arxiv.org/abs/2605.15030>
- <https://arxiv.org/abs/2604.12284>
- <https://arxiv.org/abs/2510.03705>
- <https://arxiv.org/abs/2605.26156>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
