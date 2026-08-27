---
layout: post
title: "人物 · Christina Liaghati"
date: 2026-08-27
description: "把散落在论文和新闻里的 AI 攻击编成带编号的清单，让安全团队和采购方能照着打勾"
categories: card
tags: [llm-security, card, person, policy]
giscus_comments: false
---
**把散落在论文和新闻里的 AI 攻击编成带编号的清单，让安全团队和采购方能照着打勾**

- **身份**：MITRE，ATLAS 负责人（至少到 2025 年 9 月仍在任）
- **主页**：[https://atlas.mitre.org/](https://atlas.mitre.org/)
- **从哪读起**：直接翻 atlas.mitre.org 的 Techniques 矩阵，挑 AML.T0086 那条点进去看它挂的 case study——十分钟就能看出这套编号法把什么算作「一条技术」。
- **成名作**：共同发起并长期主导 [MITRE ATLAS](https://atlas.mitre.org/)——把针对 AI 系统的攻击按 ATT&CK 的战术/技术格式逐条编号（如 AML.T0086「通过 AI agent 调用工具外泄数据」），让 AI 攻击第一次能被写进企业威胁建模表和政府采购文件。

| 时期 | |
|---|---|
| 2021–今 | MITRE，ATLAS 负责人；据 MITRE 2025 年官方稿，同时管理一个 50 余人的部门 |
| — | George Washington University 系统工程博士 |

## 把 ATT&CK 的格式套到 ML 上：两件好事和一个别扭

ATLAS 的前身是 2020 年 MITRE 与 Microsoft 等机构合作的 Adversarial ML Threat Matrix。关键决定是格式：不新造一套语言，直接沿用安全团队已经在用的 ATT&CK 两级结构——上层是「战术」（攻击者此刻想达成什么：侦察、初始访问、持久化、外泄），下层是「技术」（具体怎么做，每条一个编号）。

好处很实在。一是零迁移成本：红队报告里本来就写着「T1566 钓鱼」，现在多写一行「AML.T0051 LLM Prompt Injection」，威胁建模的表格一格都不用改。二是每条技术底下挂真实 case study——「有人往网页里塞了段人眼看不见的文字，Bing Chat 读了以后照办」这种事，一旦有了编号和案例页，采购方就能在合同附件里要求供应商说明「你们对 AML.T0051 做了什么」。抽象的「AI 有风险」做不到这件事。

别扭的地方要说清楚（这是我的判断，不是她的表述）。ATT&CK 隐含的假设是攻击者沿一条链推进：先钻进来、再横向移动、再往外搬东西，每一步留下不同的日志。很多 ML 攻击不长这样。对抗样本是一次性的，模型窃取是用查询预算定义的——攻击者花五万次 API 调用把你的模型蒸出来，中间没有任何「横向移动」可言。这类攻击硬塞进 kill chain，「战术」那一层就只剩个标签作用，防御方没法据此决定在哪儿埋检测点。ATLAS 换来的兼容性是真的，代价也是真的。

## 2025 年的 agent 补丁：条目从哪来，谁决定收不收

2025 年 10 月，ATLAS 一次性并入 14 条与 AI agent 相关的技术和子技术，来源是安全公司 Zenity Labs 的研究和它开源的攻击矩阵。新条目管的不再是模型本身，是 agent 的执行层：AI Agent Context Poisoning 及其 Memory / Thread 两个子项（前者是往 agent 的长期记忆里写一句话，让它在往后每一轮对话里都生效；后者只污染当前这一个会话）、Modify AI Agent Configuration、RAG Credential Harvesting、Exfiltration via AI Agent Tool Invocation（AML.T0086，让 agent 自己调用它有权限的工具把数据发出去）。

她的角色是策展人，不是发现者：条目由外部研究者提交，她的团队决定收不收、挂在哪个战术下、和已有条目怎么去重。真正难判的是口径——「往 agent 记忆里写指令」和「往当前会话里写指令」到底算两条技术还是一条技术的两个子项？ATLAS 这次选了后者（一条父技术带两个子项）。这个决定会一路传导到所有引用 ATLAS 的检测产品和合规清单上。

## 最难的那块：没人愿意公开自己被 AI 攻击了

ATLAS 的 case study 长期只能来自公开研究和媒体报道，生产环境里真实发生的事故几乎拿不到——被 prompt injection 骗走了客户数据的公司不会发博客。2024 年 10 月，她参与推动的 AI Incident Sharing 通过 MITRE 的 Center for Threat-Informed Defense 启动，走的是 ISAC 的路子而不是公开 CVE 的路子：企业提交经过匿名化处理的事故信息，作为交换获得接收其他成员数据的资格，圈子封闭、内容不公开。

这个设计有个绕不开的张力：匿名化到什么程度企业才肯报，和报上来的东西去掉厂商、产品、时间线之后还剩多少防御价值，是同一个旋钮的两端。启动时公布的合作方有十五家以上；到目前为止公开可核实的产出有多少，我没能查到，不给数字。

## 和 OWASP、NIST 的分工：三份清单为什么没合并

读者实际会问「我该用哪个」。ATLAS 面向红队和威胁建模，覆盖训练数据、模型供应链一直到部署，粒度是单条攻击技术；OWASP 的 LLM Top 10 面向写代码的人，管的是部署后的集成层（输出没转义、插件权限太大这类）；NIST 的对抗性 ML taxonomy 是学术分类，读完知道攻击有几大类，但不能拿去做 checklist。社区做了不少 OWASP↔ATLAS 的对照表，但没人合并，因为三份清单的读者根本不是同一批人。

她的影响力也不来自论文——2025 年 9 月她在 NIST 的场合做的是 ATLAS 概览报告，把这套编号法讲给政府和国防采购这边听；同年入选 Federal 100。ATLAS 被写进 2024 年 12 月众议院两党 AI 工作组报告，走的也是这条线。

**已核实来源**

- <https://csrc.nist.gov/csrc/media/Presentations/2025/mitre-atlas/TuePM2.1-MITRE%20ATLAS%20Overview%20Sept%202025.pdf>
- <https://csrc.nist.gov/Presentations/2025/mitre-atlas>
- <https://www.mitre.org/news-insights/news-release/mitres-christina-liaghati-named-2025-fed100-honoree>
- <https://engineering.gwu.edu/alumna-christina-liaghati-named-2025-fed100-honoree>
- <https://atlas.mitre.org/>
- <https://labs.zenity.io/p/techniques-from-zenitys-genai-attacks-matrix-incorporated-into-mitre-atlas-to-track-emerging-ai-thr>
- <https://zenity.io/blog/current-events/zenity-labs-and-mitre-atlas-collaborate-to-advances-ai-agent-security-with-the-first-release-of>
- <https://www.mitre.org/news-insights/news-release/mitre-launches-ai-incident-sharing-initiative>
- <https://ctid.mitre.org/projects/secure-ai/>
- <https://www.darkreading.com/threat-intelligence/mitre-launches-ai-incident-sharing-initiative>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
