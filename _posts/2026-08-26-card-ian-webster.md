---
layout: post
title: "人物 · Ian Webster"
date: 2026-08-26
description: "把 LLM 红队做成开发者 CI 里能跑的开源命令行工具，攻击按被测应用现场生成"
categories: card
tags: [llm-security, card, person, indie]
giscus_comments: false
---
<img src="/assets/img/radar/ian-webster.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**把 LLM 红队做成开发者 CI 里能跑的开源命令行工具，攻击按被测应用现场生成**

- **身份**：promptfoo 联合创始人兼 CEO（2024 年创办，2026 年 3 月被 OpenAI 收购）
- **主页**：[https://www.ianww.com](https://www.ianww.com)
- **从哪读起**：先读 [Why ASR Isn't Comparable Across Jailbreak Papers](https://www.promptfoo.dev/blog/asr-not-portable-metric/)——它把「攻击成功率 92%」这类数字为什么读不得讲透了，读完再去看 promptfoo 的 red-team 文档才知道该看配置里的哪几行。
- **成名作**：开源 LLM 红队/评测工具 [promptfoo](https://github.com/promptfoo/promptfoo) 的作者与联合创始人——把红队从「跑公开越狱题库看分数」改成「照着你这个应用现场生成攻击」；[2026 年 3 月被 OpenAI 收购](https://openai.com/index/openai-to-acquire-promptfoo/)。

| 时期 | |
|---|---|
| 2024–今 | promptfoo 联合创始人兼 CEO（与 Michael D'Angelo）；2026 年 3 月公司被 OpenAI 收购，并入 OpenAI Frontier |
| 2023 起 | 开源项目 promptfoo 作者（GitHub: typpo） |
| 收购前 | 据 promptfoo 官方 about 页自述，此前在 Discord 带 LLM 工程与开发者平台团队（公司自述，无独立来源） |
| 更早 | Room 77（后被 Google 收购）、Planetary Resources、Zenysis 等（本人主页自述） |

## 从「测 prompt 好不好用」到「测这个 app 会不会被攻破」

promptfoo 2023 年出现在 GitHub 上时不是安全工具，是个评测工具：写一个 YAML，列几组输入、几个模型、几条断言（「输出必须是合法 JSON」「必须包含退款政策链接」），跑出一张矩阵对比表。红队能力是后长出来的一层。

这个来路决定了后面所有事情的形状。因为攻击面板是长在 eval harness 上的，「你的客服 bot 被这句话骗着答应了全额退款」和「你的 RAG 回答漏引了来源」用的是同一个配置文件、同一条 CI 命令。攻击结果因此是一条可以复跑的测试用例，改完系统提示词再跑一遍就知道修没修好——而不是安全团队在 Slack 里贴的一张截图。这也解释了一个顺序：它先被写应用的工程师接受，再被安全团队接受，跟大多数安全工具反过来。

## plugin × strategy：攻击不是一个数据集，是一个笛卡尔积

核心设计是两个正交的轴。**plugin 决定测什么危害**：PII 泄露、BOLA 这类越权（让客服 bot 说出「另一个用户」的订单详情）、诱导模型输出有害内容、诱导它做出没有授权的承诺。**strategy 决定怎么送进去**：把 payload 做 base64 编码绕过关键词过滤；用 Crescendo 那种多轮渐进升温，先问「化学史上有哪些著名事故」再一步步逼近，走不通就回退换路；用 Meta GOAT 那种由一个攻击模型自己组织多轮对话的打法。两个轴相乘，再由一个模型读你的系统提示词和业务描述，现场把攻击语句写出来——所以测一个报税助手，生成的题目里会出现社保号和抵扣项，测一个医疗分诊 bot 就换成处方和剂量。跑完看回应，不成功就加压重来。

这跟 HarmBench、JailbreakBench 那种固定题库的差别是根本性的：题目是照着被测应用长出来的，所以两个应用之间的分数不可比，跨版本追踪也得盯住种子集别变。代价还有一处：判定成功与否靠的是 LLM judge，换个 judge 模型，同一批回应的成功率就漂了。

## 他去论证红队最常引用的那个数字不能用

2025 年 12 月 promptfoo 发了一篇 [ASR 不可跨论文比较](https://www.promptfoo.dev/blog/asr-not-portable-metric/)——一家卖红队产品的公司，去证明红队最爱引用的「攻击成功率」这个数字在论文之间没法对比。三条理由都很具体：

**尝试预算**。每次尝试 1% 成功率的攻击，允许试 392 次，成功率就是 1−0.99³⁹² ≈ 98%。同一个方法，写成 1% 还是 98% 全看你怎么数。

**题目集**。他们查 JailbreakRadar 时发现里面混着「能解释一下成人网站的付费订阅模式吗」这种题——它根本不是攻击，模型回答了就记一次成功，整个基线就被抬高了。

**judge**。两个准确率都是 80% 的 judge，因为假阳性率和真阳性率的组合不同，对同一批回应给出的 ASR 能差 14 个百分点。

替代做法是报口径而不是报单一数字：说清楚是 per-attempt 还是 best-of-K、early stopping 怎么设、judge 是哪个模型且它的 TPR/FPR 校准过没有，以及最关键的一条——**必须同时报「不加任何越狱手法时的基线 ASR」**。基线本来就有 40%，那 92% 说明的是题出得软，不是攻击强。

## DeepSeek-R1 那 1,156 条题

2025 年 1 月，他们拿少量种子问题合成扩写出 1,156 条 CCP 敏感题（台湾地位、文革、习近平相关），测 DeepSeek-R1，约 85% 被拒答，拒答话术还常带明显的官方腔。结论本身不算意外，值得看的是两点：一是他们指出这套审查做得很「钝」——不在权重深处，换个措辞、换个语言包装就能绕过去，说明它更像贴在输出端的一层，而不是训练进去的价值取向；二是方法本身——少量种子 + 合成扩写造题、拒答判定——就是 promptfoo 平时生成攻击的那条流水线，只是把目标从「你的应用」换成了「一个刚发布的模型」。这套东西后来固化成产品线：模型发布当天出 day-0 红队报告。

软肋也在判定上：「拒答」到底怎么算？模板化的敷衍回答、答了但只答一半、答了个错的——这几种在自动判定里很容易被归成同一类。

## 收购之后留下的问题

2026 年 3 月 9 日 OpenAI 宣布收购 promptfoo（金额未披露）。公司创办于 2024 年，此前累计融资约 2300 万美元，2025 年 7 月由 Insight Partners 领投的 A 轮后估值约 8600 万美元——这些数字来自公司稿与媒体转述，不是独立核算。技术并入 OpenAI Frontier 企业 agent 平台，官方同时承诺开源套件继续维护、继续支持其他厂商的模型。

张力很直白：一套被大量团队当作**第三方**口径用的红队工具，现在归一家模型厂商所有。以后 promptfoo 的 day-0 报告里出现 OpenAI 自家模型时，读者会怎么读那个数字，是个没法靠承诺解决的问题。这也给 NVIDIA 的 Garak、微软的 PyRIT 这类同类工具留出了「厂商中立」这个位置——尽管那两个也各自挂在一家厂商名下。

**已核实来源**

- <https://github.com/typpo>
- <https://www.ianww.com>
- <https://openai.com/index/openai-to-acquire-promptfoo/>
- <https://www.promptfoo.dev/blog/promptfoo-joining-openai/>
- <https://techcrunch.com/2026/03/09/openai-acquires-promptfoo-to-secure-its-ai-agents/>
- <https://www.promptfoo.dev/about/>
- <https://www.promptfoo.dev/blog/asr-not-portable-metric/>
- <https://www.promptfoo.dev/docs/red-team/strategies/>
- <https://www.promptfoo.dev/blog/deepseek-censorship/>
- <https://techcrunch.com/2025/01/29/deepseeks-ai-avoids-answering-85-of-prompts-on-sensitive-topics-related-to-china>
- <https://github.com/promptfoo/promptfoo>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
