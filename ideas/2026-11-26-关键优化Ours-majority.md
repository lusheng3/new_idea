# 关键优化：PC-majority + 先验救援（Ours-majority）三网络全胜

> 日期：2026-11-26
> 结论：**用 PC-majority（高精度底座）+ 先验救援，Ours-majority 在 CHILD/ALARM/INSURANCE 上全部超过 FedLCS 和 Ours-union，并在 CHILD/INSURANCE 超过集中式 PC 上界。**

## 一、动机

此前 Ours 用 PC-union 作底座（高召回、低精度，F1 与 FedLCS 持平）。改为 PC-majority（FedLCS 式高精度底座）+ 先验救援，兼顾精度与召回。

## 二、结果（MB F1，6 目标 × 2 种子）

| 网络 | FedLCS-majority | FedLCS-union | Ours-union | **Ours-majority** | Centralized-PC |
|---|---|---|---|---|---|
| CHILD | 0.725 | 0.652 | 0.715 | **0.822** | 0.817 |
| ALARM | 0.704 | 0.587 | 0.608 | **0.779** | 0.929 |
| INSURANCE | 0.616 | 0.601 | 0.646 | **0.667** | 0.619 |

Ours-majority 的精度/召回：
- CHILD：P=0.93 R=0.76
- ALARM：P=0.96 R=0.70
- INSURANCE：P=0.85 R=0.57

## 三、结论

1. **Ours-majority 三网络全胜 FedLCS**（+0.097/+0.075/+0.051）；
2. **在 CHILD（0.822>0.817）和 INSURANCE（0.667>0.619）超过集中式 PC 上界**——先验补上了集中式 PC 漏掉的 MB 成员（配偶/子节点）；
3. **精度保持高**（0.85~0.96），召回显著高于 FedLCS（0.57~0.76 vs 0.45~0.57）；
4. 说明「高精度底座 + 先验补召回」是正确的组合。

## 四、方法更新

最终方法应定义稳定集为：

$$
S = \text{PC}_{\text{majority}} \cup R,\quad R = \{ j : \pi_j = 1 \ \text{且}\ j \in W \}
$$

即「多数投票的高精度 PC 集」+「先验救援」。

## 五、代码

- `refine_ours_variants.py`；
- `results/refine_{child,alarm,insurance}.csv`。
