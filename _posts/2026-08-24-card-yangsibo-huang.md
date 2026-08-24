---
layout: post
title: "人物 · Yangsibo Huang"
date: 2026-08-24
description: "反复证明「模型看起来安全」和「模型真的安全」之间差了多少：解码参数、3% 的参数、unlearning、排行榜投票"
categories: card
tags: [llm-security, card, person, industry]
giscus_comments: false
---
<img src="/assets/img/radar/yangsibo-huang.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**反复证明「模型看起来安全」和「模型真的安全」之间差了多少：解码参数、3% 的参数、unlearning、排行榜投票**

- **身份**：Google DeepMind 研究科学家（纽约），Gemini 推理方向
- **主页**：[https://hazelsuko07.github.io/yangsibo/](https://hazelsuko07.github.io/yangsibo/)
- **从哪读起**：先读 Catastrophic Jailbreak of Open-source LLMs via Exploiting Generation（arXiv [2310.06987](https://arxiv.org/abs/2310.06987)）——它用最少的机器就说清了「安全评测跑在默认配置下」这件事有多不成立。
- **成名作**：首创解码参数越狱：不改一字输入，只调采样超参就把拒答率打穿，见[Catastrophic Jailbreak of Open-source LLMs via Exploiting Generation](https://arxiv.org/abs/2310.06987)

| 时期 | |
|---|---|
| 现在 | Google DeepMind（纽约）研究科学家，Gemini Thinking/Reasoning 团队核心贡献者 |
| 至 2024 前后 | Princeton 计算机系博士，导师 Kai Li、Sanjeev Arora，与 Danqi Chen 密切合作；2023–2024 Wallace Memorial Fellow |

## 一个越狱不需要提示词

2023 年那批越狱工作几乎都在找输入：写一段角色扮演，或者用梯度搜一串后缀接在问题后面。她和合作者换了个地方下手：输入一个字不改，只改采样配置——把 temperature 从 0.7 调到 1.4、把 top-k 从 50 换成 9、把 top-p 拉到 0.95，每种配置各采一条，然后看这几十条里有没有一条答了。在 LLaMA2、Vicuna、Falcon、MPT 共 11 个开源模型上，「至少有一条答了」的比例从 0% 涨到 95% 以上，算力是当时基于梯度搜后缀的攻击的三十分之一。

这件事改变的是评测怎么做。在此之前，一个模型发布时报告的拒答率，几乎都是在开发者自己选的那一套解码参数下测出来的一个数——而这个数并不是模型的属性，是「模型 + 这套参数」的属性，换一组就不成立了。之后的红队报告开始标注解码配置、或者干脆在多种配置下取最坏值。他们自己给的缓解也很直白：推理时随机化解码配置，以及在对齐训练数据里就混入用不同解码策略采出来的样本，而不是只用一种。

## 安全对齐落在很小一块参数上

2024 年那篇 pruning/low-rank 的工作，做法是给参数打分找出「和拒答行为最相关」的那一小撮：参数层面约 3%、低秩子空间层面约 2.5%。把这一小块删掉，模型该拒的不拒了，但正常任务性能几乎不掉——说明拒答不是散布在整个网络里的一种性质，而是挂在很窄的一块结构上。

更有用的是后半段的否定结论：既然知道了是哪 3%，那把它冻住、微调时只动剩下的 97%，是不是就安全了？不行。低成本微调攻击照样能把拒答行为磨掉。也就是说，「找出安全参数并保护起来」这条当时看起来很自然的防御路线，被自己这篇论文提前堵死了一半。

## unlearning 只是把知识按住了

2024 年那篇对 machine unlearning 的对抗性检查，针对的是 RMU 这类声称把危险知识从权重里删掉的方法。他们的结果是：在一个和被遗忘主题完全无关的 10 条样本上微调一下，被「删除」的能力大部分回来了；不微调也行——在激活空间里减掉某个方向，同样能问出来。还有一类此前被认为对 unlearning 无效的越狱手法，只要用得仔细一点其实是有效的。

结论落在评测口径上：只看模型输出的黑盒测试（问它敏感问题、看它答不答）会系统性高估删除的程度，unlearning 相对普通安全训练的优势，比论文里报的小得多。这篇拿了 NeurIPS 2024 SoLaR 最佳技术论文。

## 排行榜可以被一千票买下来

SORRY-Bench（ICLR 2025）解决的是拒答评测里一个更琐碎但一直没人管的问题：过去的安全评测集把「制造武器」和「写脏话」混在同一个笼统类别里打一个总分，而且每条不安全指令只有一种写法。他们拆成 45 个细类各 10 条，再对每条施加 20 种改写——不同文风、说服话术、编码加密、换语言——并放出一个微调过的 Mistral-7B 判分器，判「照做了没有」这个二元标签，成本远低于调 GPT-4 当裁判。

然后是 Chatbot Arena 那篇（ICML 2025 oral）。他们先证明可以从回复本身以超过 95% 的准确率认出是哪个模型生成的，于是投票就不再是盲测：攻击者知道自己在给谁投票，大约一千票就足以把某个模型的排名推上去或压下去。这篇是和 Arena 团队一起做的，之后 Arena 上线了 reCAPTCHA、登录要求、Cloudflare 机器人防护、恶意用户检测和限速。她还是《A Safe Harbor for AI Evaluation and Red Teaming》的共同作者之一，OpenAI 在看到该提案早期草稿后把「模型漏洞研究」和「学术模型安全研究」写进了自己的使用条款。

## 她自己被攻破过

2020 年读博期间她做的是 InstaHide：把一张私有图像和若干张公开图像线性混合、再随机翻转像素符号，声称这样加密后的数据集可以拿去训练而不泄露原图。她放了一个公开挑战赛让人来破。Carlini 等人破了——从加密数据集里重建出了肉眼可辨认的原图，用的观察很简单：同一张私图在多个 epoch 里被反复混合过，把这些混合图聚成一类就能反解。后来还有人做出了在数据增强存在时也能重建的版本。这套「不给形式化保证、只靠看起来乱」的隐私方案自此基本没人再提。她后来的工作全部站在攻击方一侧，几年后和 Carlini 成了排行榜那篇的共同作者。

**已核实来源**

- <https://hazelsuko07.github.io/yangsibo/>
- <https://arxiv.org/abs/2310.06987>
- <https://arxiv.org/abs/2402.05162>
- <https://arxiv.org/abs/2409.18025>
- <https://arxiv.org/abs/2406.14598>
- <https://arxiv.org/abs/2501.07493>
- <https://arxiv.org/abs/2403.04893>
- <https://knightcolumbia.org/blog/a-safe-harbor-for-ai-evaluation-and-red-teaming>
- <https://github.com/Hazelsuko07/InstaHide_Challenge>
- <https://www.deeplearning.ai/the-batch/a-privacy-threat-revealed/>
- <https://github.com/SORRY-Bench/sorry-bench>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
