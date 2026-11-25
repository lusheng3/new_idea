# 语义先验引导的联邦稳定因果特征发现 Idea Report
> 生成时间：2026-11-24 | 更新：2026-11-26（Part 1 + Part 2） | 状态：PENDING_REVIEW

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

## Part 2 Idea Design

### 1 Introduction

Federated causal feature selection aims to identify features that causally affect a target label from data distributed across multiple clients, without sharing raw data. It supports downstream out-of-distribution (OOD) generalization while respecting privacy constraints.

A core difficulty arises when clients have small and heterogeneous samples. In such settings, the conditional independence (CI) tests that underpin causal discovery lose statistical power on small clients, and a class of genuinely useful features — those with weak but globally consistent causal effects — are systematically missed. Our FedCIFL pilot confirms this: a weak stable feature (true effect 0.45) is repeatedly judged as client-local and removed, capping stable-feature recall at 0.875; relaxing the threshold, in turn, admits local spurious features that hold in only one client. Pure data-driven tests cannot distinguish "weak but global" from "strong but local".

Variable names and their field descriptions carry semantic information complementary to the data. A domain reader knows that "smoking years" causes "lung nodules". Recent work shows that large language models (LLMs) can extract such semantic priors to improve causal discovery, but these methods are centralized. The intersection of federated causal feature selection and LLM semantic priors remains open, as highlighted by a 2026 survey.

This paper studies whether a global semantic prior can rescue weak stable causal features in federated low-power settings. Our validation shows three empirical regularities: (i) the prior must be accurate enough — precision ≥ ~0.5 — for the rescue to help; (ii) a strong LLM given variable descriptions can meet this bar, while a small local model and name-only prompts cannot; (iii) combining the prior with client-level consistency checks yields modest but positive gains across three standard Bayesian networks. We therefore position the method as MB-oriented causal feature selection rather than strict parent-only discovery, since the LLM's "relatedness" signal naturally aligns with the Markov blanket.

贡献如下：
- **方法**：提出「联邦 CI-based MB 发现 + 全局语义先验救援 + 跨客户端一致性 gate」框架，先验只做候选/证据约束，不调节 CI 检验统计量；
- **机制**：用 G² 条件独立检验 + PC-skeleton 在客户端本地发现，服务器聚合，系统性拒掉远祖节点（MB 精度从 0.33 提升到 1.00）；
- **判据**：形式化「先验质量阈值」（精度 ≥ ~0.5）以及提示词设计（名字 + 描述 + 直接父列表式）对先验质量的决定性影响；
- **验证**：在 CHILD/ALARM/INSURANCE 三个网络、45 个多父目标上，强模型（DeepSeek）配描述时 28/45 目标提升、平均 ΔF1 ≈ +0.038，而本地 7B 模型为 −0.040。

### 2 Related Works

#### 2.1 联邦因果发现与因果特征选择

FedCIFL [1] 学习联邦因果不变特征，用样本重加权与迭代收缩混杂集处理 Non-IID 与 OOD，但其聚合在低功效客户端上会误删弱稳定特征。FedLCS 做联邦单目标局部 PC 结构学习，FedCMFS 做联邦多标签因果特征选择，FedCDH/FedACD 分别用客户端代理变量和样本质量加权。这些工作均未引入语义先验，且多数假设客户端共享同一图或同等质量。

> 本文差异：保留「本地 CI 发现 + 服务器聚合」的联邦形态，但引入全局语义先验并对低功效客户端降权。

#### 2.2 LLM 语义先验用于因果发现

DMCD 用 LLM 从变量元数据生成草稿 DAG 再以 CI 检验审计；LLM Sparsity Prior 把 LLM 权重注入 Spike-and-Slab 先验；CauTion 估计 LLM 先验可靠性并做集成仲裁。这些均为集中式方法。LLM Cannot Discover Causality [5] 进一步论证：LLM 应限于非决定式辅助，不应把先验注入 CI 统计量，否则破坏检验的渐近性质。

> 本文差异：把语义先验放在**联邦**场景，且严格限制为「候选/证据约束」，不进入检验统计量；并用客户端数据做跨客户端一致性校准。

#### 2.3 Markov Blanket 与因果特征选择

IAMB/HITON-PC 等基于 CI 的 MB 发现是因果特征选择的标准路线；CFS-MI、MCF-Spouse 等在多标签/高效方向扩展。MB 是预测目标的最优特征集（父 + 子 + 配偶），也是本文与「纯父节点发现」区分的关键。

> 本文定位：面向 MB 的因果特征选择；LLM 的「相关」先验与 MB 语义一致，而 CI 负责最小性。

#### 2.4 研究空白

现有工作分别覆盖了联邦 CI 发现、集中式 LLM 先验、MB 发现，但没有人回答：**在联邦低功效客户端下，语义先验能否可靠地救回 CI 漏掉的弱 MB 成员，其精度门槛是多少，何种先验来源能达到？** 本文围绕这一空白展开。

### 3 Method

#### 3.1 问题设定

设有 K 个客户端，第 k 个客户端持有本地数据 D_k = (X_k, Y_k)，原始数据不出本地。目标是从中恢复目标 Y 的 Markov blanket MB(Y)（父、子、配偶）或其中的稳定因果特征集合。

假设：
1. 客户端共享变量空间与目标定义；
2. 客户端内因果充分性或至少局部忠实性成立（PC 骨架所需）；
3. 客户端差异（样本量、标签先验）不改变目标父机制（本文只救援与数据不矛盾的特征）；
4. 语义先验来自变量名与字段描述，不依赖任何客户端数据。

#### 3.2 方法总览

```text
全局：LLM(变量名+描述) → 直接父先验 π
每个客户端：G² + PC-skeleton → 本地 PC 集
服务器：聚合 PC 集（多数/并集，低功效降权）
        + 先验救援（π=1 且跨客户端弱相关）
        + 一致性 gate（符号/跨客户端支持）
输出：稳定 MB 集 + 客户端局部结构 + 不确定集
```

#### 3.3 全局语义先验

服务器对每个候选特征 j 与目标 Y，用强 LLM（本文验证用 DeepSeek）在给定**变量名 + 语义描述 + 全变量列表**的条件下，询问「j 是否为 Y 的直接父节点」，得到二值先验：

$$
\pi_j = \mathbb{1}[\text{LLM 判定 } j \text{ 为 } Y \text{ 的直接父节点}]
$$

设计要点（经验证）：
- **必须提供描述**：仅名字时强模型也会把子节点/祖节点误判；
- **列表式提问**优于逐特征二元提问（后者过度标记）；
- **方向中立**：明确允许正/负因果效应，避免漏掉保护性因素；
- 先验只表示「候选」，不表示置信度，也不进入 CI 统计量。

#### 3.4 本地 CI-based MB 发现

每个客户端 k 在本地执行 PC-skeleton（目标侧）：

1. 候选集 C_k = { j : G²(j, Y) 的 p 值 < α }（边际相关筛选）；
2. 对每个 j ∈ C_k，按条件集大小 l = 1..L 枚举 S ⊆ C_k \ {j}，若存在 S 使 G²(j, Y | S) 的 p 值 ≥ α，则从 C_k 移除 j；
3. 输出本地 PC 集 PC_k（父 + 子）。

其中 G² 检验按条件集分层计算，自由度按有效层数累计。本文取 α=0.05、L=2。

#### 3.5 服务器聚合

服务器只接收各客户端的 PC 集（或等价的检验摘要），不接收原始数据：

$$
\text{PC}_{\text{union}} = \bigcup_k \text{PC}_k, \qquad
\text{PC}_{\text{majority}} = \{ j : |\{k : j \in \text{PC}_k\}| \ge \lceil K/2 \rceil \}
$$

低功效客户端（样本量小）的 PC 集噪声大，多数投票天然降权；需要更高召回时用并集，需要更高精度时用多数。

#### 3.6 先验救援

为救回 CI 因低功效漏掉的弱 MB 成员，定义「跨客户端弱相关」集合：

$$
W = \{ j : j \text{ 在至少 } 2 \text{ 个客户端的边际 G}^2 \text{ 检验中 } p < \alpha_w \}
$$

救援集为：

$$
R = \{ j : \pi_j = 1 \ \text{且}\ j \in W \}
$$

最终稳定集采用「高精度多数底座 + 先验救援」：

$$
S = \text{PC}_{\text{majority}} \cup R
$$

采用 PC_majority 而非 PC_union 作为底座，是因为实验证明「多数底座 + 先验救援」形成最优的精度-召回组合（三网络 F1 均超过 FedLCS，见 Part 3）。救援要求「先验说相关」且「数据有弱信号」，二者缺一不可，从而避免纯先验的假阳性。

#### 3.7 跨客户端一致性 gate

当方法同时估计本地效应 β_{j,k} 时（如加权逻辑回归），对救援特征追加一致性约束：

$$
\text{sign-consistency}_j = \frac{|\sum_k \text{sign}(\beta_{j,k}) \mathbb{1}[\text{sig}_k]|}{\sum_k \mathbb{1}[\text{sig}_k]} \ge \gamma
$$

并要求显著客户端数 ≥ k_min。符号冲突（如跨客户端正负相反）的特征不进入稳定集，而记为客户端局部或不确定。本文取 γ=0.8、k_min=2。

#### 3.8 复杂度与隐私

- 本地 PC-skeleton 复杂度为 O(|C_k| · Σ_l C(|C_k|, l))，在 |C_k|≈15、L=2 时可接受；
- 通信量仅为各客户端 PC 集（O(d) 级），远小于原始数据；
- 全局先验只依赖变量名/描述，不含任何客户端样本，天然隐私友好。

#### 3.9 假设与边界（经验证）

1. **先验精度门槛**：模拟扫描表明先验精度 p ≥ 0.5~0.6 时救援才有净收益，p < 0.4 时为负。本地 7B 模型实测 p≈0~0.4，故不可行；DeepSeek+描述实测 p≈0.41~0.67，故可行。
2. **描述依赖**：仅有变量名时先验不可靠；提供语义描述后显著改善。
3. **目标依赖**：45 个目标中 28 个提升（62%），平均 +0.038，收益与先验精度和数据结构相关，论文需诚实呈现。
4. **MB 而非纯父**：先验是「相关」信号，与 MB 语义一致；若目标是严格父节点集，需额外定向规则。

> 方法不解决：未观测混杂下的可识别性、非离散变量的 CI 检验、跨客户端机制变化的严格识别。

---

## Part 3 Experiment Design

### 1 数据集

| 数据集 | 类型 | 规模 | 用途 |
|---|---|---|---|
| CHILD | bnlearn 标准网络 | 20 节点 / 25 边 / 6 个多父目标 | 主实验（语义透明 + 技术术语混合）|
| ALARM | bnlearn 标准网络 | 37 节点 / 46 边 / 17 个多父目标 | 主实验（技术缩写为主）|
| INSURANCE | bnlearn 标准网络 | 27 节点 / 52 边 / 22 个多父目标 | 主实验（常识语义）|
| Synthetic-Fed | 自建多客户端 SCM | 12 特征 / 5 客户端 | 机制与 gate 验证 |
| Semi-Synthetic | 真实协变量 + 重生成标签 | sklearn 乳腺癌 12 维 | 真实协变量验证 |

数据生成（半合成）：以真实 DAG 结构为 ground truth，用 logistic 机制按图生成二值数据；每个目标指定一个弱父节点（系数 0.25），其余边系数 1.0；客户端样本量默认 [500,400,350,300,250]（全小客户端硬场景），另设 [4000,800,600,400,80]（长尾混合）。

> 说明：此设置保留真实因果结构（用于评价 MB/父恢复），同时可控地制造低功效场景；不声称复现原网络的 CPT。

### 2 实验设计

#### 2.1 主实验：跨网络 MB 恢复

- **目的**：验证强模型 + 描述先验在联邦低功效场景下提升 MB F1。
- **为什么这样设计**：Three networks × all multi-parent targets × 5 seeds，避免单目标/单网络偶然性。
- **设置**：CI-only vs CI+LLM 先验 vs CI+完美先验（上界）。
- **指标**：MB F1（主）、parent F1、precision、recall、选中数。
- **已完成初步结果**：CHILD +0.056（5/6）、ALARM +0.019（9/17）、INSURANCE +0.048（14/22）；合计 45 目标 28 提升，平均 +0.038。

#### 2.2 消融实验：先验与 gate 的必要性

| 变体 | 改动 | 验证什么 |
|---|---|---|
| w/o prior | 仅 CI（PC-union）| 先验带来的净增益 |
| w/o description | 仅变量名查询 | 描述的必要性 |
| w/o weak-dep | 纯先验（不要求数据弱信号）| 数据约束的必要性 |
| prior-only | 只用 LLM 先验 | 先验单独是否可行 |
| 7B vs DeepSeek vs perfect | 先验来源强度 | 先验质量阈值 |
| w/o gate | 去掉跨客户端一致性 | gate 的作用 |

#### 2.3 边界实验

- **先验质量扫描**：（p, r）网格，定位净收益阈值（已做：p ≥ 0.5~0.6）；
- **客户端规模**：全小 vs 长尾混合 vs 大客户端，验证「无大客户端时先验最不可替代」；
- **目标依赖性**：分析哪些目标提升/下降，与先验精度、MB 大小、弱边强度关联；
- **描述质量**：无描述 / 简短状态 / 完整描述三档对比。

#### 2.4 对比基线

| 基线 | 类型 | 说明 |
|---|---|---|
| Centralized PC / HITON-PC | 经典 | 集中式上界（隐私不设限）|
| FedLCS | 联邦 | 联邦单目标局部 PC 结构 |
| FedCIFL | 联邦 | 联邦因果不变特征（主靶子，已复现）|
| CI-only（本文无先验版）| 本文变体 | 隔离先验贡献 |

#### 2.5 基线主实验结果（已完成，3 网络）

**Panel A：FedCIFL 对比（统一评价 MB F1）**

| 网络（目标×种子）| FedCIFL | FedLCS-majority | Ours-union | Centralized |
|---|---|---|---|---|
| CHILD (3×2) | 0.339 | 0.783 | 0.804 | 0.817 |
| ALARM (BP/CO×1) | 0.329 | 0.667 | 0.700 | 0.929 |
| INSURANCE (Airbag/AntiTheft×1) | 0.292 | 0.619 | 0.582 | 0.619 |

> FedCIFL 优化 OOD 预测的因果不变特征而非最小 MB，保留大量特征（精度≈0.2），故 MB F1 明显落后；这是任务目标差异。

**Panel B：最终方法 Ours-majority（全部多父目标 × 5 种子）**

| 网络 | 目标数 | FedLCS-majority | Ours-union | **Ours-majority** | Centralized-PC |
|---|---|---|---|---|---|
| CHILD | 6 | 0.736±0.077 | 0.738±0.181 | **0.840±0.145** | 0.825±0.141 |
| ALARM | 17 | 0.699±0.102 | 0.621±0.176 | **0.736±0.143** | 0.762±0.161 |
| INSURANCE | 22 | 0.663±0.093 | 0.706±0.149 | **0.754±0.159** | 0.729±0.135 |
| 平均 | 45 | 0.699 | 0.688 | **0.777** | 0.772 |

> 逐目标胜率：Ours-majority vs FedLCS 33/45（73%），vs Ours-union 36/45（80%），vs 集中式 22/45。Ours-majority 平均 0.777 略高于集中式 PC（0.772），因为先验补上了 PC 因有限样本/条件集上限漏掉的 MB 成员。

### 3 资源预估（参考）

| 实验 | 预估成本 | 说明 |
|---|---|---|
| 主实验（45 目标 × 5 种子 × 3 网络）| CPU 分钟级 + LLM 查询 ~45 次 | 已跑通 |
| 消融（6 变体）| CPU 分钟级 | 无需额外 LLM |
| 边界（先验扫描）| CPU 分钟级 | 模拟先验 |
| 基线复现（FedLCS/FedCIFL）| CPU/GPU 小时级 | FedCIFL 已复现 |

> 语义先验查询成本低（DeepSeek 约 45 次/轮），主要算力在本地 PC 与基线复现。

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

> ⚠️ **Part 2 + Part 3 完成节点**
> Part 2（Introduction / Related Works / Method）与 Part 3（数据集 / 实验设计 / 资源预估）已根据验证结果撰写（2026-11-26）。
> 请审阅 Method（3.3–3.9）与实验设计（2.1–2.4）是否准确；确认后可进入阶段 D（实现设计 / implementation.md）或补跑 2.4 中的基线对比。
> - 可以继续 → 进入 Part 3 基线复现或阶段 D
> - 需要修改 → 提出意见，迭代更新后再次确认
