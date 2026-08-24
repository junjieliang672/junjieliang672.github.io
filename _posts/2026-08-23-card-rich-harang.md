---
layout: post
title: "人物 · Rich Harang"
date: 2026-08-23
description: "NVIDIA 安全架构师，主张注入防不住、钱该花在限制模型输出能触发什么后果上"
categories: card
tags: [llm-security, card, person, exec]
giscus_comments: false
---
<img src="/assets/img/radar/rich-harang.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**NVIDIA 安全架构师，主张注入防不住、钱该花在限制模型输出能触发什么后果上**

- **身份**：NVIDIA，个人主页自述 Principal Security Architect（NVIDIA 开发者博客作者页写作 security architect）
- **主页**：[https://rharang.github.io/](https://rharang.github.io/)
- **从哪读起**：先读 2025-10-02 的《Practical LLM Security Advice from the NVIDIA AI Red Team》——它是几十次实际评估里撞见频次最高的三类 bug，比任何框架都更快让你看出他的判断依据；再回头读 2025-09-11 的 AI Kill Chain。

| 时期 | |
|---|---|
| 至今（2026-07 仍在 NVIDIA 技术博客署名发文） | NVIDIA，安全架构师，主页自述 Principal Security Architect；与 NVIDIA AI Red Team 一同发文 |
| NVIDIA 之前 | Duo Security 算法团队 |
| 更早 | Invincea / Sophos，做用 ML 检测恶意软件 |
| 更早 | 美国陆军研究实验室（ARL）；UCSB 统计学 PhD |

## 「假设注入一定成功」：他把预算从检测挪到后果控制

他的论证很短：模型看到的一切——你打的字、RAG 从向量库里捞回来的那段文档、某个工具调用返回的 JSON——进到 context 里是同一种东西，没有任何机制标注哪段可信。所以谁能往这些通道里写字，谁就能控制输出。

由此推出的工程结论不是「训练更强的注入检测器」，而是「模型的输出要当成用户输入来对待」：不许把模型吐出的字符串直接丢进 `exec`/`eval`，改成把回复映射到一张预先定义好的函数白名单（模型只能说「调 `send_email`」，不能自己写代码）；下游系统对模型输出做和对表单输入一样的校验。

他和 guardrail 路线的分歧点要说清楚：他不认为检测无用，而是检测的漏报率没法做承诺，而「代码执行放在沙箱里」「出口网络默认拒绝」这类控制的成本是可估的、失效方式是可预测的。2026-07-30 与 Becca Lynch 合写的 agent 部署建议里这句话最直白：控制必须实施在模型的控制平面之外。

## AI Kill Chain：五个阶段里只有两个值得砸钱

2025-09-11 他发的框架：Recon → Poison → Hijack → Persist → Impact，加一条 iterate/pivot 回路。价值不在阶段名，在他的取舍：Poison 和 Hijack 你堵不住（前面那条论证决定的），所以防御投资应该压在 Persist 和 Impact 上。

Persist 单独成一阶段是这个框架里最有信息量的部分。一次注入如果只影响当前会话，是事故；如果 payload 被写进 RAG 向量库、写进 agent 的跨会话记忆或共享数据库，它就会在别的用户提问时被重新检索出来。对应的建议因此是入库前净化、给用户可见可清的记忆开关、任何写入共享状态的动作要审批。

和 MITRE ATLAS 的分工也清楚：ATLAS 是攻击技术的目录，这个是给应用架构师看的流程图，每个阶段能直接对上一笔防御预算。

## 红队几十次评估里反复撞见的三类 bug

2025-10-02 那篇是八人合署（Joseph Lucas、Becca Lynch 等在内），说的是实测频次，不是理论枚举。「几十个 AI 应用」是 NVIDIA 自陈的数字，没有独立核实。

第一类，对模型输出跑 `exec`/`eval`，直接 RCE。第二类，RAG 的权限模型：入库时没保留 per-user 权限，用户能检索到本不该看的文档；以及写权限过宽——一个有写权限的攻击者可以把触发条件设得极窄，只在有人问某个特定问题时才生效，抽样检查基本发现不了。第三类，模型输出里的 Markdown/HTML 被前端直接渲染，攻击者把 context 里的数据编码进图片 `src`，浏览器一渲染就自动发出请求，数据就出去了。

第三类的修法他们给的是钝招：干脆禁掉 active content 渲染、外链先把完整 URL 展示给用户、用 CSP 限制图片来源。理由是精细净化要枚举所有能发起网络请求的写法，钝招不用。

2026-07-30 那篇的失效模式换了一层，全在基础设施上：agent 没有访问控制谁都能调、代码执行工具能任意命令注入、没有出口网络限制、密钥明文放在 agent 环境里。

## 自治度分级，以及那个有争议的结论

2025-02-25 他和 Martin Sablotny 提的四级：L0 一次请求一次推理；L1 多次推理但顺序预先固定；L2 在固定的决策点上由模型决定要不要调工具；L3 模型自由决定调什么、什么时候改计划。

两人的判断是：风险主要在 agent 能碰到的工具和权限里，自治度本身不等于风险，它增加的是不可预测性、从而让威胁建模变难。他们的原话是，在没有能执行敏感或物理动作的工具时，操纵 AI 组件的主要风险就是输出错误信息，跟工作流多复杂无关。按这个逻辑，一条 L1 的固定流水线只要接了能删库的工具，就比一个只能读文档的 L3 危险。这是他们两人的立场，不是行业共识——不少合规框架仍在按自治度分档设限。

## 供应链那一侧

他也做和 prompt 完全无关的一类问题。2025-07-28 NVIDIA 在 NGC 上线模型签名，用的是 OpenSSF 的 OMS 格式：不改动权重文件本身，而是另出一个 detached bundle，里面装全部文件的哈希清单和一个覆盖整个 bundle 的签名，支持企业 PKI、自签证书和 Sigstore 无密钥签名三种密钥管理。它解决的是「你下载的这份权重是不是发布者发的那份」——pickle 反序列化 RCE、Hub 上的仿冒仓库这类事。他个人主页自述参与发起了这个跨厂商工作组；OMS 规范本身的致谢里他与 Google、Red Hat 等多方并列，不是单独主导。

还有一条会让人不舒服的立场：2025-09-26 他和 Joseph Lucas、Erick Galinkin 主张不该给模型权重发 CVE。理由是被提名的绝大多数「模型漏洞」实际上是框架/应用代码的洞、供应链问题，或者只是模型的统计行为；类比是 blind SQL injection 的洞在应用里，不在数据库里。唯一的例外是训练数据被投毒、在特定权重文件里留下可复现后门的情形。

**已核实来源**

- <https://rharang.github.io/>
- <https://developer.nvidia.com/blog/author/rharang>
- <https://developer.nvidia.com/blog/modeling-attacks-on-ai-powered-apps-with-the-ai-kill-chain-framework/>
- <https://developer.nvidia.com/blog/practical-llm-security-advice-from-the-nvidia-ai-red-team/>
- <https://developer.nvidia.com/blog/agentic-autonomy-levels-and-security/>
- <https://developer.nvidia.com/blog/four-ways-to-deploy-more-secure-ai-agents/>
- <https://developer.nvidia.com/blog/bringing-verifiable-trust-to-ai-models-model-signing-in-ngc/>
- <https://developer.nvidia.com/blog/why-cves-belong-in-frameworks-and-apps-not-ai-models/>
- <https://genai.owasp.org/contributors/2/>
- <https://rharang.github.io/photo.jpg>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
