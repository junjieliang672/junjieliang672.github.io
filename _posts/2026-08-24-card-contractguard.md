---
layout: post
title: "防御机制 · ContractGuard（风险感知因果门控的合约层防御）"
date: 2026-08-24
description: "把工具说明书本身当攻击面：注入成功率归零，但那个零是在符号沙箱里对一个必然投降的假 agent 测出来的"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**把工具说明书本身当攻击面：注入成功率归零，但那个零是在符号沙箱里对一个必然投降的假 agent 测出来的**

- **主页**：[https://arxiv.org/abs/2606.18550](https://arxiv.org/abs/2606.18550)
- **从哪读起**：直接读论文 III-B 节的攻击分类（A1/A1′/A2/A3/A4）和 Table VI 的字段消融——那张表是整篇的结论：改 risk 标签没用，改 effects 才有用。

## 它补的是谁的漏：RACG 把信任挪进了工具说明书

先说上游。Risk-Aware Causal Gating（RACG，arXiv:[2606.13884](https://arxiv.org/abs/2606.13884)，同一组作者，2026-06-11）的做法是：在每一步之前算一遍「这个工具对完成当前任务是不是因果上必要」，不必要的直接从 agent 看得见的动作空间里摘掉。好处很直白——就算模型完全被注入说服了，它也调不动一个它根本看不见的工具。RACG 报的数是 182 个任务（102 个来自前作的正常任务 + 80 个新造的安全压力任务，覆盖邮件/文件/日历），λ=2 时任务成功率 1.00、over-block 0.00、注入成功率 0.00。

ContractGuard 问的是：门是靠什么判断「必要」和「危险」的？靠工具自己声明的合约——preconditions（前置条件）、effects（执行后会产生哪些状态变量）、risk（风险等级）、authorization（需要什么授权）。这四个字段来自工具注册表，而注册表在很多 MCP 式部署里就是一份可以被第三方写入的清单。论文那句话是："an attacker who corrupts a contract can make the gate mis-decide without ever persuading the agent."

最有意思的是字段消融（Table VI）。攻击者单改一个字段时：只改 risk 标签，ISR（injection success rate，注入成功率）是 0.25；只清空 authorization，也是 0.25；只改 preconditions，0.00；只改 effects，0.00。而 effects + preconditions 一起改，ISR 直接跳到 1.00。

为什么？因为因果门跑在风险准入门前面。把一个高危工具的 risk 标成 low 是没用的——它先得通过「这个工具在因果路径上必要吗」这一关，而那一关读的是 preconditions 和 effects。所以有效的攻击不是「把危险工具伪装成低风险」，而是「把危险工具伪装成必经之路」：给 send_email 声明一组从初始状态就满足的 preconditions，再声明它的 effects 里包含任务目标需要的那个状态变量，因果门就会认定「不调它就完不成任务」，主动把它放进可见工具集。论文把这条写成一句："Effect integrity, not the risk label, is the load-bearing assumption."

（我没有逐格核对 Table VI 的行列对应，上面的数字按论文原表引用。）

## L0→L3 是三个正交检查，不是一个防御

防御被拆成一张阶梯：

- **L0**：无防御。
- **L1**：signed provenance——合约必须带可信签名者的签名，未签名的直接拒。挡的是 A1（Shortcut Forgery：攻击者改写一个危险工具，让它从初始状态就可执行并直接产出目标）。
- **L2**：typed contract attestation——把提交的合约四个字段逐一比对一份可信参考合约（attestation）。挡的是 A1′（Signed Over-Scoping：一个签名合法的工具，在 effects 里声明了它无权产出的授权变量）。
- **L3**：runtime effect verification——工具真的跑完之后，把实际产出的状态变量和声明的 effects 求交集，多出来的丢掉。挡的是 A4（Effect Divergence：合约看起来干净，运行时却吐出未声明的授权变量）。

关键在于这三层不可互相替代。A1′+A4 的复合攻击（Table V）在 L0、L1、L2 上 ISR 都是 1.00，只有到 L3 才变 0.00。签名管不住「签名者自己越权声明」，attestation 管不住「声明与运行时不一致」。所以「部署一半」等于没部署——论文自己的说法是 full stack required。

自适应攻击者（白盒、穷举扰动）那组也是同一个形状：L0 1.00、L1 1.00、L2 1.00、L3 0.00，L3 上没有找到任何一个能过关的扰动。

真模型这一路是唯一离开符号沙箱的部分：六个托管模型（Claude Opus 4.8、Sonnet 4.6、Haiku 4.5；Amazon Nova Premier、Nova 2 Lite；GPT-OSS-120B），temperature 0。A1 在 L0 上是 1.00 ISR，六个模型全中；A4 在 L0 上是 0.88 而不是 1.00，论文的解释是八个攻击目标里有一个本来就可以被合法授权，所以那次「成功」不算注入。L3 上六个模型全部 0.00。我没见到逐模型分解表，不宣称每个模型单独都是 0.88。

## 代价那一栏：1350 个 token 从头到尾没动过

防御卡最该盯的是代价，而 Table VII 四档（L0/L1/L2/L3）诚实合约下的数是：任务成功率 1.00、授权类任务成功率 1.00、被误丢工具数 0、平均 token 1350——四档完全一样。

恒定本身就是线索。1350 这个数不是量出来的模型开销，它是 RACG 论文里那个 serialized-context proxy（把选中的工具序列化进上下文的模拟量）的同一个数量级值（RACG 报 ~1,350，对照「全部工具都暴露」的 ~29,875）。签名验证、attestation 查表、运行时 effect diff 这三样都不进这个计数器。论文自己承认了："the signing, attestation, and runtime-verification checks themselves carry latency and engineering overhead in a real deployment, which our symbolic benchmark does not model"。ContractGuard 全文没有任何一个延迟数字——没查到，不是我没找。

对照组给得很清楚：RACG 在七个真实 LLM 上量到的是平均 2,456 token/任务，对照 risk-agnostic causal filtering 的 2,101，约 +17%。也就是说同一批作者在真模型上一测就有 17% 的开销出现，而 ContractGuard 的符号基准里这一栏是 0。离开沙箱之后代价不会是 0，只是没人给出数。

「零过拒」这个数也要看它是怎么来的。ContractGuard 的合约校验是布尔判定：合约要么和 attestation 逐字段相等且签名有效，要么不相等。没有阈值、没有分类器、没有可调的敏感度。诚实合约集在构造上就是「和 attestation 相同的那些合约」，于是它们 100% 通过是定义的推论，不是调参调出来的结果。这和 RACG 那边形成对比：RACG 有一个 λ，λ=2 时 over-block 0.00，λ=0.5 时 over-block 就涨到 0.50、整体成功率掉到 89%——有旋钮的地方就有 tradeoff 曲线，ContractGuard 这一层没有旋钮，所以也没有曲线可看。

真实部署里的过拒率取决于一件论文没测的事：合约会不会因为版本升级、参数扩展、多租户定制而合法地偏离 attestation。一个工具升级了返回字段，L3 的 effect 交集就会把新字段剪掉——这是静默降级，不计入 over-refusal，也不在 Table VII 里。

## 攻击者只有三种措辞、没有反馈，打的是一个必然投降的假 agent

预算写得很窄，那个 0.00 就很难外推。逐条：

- **攻击面**：可以改声明合约的 description / preconditions / effects / risk / authorization，可以制造运行时 effect divergence。不能改门、不能改 agent、不能伪造可信签名、不能改 attestation。
- **搜索空间**：每个目标 256 种合约配置 × 8 个目标 = 2,048 种；乘三种注入措辞（direct / roleplay / obfuscated，论文没给出这三种的原文），每个防御层 6,144 次 phrasing-trial，四层上限 24,576 次。
- **实际执行**：1,898 种配置 / 5,886 次 phrasing-trial。因为 L0–L2 上一旦找到 ISR=1.00 的扰动就短路停搜；只有 L3 因为一个都不成功，才真的把 256 种配置穷举完。
- **单轮、无反馈**：攻击者不观察 agent 的中间输出再调整下一次注入。
- **agent 是假的**："the agent is worst-case adversarially compliant, obeying any injected instruction whose target tool is visible"。

最后这一条决定了整篇的性质。如果 agent 定义为「只要工具可见就服从」，那 ISR 就退化成「攻击者能不能让工具可见」的代理指标。于是「L3 下 ISR=0.00」等价于「L3 下没有伪造合约能同时过签名校验、attestation 比对和运行时 effect 交集」——这是一条关于校验逻辑的会计恒等式，不是关于模型行为的经验发现。论文没有掩饰这一点，它说这个选择 upper-bounds injection success。但读者要知道：这个数字对「模型有多容易被说服」零信息量。

论文还明写了 AgentDojo、InjecAgent、AgentBench 为什么不用——那三个测的是 policy 能不能被诱导，"while holding the tool metadata fixed and trusted"，正好是 ContractGuard 假定被污染的那一层。合理，但代价是这套防御从未在任何一个公开注入评测集上跑过。

还有两个断点。一是根信任：Assumption 1 说得很直白，"If an attacker can forge a trusted signer's signature or replace the attestation itself, ContractGuard degrades to no defense"——整套东西的强度等于签名注册表的强度，而论文不负责后者。二是不可逆副作用：L3 是事后校验，工具先跑再对 effects 取交集，论文自己列举 "an email already sent, a file already deleted, funds already transferred"。从符号 state 里删掉一个 effect 变量，不会把邮件收回来。

第三方复现：截至 2026-08，本地 6,574 篇语料里提到 ContractGuard 的只有这一篇论文本身；公开检索也没找到任何独立复现或绕过报告。两位作者没有机构挂靠，也没有第二组人跑过这个基准。另注意 arXiv:[1911.10472](https://arxiv.org/abs/1911.10472) 也叫 ContractGuard，那是以太坊智能合约入侵检测，和这里无关。

**已核实来源**

- <https://arxiv.org/abs/2606.18550>
- <https://arxiv.org/html/2606.18550v1>
- <https://arxiv.org/abs/2606.13884>
- <https://arxiv.org/html/2606.13884v1>
- <https://arxiv.org/abs/1911.10472>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
