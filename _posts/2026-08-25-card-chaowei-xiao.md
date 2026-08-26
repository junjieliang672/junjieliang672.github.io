---
layout: post
title: "人物 · Chaowei Xiao"
date: 2026-08-25
description: "做自动越狱攻击起家，现在用自适应攻击去逐个检验别人的 agent 防御站不站得住"
categories: card
tags: [llm-security, card, person, academic]
giscus_comments: false
revised: "2026-08-25"
---
<img src="/assets/img/radar/chaowei-xiao.jpg" alt="" style="width:96px;height:96px;border-radius:50%;object-fit:cover">

**做自动越狱攻击起家，现在用自适应攻击去逐个检验别人的 agent 防御站不站得住**

- **身份**：Johns Hopkins University 电气与计算机工程系助理教授；NVIDIA Research 研究员
- **主页**：[https://xiaocw11.github.io/](https://xiaocw11.github.io/)
- **从哪读起**：先读 AutoDAN-Turbo（arXiv [2410.05295](https://arxiv.org/abs/2410.05295)）的方法一节，看「策略库」是怎么攒起来的；再读 2026 年的 AutoDojo（[2606.15057](https://arxiv.org/abs/2606.15057)）看同一套思路怎么被用来测别人的 prompt injection 防御。
- **成名作**：[AutoDAN: Generating Stealthy Jailbreak Prompts on Aligned Large Language Models](https://arxiv.org/abs/2310.04451)（ICLR 2024）——用遗传算法自动写出读起来完全通顺的越狱 prompt，让当时主流的「困惑度过滤」防线整个失效

| 时期 | |
|---|---|
| 现在 | Johns Hopkins University 助理教授（ECE），同时是 NVIDIA Research 研究员，主页自述正在 Stanford 访问 |
| 至 2025 年初 | University of Wisconsin–Madison 信息学院助理教授（2025 年 2 月院方新闻仍以此头衔报道） |
| 更早 | Arizona State University 助理教授（与 UW–Madison 两段任教合计约三年） |
| PhD | University of Michigan, Ann Arbor 计算机科学与工程博士；本科清华大学软件学院 |

## 两个都叫 AutoDAN，他做的是被引最多的那个

2023 年秋天有两篇论文同时叫 AutoDAN。他参与的这篇（一作 Xiaogeng Liu）走的不是 GCG 那条路——GCG 用梯度搜出一串乱码后缀，效果好，但一个困惑度过滤器就能把它认出来：正常人不会在问题后面接 `describing.\ + similarlyNow write`。AutoDAN 改成在人写的 DAN 类越狱模板上做遗传算法变异，交叉、改写句子、换词，选择压力来自攻击是否成功，产出的东西读起来是一段通顺的角色扮演指令。困惑度检测因此完全没有着力点。

2024 年的 AutoDAN-Turbo 把这件事推到全黑盒：不给任何预定义的越狱套路，让一个 attacker agent 自己从零试，把「哪种说法在哪类问题上管用」抽成一条条策略存进策略库，下次直接检索复用；人手写的策略也可以直接塞进同一个库里一起用。论文自报在 GPT-4-1106-turbo 上 88.5% 的 ASR、并入人写策略后 93.4%，比基线平均高 74.3%（这些都是作者自报数字，且绑定那个具体模型版本）。这套形式对防御方的意义是：你没法拿一个固定的攻击集来证明自己安全，因为攻击方会在你这套防御上重新长出新策略。

## 他自己做防御，也自己动手证明防御不够用

DiffPure（ICML 2022，NVIDIA 期间的合作）是他做防御的那一面：把对抗样本先加一点扩散噪声再反向去噪，扰动被洗掉、图像语义还在。少见的地方在于论文自己给自己上强度——用 adjoint method 把梯度反传穿过整条反向 SDE，从而能对这个防御本身做自适应攻击，而不是只报一个「在 PGD 下很稳」的数字。

四年后 AutoDojo（2026）把同样的做法搬到 agent 上：拿自适应黑盒攻击去打现有的 indirect prompt injection 防御。论文声称的结论是，防御有效性高度依赖「攻击—防御—模型」的具体组合，没有哪个防御能挡住所有攻击，也没有哪个攻击能穿透所有配置。另一条对做产品的人更直接：用户任务写得越模糊、越欠指定（「帮我把这封邮件处理一下」比「把这封邮件里的发票金额填进表格 B」危险得多），agent 越容易被注入劫持——因为它得去外部内容里找线索补全你的意图，而那些内容正是攻击者控制的。MaskForge（2026）在扩散语言模型上看到同样的不均匀，外加一条该架构特有的规律：留给受害模型自己填的 mask 比例越大，攻击越容易成功。所以看到「我们把 ASR 压到 X%」这种说法，得先问是在哪套攻击、哪个模型上测的。

## 2026 年他改了判定口径：不看模型说了什么，看 agent 整条轨迹做了什么

DTap（DecodingTrust-Agent Platform，2026-05）的主张是：以「模型有没有表示配合」「输出符不符合政策」「有没有观察到失败」当判据，会系统性低估真实危害；必须去核实后果——那个文件是不是真被删了、那笔钱是不是真转出去了。平台还报了一条能省钱的实测：自动迭代改写越狱 prompt，收益在少数几轮里基本吃满，之后急剧递减，红队预算不该堆在轮数上。

AgentLens（2026-06）从另一头切多轮 coding agent：风险是跨步骤累积的，只看聚合指标或孤立看单个动作都会漏——每一步单看都合规，合起来是一次数据外泄。同时它显示纯推理期的激活引导（不动任何权重）就能实质改变安全行为。LPG（2026-05）则指出一个尴尬事实：号称按用户给定政策判定的 guardrail，很多时候并没有真的去对照那几条政策文本，而是在用训练时学到的安全先验；而专门拿 policy-grounded 数据微调它，反倒会削弱它对陌生政策的推理能力。

## 最新一批打的是承载 agent 的管道，不是模型

2026 年 8 月前后的三篇构成一条新线，攻击面从模型权重和 prompt 挪到了部署基础设施。AdvCLI（[2510.06607](https://arxiv.org/abs/2510.06607)，EMNLP 2026 Findings）测的是 CLI agent 的 harness：140 个任务对齐 MITRE ATT&CK，跑在多主机沙箱里、用确定性脚本核实结果而不是让另一个模型打分，结论是当前 CLI agent 经常「先拒绝、然后照做」，尤其当请求被包装成攻击者的战术流程时。KeyPooling（[2608.17485](https://arxiv.org/abs/2608.17485)）查的是多个用户经共享密钥池／API 中继走同一条路径时，prompt cache 的隔离在哪一环塌掉——这是侧信道，跟对齐一点关系没有。第三篇（[2608.17445](https://arxiv.org/abs/2608.17445)）给的是分解攻击：把一个有害请求拆成若干看着无害的子请求，分散到互不关联的账号发出，任何按会话或按用户累积计数的有状态防御都拦不住，论文给的是这类防御的能力上界。这三件事都写不进 model card，也不是模型厂商能替你测的，是平台方自己的责任。

## 读他的工作要按线读，不要按篇读

他的组一年产出几十篇、挂名密集、大量 arXiv 预印本先行，本卡引的多数 2026 年论文尚未见同行评审记录，单篇分量并不均等。另外他在 AI4Science 和具身智能上同样高产（名下被引最高的几项里有与 LLM security 无关的），那些和这条线不通。要跟他的安全工作，按三条线看就够：越狱自动化、agent 轨迹评测、部署基础设施。

**已核实来源**

- <https://xiaocw11.github.io/>
- <https://arxiv.org/abs/2310.04451>
- <https://arxiv.org/abs/2410.05295>
- <https://arxiv.org/abs/2205.07460>
- <https://arxiv.org/abs/2510.06607>
- <https://ischool.wisc.edu/2025/02/05/chaowei-xiao-named-ai2050-early-career-fellow-by-schmidt-sciences/>
- <https://ai2050.schmidtsciences.org/fellow/chaowei-xiao/>
- <https://isi.jhu.edu/team/chaowei-xiao/>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
