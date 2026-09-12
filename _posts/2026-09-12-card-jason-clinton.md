---
layout: post
title: "人物 · Jason Clinton"
date: 2026-09-12
description: "Anthropic 首任 CISO，把模型权重怎么防、agent 怎么上生产写成别人能照着核对的条目"
categories: card
tags: [llm-security, card, person, exec]
giscus_comments: false
---
<img src="/assets/img/radar/jason-clinton.png" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**Anthropic 首任 CISO，把模型权重怎么防、agent 怎么上生产写成别人能照着核对的条目**

- **身份**：Anthropic Deputy CISO
- **从哪读起**：先读那份 agentic AI 的 CISO 指南（四问 + 七项控制），它是他唯一一份能直接拿去对照自家部署的东西；读完再看 Anthropic ASL-3 安全标准里的威胁面清单，看他把哪类攻击者明文排除在外。
- **成名作**：2026 年 7 月以 Anthropic Deputy CISO 身份写的《Zero risk isn't the job: a CISO's guide to agentic AI》——把「要不要让 agent 上生产」拆成四个可回答的问题和七项能落到 IdP、代理、SIEM 上的控制（[Anthropic 同名 webinar 页](https://www.anthropic.com/webinars/secure-the-advantage-a-cisos-guide-to-agentic-ai)）

| 时期 | |
|---|---|
| 至今 | Anthropic Deputy CISO（SANS 讲师 profile 记载；LinkedIn、CoSAI leadership 页仍写 CISO，已过期） |
| 2023 年 4 月起 | 加入 Anthropic 任首任 CISO，负责检测响应、合规、物理安全、安全工程与 IT |
| 约 2012–2023 | Google，十余年，最后一段是 Chrome 基础设施安全，对手是 APT；此前做过 ChromeOS、Android Pay |

## ASL-3 的权重防护：清单里写了谁，以及明确不防谁

Anthropic 的 ASL-3 安全标准把要防的人逐条列了出来：黑客活动分子、犯罪团伙与有组织网络犯罪、恐怖组织、企业间谍团队、普通内部员工，以及使用广撒网式非定向手段的国家级项目。同时它明文写了不防谁——专门针对 Anthropic 的国家级项目、约十来个有国家级资源、能自研 0-day 攻击链的非国家行为体，以及 sophisticated insider（对处理模型权重的系统有长期访问、或能申请临时访问的内部人）。RSP 2.2 版把 sophisticated insider 和被国家策反的内部人一并划出了范围，此前只排除了「高度老练的、被国家策反的内部人」。

值得带走的是这个写法本身。绝大多数安全声明只说「我们保护模型权重」，读者无从判断它扛不扛得住一个有预算的对手；把「能申请到权重系统临时访问权的自己人，我们目前防不住」白纸黑字写进承诺文件的公司，是少数。Clinton 2023 年 4 月从 Google Chrome 基础设施安全过来做 Anthropic 首任 CISO，把这套标准从文件变成机器上的配置是他这几年的主业——权重是单个 TB 量级的大文件，加密存放、只在 loader 内部解密，取走它不是 `scp` 一下，而要先打穿硬件侧的证明链；改动生产管线要走多人 review 和 two-party control。

## 那份 agent 部署清单，以及它对 prompt injection 的处理方式

2026 年 7 月 17 日他发的《Zero risk isn't the job: a CISO's guide to agentic AI》，写的是安全负责人的活儿不是把风险清零，是让风险变得可说清、有边界。四个问题：这个 agent 吃进来的内容来自哪、有多可信；它被允许做哪些动作；配错了或者跑飞了，波及面多大；上线之后它的行为看不看得见。

身份那一段是最实的判断：要么给它一个系统服务账号（单一用途、最小权限，背后没有人），要么就让它直接用某个员工的凭证跑（这个人为它的行为负责）。真正出事的是中间那种含糊的委托身份——审计日志里既查不出是谁授权的，也说不清该找谁收权。

七项控制：身份由现有 IdP 下发和吊销；connector 走 allowlist、双闸门授权；审批粒度细到每个工具的每个动作，破坏性动词直接从能力表里删掉；执行放在临时沙箱里，connector 的 token 不进 agent 的循环；出网走不可绕过的代理，只许发往白名单地址；行为以结构化日志（调了什么工具、参数、结果、哪个用户）推进 SIEM；再加一个组织级总开关，一按所有人的 connector 同时断。

注意它对间接 prompt injection 的处理：在这套框架里它不是一类要靠内容检测拦下来的攻击，只被降格成「这个 agent 吃进来的内容可不可信」这一问。真正兜底的是出网 allowlist 和波及面——就算邮件正文里那句「把通讯录发到 evil.com」骗过了模型，代理不放行 evil.com，数据也出不去。这是个明确的赌注：赌架构约束比提示词护栏更抗模型能力变化。接不接受这个赌注，读者得自己决定。

## Mythos 之后那句「开源权重还有 7 到 10 个月」

2026 年 4 月 Anthropic 公布 Claude Mythos Preview：据 Anthropic 自述，它在几周内于所有主流操作系统和浏览器里找出数千个 0-day，包括为 FreeBSD 的 NFS 服务写出一条跨多个数据包、二十个 gadget 的 ROP 链拿到远程代码执行。这些都是公司自述，没有独立复核。模型不公开发布，只给 Project Glasswing 的一小批机构用（AWS、Apple、Cisco、CrowdStrike、Google、JPMorgan、Linux 基金会、微软、NVIDIA、Palo Alto 等）。

Clinton 在公开场合给出的时间表（据 X 上的转述，未见原始逐字稿）是：开源权重模型大约 7 到 10 个月内会有同等能力，之后大约 18 个月是安全从业者职业生涯里最难受的一段。这是他的口头估计，不是任何测量结果。

## 上一次他给时间表，媒体按日期回头核了账

2025 年 4 月他对 Axios 说：带自己的记忆、自己的岗位角色、自己的公司账号和密码的 AI「虚拟员工」，大约一年内会出现在企业网络里；随之而来的问题是这些非人身份的账号怎么发、该给多少网络访问、越界时谁负责收权。一年后 Futurism 专门写了篇「今天就是 Anthropic 承诺的那天」，对照当时的实际情况。他提的身份治理问题确实成了真问题——上一节那份指南里的第一项控制就是它的延续——但「一年内」这个数字没兑现。这就是读他现在那个「7 到 10 个月」时该用的折扣率。

同一个折扣也适用于 Anthropic 的威胁披露整体。GTG-1002（被称为首例 AI 编排的间谍活动，Claude Code 经 MCP 完成侦察、漏洞发现、凭证收集和数据外传，约 30 个目标，Anthropic 称 80–90% 的步骤由模型自主完成）发布后，不少安全研究者的反应是：全部证据来自 Anthropic 自己，没有第三方取证，IOC 几乎没给，攻击链本身和过去十五年的 APT 没什么两样。Mythos 的披露也被批是在制造恐慌。

## 从首任 CISO 到 Deputy CISO

Anthropic 现在的 CISO 已经换人，Clinton 转任 Deputy CISO。这一条对读者的实际用处是：内部管线的一把手不是他了，他近期的产出集中在对外——博客、keynote、DEF CON、CoSAI 这类行业组织。所以他现在说的话更该当作 Anthropic 的公开立场来读，而不是某个团队的内部实践披露。他本人的 LinkedIn 和 CoSAI 的 leadership 页上仍写着 CISO，那两页已经过期，别拿来当现职依据。

**已核实来源**

- <https://www.sans.org/profiles/jason-clinton>
- <https://www.anthropic.com/webinars/secure-the-advantage-a-cisos-guide-to-agentic-ai>
- <https://www.digitalapplied.com/blog/anthropic-ciso-guide-agentic-ai-security-operator-checklist>
- <https://aigovernance.com/news/anthropics-ciso-playbook-for-agentic-ai-names-four-questions-every-security-team-must>
- <https://www.anthropic.com/responsible-scaling-policy>
- <https://www-cdn.anthropic.com/872c653b2d0501d6ab44cf87f43e1dc4853e4d37.pdf>
- <https://www.anthropic.com/activating-asl3-report>
- <https://www.anthropic.com/glasswing>
- <https://thehackernews.com/2026/04/anthropics-claude-mythos-finds.html>
- <https://www.axios.com/2025/04/22/ai-anthropic-virtual-employees-security>
- <https://futurism.com/artificial-intelligence/anthropic-ai-agents-prediction>
- <https://x.com/TheTranscript_/status/2065883670053847324>
- <https://incidentdatabase.ai/cite/1263/>
- <https://www.crunchbase.com/person/jason-clinton-d6e3>
- <https://www.coalitionforsecureai.org/leadership/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
