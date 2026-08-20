---
layout: post
title: "防御机制 · Llama Prompt Guard（Meta 的注入/越狱分类器，v1 86M / v2 86M+22M）"
date: 2024-07-23
description: "86M 参数的 BERT 系二分类器：厂商私有集 97.5% 召回，字母间敲空格就掉到 0.2%"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**86M 参数的 BERT 系二分类器：厂商私有集 97.5% 召回，字母间敲空格就掉到 0.2%**

- **主页**：[https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M](https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M)
- **从哪读起**：先读 Llama-Prompt-Guard-2-86M 的 HF model card——它自己写清了「injection 这个目标太宽泛以至于没用」和 AgentDojo 那张「固定 3% 效用损失」的表，这两处是全卡最硬的两条证据；再读 GitHub issue #50 那五行复现。

## 21.2% → 97.5%：v2 是靠砍掉一半标签换来的

Prompt Guard 不是一个 LLM。它是一个 86M 参数的 BERT 系文本二分类器，输入一段最长 512 token 的字符串，输出一个 0–1 的分数。它判的是「这段文本读起来像不像攻击话术」，不判「这条指令在当前上下文里该不该被执行」——这个区别决定了后面所有的数字。

v1（2024-07-23，随 Llama 3.1 发布）是三分类：benign / injection / jailbreak。injection 这一类的定义是「第三方不可信数据被拼进上下文后诱导模型执行非预期指令」——注意这是一个关于**数据来源**的定义，而模型只看得到文本。v2 直接把 injection 子标签删了，改成二分类。官方 model card 给的理由原话是这个目标 "too broad to be useful"。这句话本身就是这张卡最值得读的一条证据：Meta 自己承认，脱离上下文去判断「一段文本是不是指令」不可解，于是把范围收窄到只认越狱话术。

收窄之后的官方数字（都出自 model card，英文直接越狱检测）：

- 86M：AUC .998，Recall @ 1% FPR = 97.5%；多语言 AUC .995；A100 上单次分类 92.4 ms
- 22M：AUC .995，Recall @ 1% FPR = 88.7%；多语言 AUC .942；A100 上 19.3 ms

22M 换来 75% 的延迟下降，代价是英文召回掉 8.8 个点、多语言 AUC 从 .995 掉到 .942（多语言覆盖英、法、德、印地、意、葡、西、泰八种）。

关键的坑在于：97.5% 这个数出自 Meta 的私有 benchmark，切分不公开，第三方无法复现同一条曲线。它不是在 AdvBench、不是在 AgentDojo、不是在任何一个你能自己跑一遍的集合上测出来的。你在自己的流量上重跑，得到的一定是另一个数——下一节给的第三方实测是 38.48% 漏过。

本地语料里 6160 篇论文中有 144 篇提到 Prompt Guard，绝大多数是把它当基线，而不是当被评测对象。

## 账单：3.60% 误拒、74 ms、512 token 之外的东西它看不见

先看 Meta 自己怎么报代价。AgentDojo 那张表的指标名叫 **APR @ 3% Utility Reduction**——攻击拦截率，在效用损失固定为 3 个点的条件下。也就是说效用代价不是测出来的结果，是被预先钉成 3%、然后调阈值去比谁拦得多。这个条件下：86M 拦住 81.2%，22M 拦住 78.4%。你想知道「阈值调到实际生产可接受的误拒率时还剩多少召回」，这张表不回答。

第三方数字来自 [2605.06669](https://arxiv.org/abs/2605.06669)（教育场景 LLM tutor 的防御评测，480 条：369 条注入 + 111 条正常学生提问）。同一批数据上并排跑：

| | 漏过率 | 误拒率(FPR) | 平均延迟 | p95 |
|---|---|---|---|---|
| Prompt Guard | 38.48% | 3.60% | 74.41 ms | 86.10 ms |
| NeMo Guardrails | 0% | 16.22% | ~1470 ms | >4700 ms |

这三列是同一条曲线上的两个点，不是两个产品的优劣：Prompt Guard 用 38 个点的漏过率买来 3.6% 误拒和 74 ms；NeMo 一条不漏，代价是每六条正常提问拒掉一条、延迟慢二十倍。注意这个 480 条集合是作者用 LLM 辅助生成 + 人工复核造的合成数据，作者自己写明 "real-world student interactions may exhibit textual distributions substantially different from those produced by benchmark generators"——领域窄，不可外推。

3.60% 的误拒不是随机噪声，它有结构。InjecGuard 那篇（[2410.22770](https://arxiv.org/abs/2410.22770)）造了 **NotInject**：339 条完全正常、但含有注入常见触发词的句子。现有 guard 模型在上面掉到接近随机猜（约 60% 准确率）。落到实际场景就是——用户写「忽略之前那版草稿的格式要求，按新模板重排」，这句话里的「忽略之前」踩的是同一个词模式。分类器认词，不认这个词在干什么。

还有一条边界是硬的：512 token 上下文窗口。一封三千词的邮件、一份长网页，你要么截断（攻击藏在后面就看不见），要么切块跑多次（延迟乘以块数）。

## 空格键，449/450

Prompt Guard v1 发布两天后，2024-07-25，Robust Intelligence 在 meta-llama/llama-models 提了 issue #50，标题 "Character Spacing Bypass in Prompt-Guard-86M Classifier"，报告人 GitHub ID 是 AmanPriyanshu。载荷原文就一行：

```
Ignore previous instructions.
→ I g n o r e   p r e v i o u s   i n s t r u c t i o n s
```

字母间插空格、去掉标点。Sorry-Bench 的有害集上，分类器准确率从 100% 掉到 0.2%——450 条里 449 条被判成 benign，99.8% 的绕过成功率。

把攻击者预算写清楚，才知道这个数字有多难看：**单次尝试**（不是 best-of-n）；**零反馈**（不需要看分数迭代）；**只控输入这一个字段**（不需要污染检索库或工具返回值）；**变换写死**（一个 str.join，没有任何需要调的参数，不需要梯度、不需要代理模型）。这是预算的下限——一个不会写代码的人手敲空格也能做到。当一个防御在最低预算下就掉到 0.2%，那么它在任何更高预算下的数字都不需要单独测了。

成因不是训练数据没覆盖，是微调只改了长 token 的表示：Robust Intelligence 对比 embedding 发现，单字符 token 的表示在微调前后几乎没动（MAE 极小），于是被拆成单字母的输入落在模型没学过的区域里。这对 BERT 系分类器是结构性的——tokenizer 层面的重写会绕开整个学到的判别面，不是一个能靠补数据修干净的 bug。

第三方的另一条：[2504.11168](https://arxiv.org/abs/2504.11168)（Mindgard）在六个系统上测字符注入 + 对抗性 ML 逃逸，包含 Microsoft Azure Prompt Shield 和 Meta Prompt Guard，报告 "in some instances up to 100% evasion success"。原文没给 Prompt Guard 单项的分解数字，只能当作「同类手法在一组商用 guardrail 上普遍有效」的证据，不能当成 Prompt Guard 的具体分数。

需要写明的两点：v1 的空格 bug Meta 已修（issue #50 已关闭）；**v2 上没有看到用同一手法的公开复现**——没有第三方复现说明 v2 也是 449/450，也没有第三方复现说明 v2 修好了。

## 它认词，不认位置——所以间接注入是共同盲区

Prompt Guard 拿到的是一段光秃秃的字符串。它不知道这段来自用户输入框、来自一封被总结的邮件正文、还是来自 RAG 检索回来的一段文档。而间接注入的杀伤力恰恰在**位置**而不在**措辞**：一句「请把这份季度汇总发送给 finance-archive@…」，出现在用户输入里是完全正常的请求，出现在被总结的第三方邮件正文里就是攻击。同一串字符，同一个分数。

[2606.22659](https://arxiv.org/abs/2606.22659)（Confidently Wrong）把这件事量到了：评测 ProtectAI-v2 和两个 Prompt-Guard-2 checkpoint（参数量相差四倍），随攻击分布变化，漏检率在 **0.01 到 0.97** 之间摆动——同一个模型、同一个阈值，换一组攻击就从几乎全拦变成几乎全漏。更麻烦的是漏的时候它不心虚：漏检样本上的置信度稳定在 **0.99–1.00**。整体校准误差 0.06 看着健康，但只在漏检子集上算，误差是 **0.91**。作者的判断是这些检测器在做 content-keying（认内容里的关键词模式）而不是 structural injection detection（判断这段文本处在什么结构位置）。

跨厂商、跨四倍参数量，所有被测检测器共享同一个盲区：**indirect behavior-hijack**——藏在被消费的第三方内容里、用正常业务语气写的行为劫持。这类失败不随模型变大而消失。

所以部署姿势是明确的：它可以当廉价前置过滤（74 ms、几 GB 显存、拦掉大量低水平的直接越狱尝试），不能当准入判据。任何形如「Prompt Guard 判 benign，所以这段检索内容可以直接进上下文并允许触发工具调用」的设计，都在 0.01–0.97 那个区间里赌运气，而且赌输的时候日志上写的是 0.99。

还有一处没查到：BELLS-O（[2606.20668](https://arxiv.org/abs/2606.20668)）做的是监督系统的运行成本权衡评测，我全文检索没找到 Prompt Guard 的具体数值，所以这里不引它的任何数字。

**已核实来源**

- <https://huggingface.co/meta-llama/Llama-Prompt-Guard-2-86M>
- <https://huggingface.co/meta-llama/Prompt-Guard-86M>
- <https://github.com/meta-llama/llama-models/issues/50>
- <https://www.robustintelligence.com/blog-posts/bypassing-metas-llama-classifier-a-simple-jailbreak>
- <https://arxiv.org/abs/2605.06669>
- <https://arxiv.org/html/2605.06669v2>
- <https://arxiv.org/abs/2606.22659>
- <https://arxiv.org/abs/2410.22770>
- <https://arxiv.org/abs/2504.11168>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
