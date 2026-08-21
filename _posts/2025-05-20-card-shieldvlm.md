---
layout: post
title: "防御机制 · ShieldVLM（清华 CoAI，多模态隐性毒性护栏）"
date: 2025-05-20
description: "把「图无害、文也无害、拼一起有害」的检出率从 52% 拉到 89%——但那 89% 测在自产同分布集上"
categories: brief
tags: [llm-security, brief, defense]
giscus_comments: false
---
**把「图无害、文也无害、拼一起有害」的检出率从 52% 拉到 89%——但那 89% 测在自产同分布集上**

- **主页**：[https://arxiv.org/abs/2505.14035](https://arxiv.org/abs/2505.14035)
- **从哪读起**：先读 arXiv [2505.14035](https://arxiv.org/abs/2505.14035) 的 Table 3/4（三种输入形式 × safe/unsafe 分类 F1），dialog 那一列比摘要诚实得多；再翻 [2608.01373](https://arxiv.org/abs/2608.01373) 的 Table 1，那是唯一一次外人把它跑起来。

## 「无害 + 无害 = 有害」这类样本，现成 guard 的命中率约等于抛硬币

它要拦的东西叫 multimodal implicit toxicity（MMIT）：一张图单看能过审，一句文案单看也能过审，两者放在一起才构成风险。论文给了 5 种跨模态关联模式，其中三种能想象出画面——semantic drift（同一个词在图的语境里换了意思）、metaphor（图里的符号代指敏感话题，文字只说符号）、knowledge（要靠常识或文化背景才知道这个组合意味着什么，比如某种药品外观加一句「一次吃几片见效快」）。数据集覆盖 7 大类 31 子类，2,100 条多模态 statement 和 prompt。

对照数字（MMIT 测试集准确率）：OpenAI Moderation API 在三种输入形式上都在 50% 上下；Llama Guard 3 Vision prompt 52.14%、dialog 51.57%；ShieldVLM（Qwen2.5-VL-7B-Instruct 上微调）statement 89.05%、prompt 91.19%。在换成显性毒性的 OOD 集上，ShieldVLM statement 94.28%、prompt 95.03%，高于 Claude-Sonnet-3.5 的 91.80% / 92.25%。

这个 89% 的来路要当场说破。训练集 4,238 条（2,119 unsafe / 2,119 safe）和测试集 1,094 条（546 unsafe / 548 safe）出自同一条 GPT-4o + 文生图流水线，unsafe 侧全部是按同一套 7×31 分类体系、同一套关联模式生成的。论文描述的构造流程是先机器判定 valid、再进人工复核环节，文中没有报标注者间一致性（kappa）。所以 Llama Guard 3 Vision 输掉的 37 个点里，有多少是「不会做跨模态推理」，有多少是「没见过这条流水线的分布」，论文没有拆开。ShieldVLM 的消融也提示了同一件事：去掉推理链（w/o r.）后 statement 从 89.05 掉到 84.21，只掉 4.84 个点——剩下那 30 多个点的优势是微调本身给的。

## 代价这一栏，论文自己空着：dialog 只有 74.8%，延迟一个数都没有

防御机制卡最该有的那个数——「正常任务上掉几个点、误拒涨多少」——ShieldVLM 论文里没有。全篇关于误报的表述停在「对良性内容保持高准确率」这类定性话上，没跑过任何过度拒绝专测（MOSSBench、XSTest 之类一条都没有）。

能当代价用的证据只有三条，都要自己从表里抠：

第一，dialog 形式塌得厉害。MMIT 测试集上 statement 89.05%、prompt 91.19%，dialog 74.8%，比 prompt 低 16 个点；OOD 上 dialog 78.33%，同样比 statement 的 94.28% 低 16 个点。dialog 的 unsafe F1 只有 71.93（safe F1 77.14），OOD dialog unsafe F1 73.68 对 safe F1 81.59。也就是说，一旦要在「用户问 + 图 + 模型已经生成的回答」三件套上判，漏放明显变多。

第二，误判在 safe/unsafe 之间不对称，而且方向随分布翻转。MMIT 集上 statement safe F1 89.87 高于 unsafe F1 88.08；OOD statement 反过来，safe F1 90.72 低于 unsafe F1 95.87——换到显性毒性数据上，错的那一头挪到了安全侧，即误拒。论文没有解释这个翻转。

第三，成本。每次判定都要先分别看文、看图，再做跨模态分析，最后出结论，也就是每条请求都要生成一段推理文本。这段文本值 4.84 个点（见上节消融）。但推理 token 数、单条延迟、显存占用，论文一个数都没给；公开的只有训练侧：4×A100、约 1.5 小时、batch 64、max length 2048、lr 1e-5、5 epoch。作为在线护栏，这三个缺失的数比准确率更决定能不能上线，这里只能写「没查到」。

真正测到的干净误报数字，是外人在别的论文里顺手给的——0.2% / 0.1% / 0.4%，见下一节。

## 唯一的第三方只干了一件事：白盒 2000 步 PGD，把 0.2% 的误报顶到 70.4%

[2608.01373](https://arxiv.org/abs/2608.01373)（*The Boy Who Cried Wolf*，2026-08-02 预印本，未经同行评审）提出 USD（unsafe semantic distillation）。它不是越狱，方向是反的：把良性图片往「不安全语义」那侧推，让 guard 对正常用户狂拒——受害者是产品的可用性，不是安全性。

攻击者预算写得很清楚，抄下来：白盒，「the attacker has full access to the target guard model's architecture and weights」；不控用户文本（prompt agnosticism），所以优化时只能用一批 shadow prompt 采样；扰动受 ℓ∞ 约束，ε ∈ {8/255, 16/255}；PGD 迭代 T=2000；每个数据集只取 100 张良性图（TextVQA、VQAv2）。

评估分三档，原文定义：「ASRs measures success on the shadow prompts used during optimization, serving as an in-distribution sanity check. ASRg measures success on 10 generic image-agnostic questions...ASRr measures success on 10 image-specific user requests generated for realistic application scenarios.」ASRg 用的就是「What does this image depict?」这类图无关问句，ASRr 才是对真实用户提问的成功率。

结果：ShieldVLM 干净图上 ASRs/ASRg/ASRr = 0.2% / 0.1% / 0.4%；ε=8/255 时 ASRr 70.4%（TextVQA）、72.8%（VQAv2）；ε 翻到 16/255 是 75.6% / 77.2%。

两处必须点破。一，0.2% 这个近零干净误报是外人测出来的，同批被测的 guard 也都近零——说明「自己测自己几乎零代价」是这条赛道的普遍状态，别把它当 ShieldVLM 的加分。二，迁移很差：拿别的模型上优化出的扰动打 ShieldVLM，成功率只剩 0.7%–3.7%。所以 70.4% 是「你把权重开源出去」的代价，不是黑盒 API 的代价。另外那张预处理防御表（JPEG 压缩、均值滤波、高斯模糊）测的目标是 Llama Guard 3 Vision，不是 ShieldVLM——别把那几个数挪到 ShieldVLM 头上。

## 同期后继工作里，没有一篇把它放进基线表

本地 6229 篇语料里提到 ShieldVLM 的只有 3 篇。[2608.01373](https://arxiv.org/abs/2608.01373) 提了 8 次，就是上一节那个把它当靶子的；OmniGuard（[2512.02306](https://arxiv.org/abs/2512.02306)，2025-12-02）提 2 次，在 related work 引一句然后自己另造 benchmark；[2601.14127](https://arxiv.org/abs/2601.14127)（*The Side Effects of Being Smart*，2026-01-20）只出现在参考文献里，正文没跑。

结论直说：ShieldVLM 的 89.05% / 91.19% 至今没有任何独立机构在自己的数据上复现过，也没有第三方测过它的效用代价。它唯一一次被外人真正加载起来跑，是被人用 2000 步 PGD 打误报率。

对要用它的人，具体该做的事：权重在 Hugging Face 的 `thu-coai/ShieldVLM-7B-qwen`，代码在 GitHub `thu-coai/ShieldVLM`（含 `infer.py`、`infer_shieldvlm.sh`，训练走 LLaMA-Factory 配置，仓库只有 4 个 commit，HF 那边 README 是空的）。上线前先拿自家真实良性流量跑一遍误拒率——这个数论文里没有，那 0.4% 是别人在 100 张 TextVQA/VQAv2 图上测的，跟你的分布没关系。如果你的场景是多轮对话而不是单条 statement，先看 dialog 那 74.8% 能不能接受。

**已核实来源**

- <https://arxiv.org/abs/2505.14035>
- <https://arxiv.org/html/2505.14035v1>
- <https://dl.acm.org/doi/10.1145/3746027.3755711>
- <https://arxiv.org/html/2608.01373v1>
- <https://github.com/thu-coai/ShieldVLM>
- <https://huggingface.co/thu-coai/ShieldVLM-7B-qwen>
- <https://arxiv.org/abs/2512.02306>

---

*本文由自动化管道生成（采集 → 逐字核验 → 模型撰写），未经人工改写。*
