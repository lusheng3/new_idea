# 语义先验引导的联邦稳定因果特征发现 Idea Report
> 生成时间：2026-11-24 | 状态：PENDING_REVIEW

---

## Part 1 Topic Overview

### 1 Motivation

联邦因果特征选择（如 FedCIFL）在保护隐私的前提下，从分散客户端恢复对目标标签有因果效应的特征集合，支撑下游的 OOD 泛化。然而，当客户端样本量长尾、Non-IID 且存在低功效客户端时，**纯数据驱动的条件独立检验/效应估计在小样本客户端上失去功效**，导致一类关键特征被系统性误删：那些效应较弱、但在所有客户端都真实存在的「弱稳定因果特征」。

本项目前期在 FedCIFL 官方代码上的机制验证已确证这一现象：弱稳定特征 X4（真实效应 0.45，全局存在）在 8 个随机种子下被稳定门控反复判为 `Client-Local` 而删除，稳定特征召回率长期卡在 0.875；而一旦放宽阈值，局部伪特征 X9/X10（只在单客户端成立）又会被一并放进来。**数据本身无法区分「弱但全局」与「强但局部」，这正是纯数据驱动方法在小样本联邦场景的根本瓶颈。**

与此同时，变量名本身携带语义信息（"吸烟年数"与"肺结节"的方向，人一眼可知），这部分信息在现有联邦因果方法中完全未被利用。集中式因果发现已证明 LLM 语义先验有效，但「联邦 + LLM 语义先验」这一交叉点尚无工作落地（2026 综述明确列为开放问题）。

**本研究的必要性：**

- **应用必要性**：医疗、金融等多机构隐私敏感数据中，小样本客户端普遍存在，弱稳定因果特征对下游 OOD 泛化至关重要 [1][2]
- **理论必要性**：纯数据驱动的 CI 检验在小样本下功效不足（type-II 错误），弱全局效应无法与局部伪效应在数据上区分 [3]；语义先验是打破这一平局的唯一外部信息源
- **时机必要性**：LLM 语义先验在集中式因果发现已证明有效，但联邦场景无人做；2026 联邦因果综述明确把「LLM 先验接入联邦结构学习」列为 open problem [4]

### 2 Research Questions

以上 Motivation 从一个可验证的 gap 出发：现有联邦因果特征选择方法在低功效客户端上无法恢复弱稳定特征。由此推导三层 RQ。

#### 主要研究问题（Primary RQ）

**RQ1：在客户端样本量长尾、Non-IID 且存在低功效客户端时，如何恢复弱稳定因果特征（弱但全局的父变量），并避免被误判为局部伪特征而删除？**

- **对应 gap**：FedCIFL 的效应聚合在小样本客户端上功效不足，弱稳定特征被误删（本 pilot 已证实稳定召回 0.875）[1]
- **新颖性**：现有联邦因果方法（FedCIFL/FedLCS/FedCMFS）均未引入语义先验；LLM 语义先验的因果工作均为集中式 [4]
- **可回答性**：主实验——合成多客户端 SCM + 半合成公开数据，度量稳定特征 recall/precision 与未见客户端 AUC

#### 次要研究问题（Secondary RQs）

**RQ2：语义先验能否区分「弱但全局的稳定特征」与「只在少数客户端成立的局部伪特征」，从而在救回弱稳定特征的同时不放大局部误纳入？**

- **与 RQ1 的关系**：回答 RQ2 即为 RQ1 提供方法设计依据——语义先验作为客户端证据状态的信号（而非检验阈值调节）
- **对应 gap**：现有门控规则（V1-sign）无法区分 `Client-Local` 状态中的弱全局特征与真局部伪特征 [本项目 pilot]
- **对应实验**：消融——去掉语义先验（V1-sign 对照）、匿名先验对照、噪声先验

**RQ3：语义先验存在噪声时，冲突 gate 能否兜住数据可检测的错误（符号冲突）；数据不可检测的定位型错误在什么条件下无法纠正，从而构成可识别性边界？**

- **与 RQ1/RQ2 的关系**：RQ3 追问方法的适用边界与先验可靠性的极限，使论证更严谨
- **对应实验**：噪声先验 + conflict gate；可识别性边界的量化（定位型错误在弱全局效应下的不可区分性）

### 3 Key Works

选取逻辑：以「联邦因果特征选择」为靶子，「LLM 语义先验」「先验可靠性」为方法来源，「联邦+LLM 交叉」为新颖性依据，各选最直接相关的工作。

| 简称 | 会议/期刊 | 年份 | 核心贡献（一句话） | 对本研究的借鉴价值 |
|------|----------|------|-----------------|----------------|
| FedCIFL | AAAI | 2025 | 联邦因果不变特征学习，样本重加权+迭代收缩混杂集 | 主靶子与代码基础，其低功效误删即本文要解决的问题 |
| FedCMFS | arXiv | 2024 | 联邦多标签因果特征选择（label blocking 等） | 最直接查新对象，界定多标签边界 |
| FedLCS | SCIS | 2025 | 联邦单目标局部 PC 结构学习 | 界定「联邦局部结构」已有工作，本文落在稳定父变量+语义先验 |
| DMCD | arXiv | 2026 | LLM 语义草稿 DAG + CI 检验审计 | 最接近的集中式「语义+统计」框架，本文差异在联邦 |
| LLM Sparsity Prior | arXiv | 2026 | LLM 权重→Spike-and-Slab 先验，低数据下特征选择 | 集中式低数据先验的近邻，本文须落在联邦+因果 |
| CauTion | arXiv | 2026 | LLM 先验可靠性估计 + 集成仲裁 | 先验可靠性机制的参考，本文在联邦用客户端做校准 |
| LLM Cannot Discover Causality | arXiv | 2025 | 证明 LLM 应限于非决定式辅助，不可调节 CI 统计量 | 本文方法约束的直接来源 |
| FedCDI Survey | arXiv | 2026 | 联邦因果发现综述，明确 LLM 先验为 open problem | 新颖性背书与边界定位 |

#### FedCIFL（AAAI 2025）[1]

联邦特征选择中学习因果不变特征：设计样本重加权消除选择偏差引入的伪相关，迭代估计每个特征与标签的联邦因果效应并收缩混杂集。

> 借鉴价值：本文的靶子方法与本项目已复现的代码基础，其绝对值聚合在小样本客户端上的功效不足正是本文要解决的痛点
> 是否关键工作：是，因为它是主靶子且已复现

#### DMCD（arXiv 2026）[4]

两阶段框架：LLM 从变量元数据生成稀疏草稿 DAG 作为语义先验，再用 CI 检验审计修正；消融表明收益来自语义推理而非记忆 benchmark 图。

> 借鉴价值：最接近的集中式「语义+统计」框架，本文的差异点在联邦多客户端与未见客户端评价
> 是否关键工作：是，因为它是集中式最近邻，用于界定本文新颖性

#### LLM Sparsity Prior（arXiv 2026）[3]

把 LLM 生成的权重注入 Spike-and-Slab 先验的 inclusion probability，低数据区间（n=100~200）收益最明显。

> 借鉴价值：集中式「LLM 先验 + 低数据特征选择」的近邻；本文须明确落在联邦+因果，避免与其重叠
> 是否关键工作：是，因为它是需要正面区分的近邻

#### LLM Cannot Discover Causality（arXiv 2025）[5]

证明 LLM 的回归式建模缺乏因果推理的理论根基，应限制为非决定式辅助（如启发式搜索），尤其不应把先验注入 CI 检验统计量。

> 借鉴价值：本文方法边界（先验只做证据状态、不做检验阈值）的直接依据
> 是否关键工作：是，因为它规定了本文方法不能走的路线

---

## References

[1] Guo, Xianjie, et al. "Federated Causally Invariant Feature Learning." *Proceedings of the AAAI Conference on Artificial Intelligence*, vol. 39, no. 16, 2025.
> 主要工作：联邦因果不变特征学习
> 引用原因：主靶子方法，已复现
> PDF：`E:\因果研究\lunwen\Guo 2025 - Federated Causally Invariant Feature Learning (AAAI).pdf`

[2] FedSDWC. "Federated Synergistic Dual-Representation Weak Causal Learning for OOD." *AAAI*, 2026.
> 主要工作：联邦因果 OOD
> 引用原因：2026 年 OOD 基线
> PDF：`docs/papers/FedSDWC_AAAI2026.pdf` [待核实本地路径]

[3] "LLM Sparsity Prior for Robust Feature Selection." *arXiv:2605.23102*, 2026.
> 主要工作：LLM 先验低数据特征选择
> 引用原因：集中式近邻，需正面区分
> PDF：`[PDF 不可用]`

[4] "A Survey on Federated Causal Discovery and Inference." *arXiv:2606.23741*, 2026.
> 主要工作：联邦因果发现综述
> 引用原因：LLM 先验列为 open problem 的背书
> PDF：`[PDF 不可用]`

[5] "LLM Cannot Discover Causality, and Should Be Restricted to Non-Decisional Support in Causal Discovery." *arXiv:2506.00844*, 2025.
> 主要工作：LLM 因果能力的理论限制
> 引用原因：方法约束来源
> PDF：`[PDF 不可用]`

---

## 待核实清单
> 由 AI 自动维护，人工核实后逐一打勾。

- [ ] FedSDWC、DMCD、LLM Sparsity Prior、CauTion、FedCDI Survey 的 PDF 尚未下载到本项目 `docs/papers/`（位置：Section 3 Key Works）
- [ ] 引用编号 [2][3][4][5] 的完整题录信息需经 web_search 最终核验（位置：References）

---

> ⚠️ **阶段 A 完成节点**
> 请审查 Part 1 的 Motivation、三层 RQ 与 Key Works 后，确认是否可以进入阶段 B（Idea 深化）。
> - 可以继续 → 进入阶段 B
> - 需要修改 → 提出意见，迭代更新后再次确认
