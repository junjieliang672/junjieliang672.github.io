---
layout: post
title: "系统/工具 · bounded-agents（Agentic Principal Chain, APC 参考实现）"
date: 2026-08-23
description: "把「这一步能不能做」从模型挪到会话级授权信封：六道合取检查加动作组合封闭，作者自己发的参考实现"
categories: brief
tags: [llm-security, brief, tool]
giscus_comments: false
---
**把「这一步能不能做」从模型挪到会话级授权信封：六道合取检查加动作组合封闭，作者自己发的参考实现**

- **主页**：[https://github.com/xmuruaga/bounded-agents](https://github.com/xmuruaga/bounded-agents)
- **从哪读起**：先跑 `python scripts/demo.py`——它按 C1–C6 六个条件各走一遍，能在几秒内看清「哪条动作被哪一条拒了」，比先读论文的形式化部分快得多。
- **成名作**：论文 [Bounded Agents: Delegation Security for Multi-Agent AI Systems](https://arxiv.org/abs/2608.15888)（arXiv [2608.15888](https://arxiv.org/abs/2608.15888)）：在 AgentDojo 的 compromised-model 设定下把外泄从 75–100% 压到 0%，并拦下全部 544 个 InjecAgent 数据窃取样例。

## 一次 tool call 要过六道闸，闸门看的是信封不是 prompt

APC 的判断对象不是模型输出的那段文本，而是一个**授权信封**（Authorization Envelope）：会话开始时生成，里面装着 scope（这个 agent 能碰哪些动作类、哪些资源）、principal chain（用户 → 主 agent → 子 agent 的一串可验证身份）、委托预算（还能花几次调用）、以及一句意图声明（「帮我订下周去里斯本的机票」）。关键约束是**单调收窄**：主 agent 把活派给子 agent 时，只能把信封裁小，不能加权限——论文把这条叫 Blast Radius Monotonicity 并给了证明（不过明说「the theorems are not machine-checked; mechanized proofs are future work」）。

零件按输入输出拆：

- **PDP（Policy Decision Point）**：输入 =「待执行的一次动作请求 + 本会话已累积的授权状态」，输出 = allow/deny 加上被哪一条拒。六条是合取的，README 强调 "no scoring, no overrides"：C1 身份绑定、C2 scope + 组合、C3 上下文绑定（会话与策略版本对不对）、C4 审批绑定（高影响动作要一个哈希绑定的批准 token）、C5 证据承诺（写不进审计就 fail-closed 拒掉）、C6 意图绑定（这个动作和声明的任务相关吗）。
- **PEP（Policy Enforcement Point）**：坐在 agent runtime 和真正的后端之间，充当 tool/MCP 网关，放行之前先把证据写下去。
- **Evidence Store**：SHA-256 哈希链，每条带前一条的哈希，事后能看出有没有被改。
- **组合封闭（composition closure）**：C2 里真正新的那半。它读的是「本会话之前已经做过什么」。论文自己的例子逐字是："An agent authorized to read confidential documents and to send external email can combine both to exfiltrate data—without violating any individual permission." 读机密单独合法，发外部邮件单独合法，两个连起来在同一会话里就被拒。论文还处理了绕道版本——先把机密写进一个中间文档再发出去，pairwise 规则漏掉，需要 k-tuple 限制才拦得住。

效果上，意图绑定（C6）那一格给的数字是：destruction 类攻击成功率 38.6% → 4.0%，manipulation 90.5% → 12.1%。摘要没写这两个数分别对应哪个数据集的哪个子集。

## 四条离线评测不用 API key，AgentDojo 那半要 AWS Bedrock

到第一个结果：

```
git clone https://github.com/xmuruaga/bounded-agents.git
cd bounded-agents
pip install -e ".[dev]"
pytest                    # 215 tests
python scripts/demo.py    # walkthrough
python scripts/verify_numbers.py
```

仓库自报 215 个测试、核心库 90% statement coverage、`apc/` 约 2,500 行且零运行时依赖——这三个数是 README 自述，我没独立跑过。

3,154 个评测实例分成两半，这个分界比总数重要：**离线/确定性 1,554 条**（InjecAgent 1,054 + ASB 400 + 委托链 99），**带模型跑的 1,600 条**（AgentDojo：949 个 utility 任务对 + 609 个 compromised-model + 42 个自适应变体）。后一半用 Claude Sonnet 4.6 跑效用、Claude Haiku 4.5 跑 compromised-model，都经 AWS Bedrock——没有凭证这部分复现不了。

 compromised-model 这个设定要说清楚：论文的原话是把 ground-truth 的攻击调用**插在第一个正常 tool call 之后**，「tests APC independently of model behavior」。也就是说 0% 这个数字衡量的是「给定攻击动作已经出现在轨迹里，PDP 会不会放行」，而不是「模型在真实对抗提示下会不会被骗」。这两件事的难度不在一个量级上，引用 0% 时得带上这个限定。（我推断离线那 1,554 条也是把动作轨迹回放给 PDP 而非驱动模型，因为它们不需要任何 API key；README 没有逐字这么说。）

## 引擎判一次 0.049 毫秒，难的是那张禁令表谁来写

宣传语里最亮的两个数——0% 外泄、亚毫秒延迟——恰好是这套东西里最容易的部分。延迟测出来 p50 = 0.049 ms、p99 = 0.240 ms（五次重复中 p99 最高到 0.261 ms），硬件是 Intel Core i5-1245U / 16GB / Windows 11 的 idle host。这是一台笔记本上的单进程查表开销，不含网络 I/O、不含 evidence 落盘的真实延迟，别当成端到端开销读。

真难点 README 自己写在了显眼处：**"composition restrictions are authored, not inferred"**。哪两类（或 k 类）动作凑在一起算外泄，是人手写的表。这意味着接进你自己的 tool 集合时，你得先给所有工具做一套动作分类法（`read_confidential` / `send_external` / `write_shared` 这种粒度），再枚举禁止的组合。漏一条就是一个通道，而且是静默的通道——PDP 会理直气壮地 allow。论文的 Composition Soundness 定理本身也带着这个前提：它只在「restriction set 完整 + 串行准入」下成立。

另一边是表写紧了业务就崩。interactive 配置下的效用保持率按域差得很开：Workspace 86.7%、Travel 60.0%、Slack 60.0%、Banking 58.8%；换成百分点差是 Workspace −9.6 pp、Banking −10.0 pp、Slack −13.0 pp，而 Travel 是 +0.2 pp——基本没掉。摘要给的总体数是两个设定下分别 −8.6 和 −13.9 个百分点。Travel 几乎不掉是因为订机票这类任务里很少出现「读了敏感东西再往外发」的合法组合；Slack 掉 13 个点是因为它天然就在做跨边界转发。你要付多少效用代价，取决于你的业务本来有多少合法动作长得像外泄，这件事没法从论文的数抄过去。

## 它明说不管的四件事

① **拆会话（session splitting）**是设计上放行的。论文的原话：组合状态是 per-session 的，「cross-session tracking requires durable lineage state」，落在被信任的 T3 边界外。攻击者在会话 A 里读机密、会话 B 里发邮件，两次各自都干净。

② **授权范围内的单次滥用**不在射程——该 agent 本来就能发邮件，只是发给了错的人。论文给了这一格的残余成功率：**60.4%**。这是全篇最诚实的一个数，也是最该被引用的一个。

③ **参数级攻击**不管：动作类型合法、参数被改（转账收款人被换掉），APC 看的是动作类不是参数值，语义校验明确列为 out of scope。

④ **信任集中到网关**：T1–T3 假设 PDP/PEP 不可被攻破、签名不可伪造、对手不能跨会话行动。网关塌了整套失效，而 agent runtime 恰恰是最容易被 supply chain 摸到的那一层。

再加三条：23 个自适应攻击是作者自研自测（覆盖六个条件、两个定理和 A1–A4），没有外部红队复现；定理没机器验证；README 自称 reference implementation，无持久化、无分布式策略存储。

什么时候不该用它：单 agent 无委托链时，六条里有四条退化成常规 RBAC，收益只剩审计链；动作空间小到组合不出坏结果时（比如只读检索），你付的是效用代价换不到东西；需要生产级策略分发、多实例共享会话状态时，这份实现没有那一层。以及——如果你的威胁模型里对手能开两个会话，第一节讲的组合封闭对他不成立。

**已核实来源**

- <https://github.com/xmuruaga/bounded-agents>
- <https://arxiv.org/abs/2608.15888>
- <https://arxiv.org/html/2608.15888v1>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
