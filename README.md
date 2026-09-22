# 论文深度解析：面向多元序列相关数据的无参数控制图

> **原文标题**：A Data-Driven Nonparametric Control Chart for Multivariate Serially Correlated Data Monitoring in Advanced Industrial Scenarios
> **期刊**：Computers & Industrial Engineering, 216 (2026) 112015
> **DOI**：10.1016/j.cie.2026.112015
> **作者**：Cang Wu, Dong Wang, Min Luo, Yongjun Du（兰州理工大学）；Wenpo Huang（杭州电子科技大学）；Lijun Shang（佛山大学）；**Shubin Si（通讯，西北工业大学）**
> **关键词**：Nonparametric charts（无参数控制图）、MEWMA、RF algorithm（随机森林）、Decorrelation（去相关）、Recursive computation（递归计算）
> **解析对象**：`C:\Users\admin\Downloads\1-s2.0-S0360835226002160-main.pdf`（20 页）

---

## 1. 研究问题与动机

现代智能制造中多传感器网络产生海量**多元、序列相关（自相关）、分布未知**的时间序列数据。传统 MSPC（多元统计过程控制）方法面临三重失效：

1. **独立性假设失效**——Shewhart / MCUSUM / MEWMA 等经典图均假设观测独立且多元正态，实际工业数据存在强自相关与非平稳性；
2. **分布假设失效**——实际数据多为未知非高斯分布，参数化控制图不可靠；
3. **已有无参数方法仍假设时间独立**，且现有融合机器学习的方案**只检测正漂移**，忽略同样危害质量的负漂移，对早期微弱故障（易被去相关后的噪声与残余相关掩盖的 "masking effect"）敏感度不足。

本文提出 **C-RF-MEWMA**（Cholesky–Random Forest–MEWMA）控制图：先用 Cholesky 分解剔除序列相关，再用 MEWMA 递归序列喂给随机森林分类器，**以 RF 输出的受控（IC）概率直接作为监控统计量**，同时覆盖正、负漂移，全程无分布假设。

---

## 2. 方法体系：C-RF-MEWMA 四步框架

方法聚焦 Phase II（在线监控）。设在线观测为 $x_i$，历史受控/失控数据为 $X_{ic}$、$X_{oc}$，四步流程如下：

| 步骤 | 目标 | 方法/模型 | 输入 → 输出 |
|---|---|---|---|
| Step 1 | 去除序列相关 | Cholesky 分解 | p 维序列相关数据 → p 维标准化独立数据 |
| Step 2 | 数据变换（信息增强） | MEWMA 递归 | 去相关标准化数据 → MEWMA 序列 $E_i$ |
| Step 3 | 构建判别模型 | 随机森林（RF） | 历史 MEWMA 序列 → IC 概率 $p(E_i \in IC)$ |
| Step 4 | 在线监控 | 控制图 | 在线 MEWMA 序列 + 控制限 $L$ → $p(E_i \in IC) \leqslant L$ 时报 OC 警 |

> **核心创新点**：RF 不是传统意义上的特征提取器，而是**判别式统计量生成引擎**——通过汇总 $T$ 棵决策树的投票直接输出概率型监控统计量；MEWMA 采用**分段递归**形式防止统计量随时间收敛到稳态（见式 (3)(4)）。

---

## 3. 关键符号总表

| 符号 | 含义 |
|---|---|
| $p$ | 过程维度（质量特性个数） |
| $X_n = (X_{n1}, X_{n2}, \ldots, X_{np})'$ | $n$ 时刻的 $p$ 维观测向量（$n \geqslant 1$） |
| $\mu$ | 过程均值向量 |
| $\gamma(s) = \mathrm{Cov}(X_i, X_{i+s})$ | 滞后 $s$ 的自协方差矩阵；平稳假设下只依赖 $s$ 而非 $i$ |
| $b_{\max}$ | 序列相关时域上限，$s > b_{\max}$ 时 $\gamma(s) = 0$（短程相关假设） |
| $X_{IC} = \{X_{-n_0+1}, \ldots, X_{-\tau_0}\}$ | Phase I 历史受控（IC）数据集 |
| $X_{OC} = \{X_{-\tau_0+1}, \ldots, X_0\}$ | Phase I 历史失控（OC）数据集 |
| $n_0,\ \tau_0$ | IC / OC 历史样本规模参数 |
| $\hat{\mu}^{(0)},\ \hat{\gamma}^{(0)}(s)$ | IC 状态下的均值向量、自协方差矩阵矩估计 |
| $\hat{\mu}^{(1)},\ \hat{\gamma}^{(1)}(s)$ | OC 状态下的均值向量、自协方差矩阵矩估计 |
| $\hat{\Sigma}_{i,i}$ | 长向量 $(X_{i-b}, X_{i-b+1}, \ldots, X_i)$ 的协方差矩阵估计（分块结构） |
| $\hat{\sigma}_{i-1}$ | 当前观测与前 $b$ 个观测的互协方差堆叠向量 |
| $\hat{e}_{i-1}$ | 前 $b$ 个中心化观测的堆叠长向量 |
| $\hat{d}_i$ | 剔除历史线性影响后的条件残差协方差（$p \times p$） |
| $L_i,\ Q_i$ | Cholesky 变换的下三角矩阵与变换后块对角协方差 |
| $X_i^*,\ x_i^*$ | 去相关并标准化后的观测（历史 / 在线），近似 i.i.d. 且协方差为 $I_p$ |
| $\xi_j$ | 去相关后、标准化前的残差向量 |
| $\lambda$ | MEWMA 平滑系数（取 0.2） |
| $M$ | MEWMA 分段长度 / 记忆窗（取 15） |
| $S_{m,l}$ | 第 $m$ 段、第 $l$ 步的 MEWMA 递推向量 |
| $E_i$ | MEWMA 序列输出（RF 的输入特征向量） |
| $T$（$n_{tree}$） | RF 中决策树的数量（取 300） |
| $w_t^j(x) \in \{0,1\}$ | 第 $t$ 棵树将 $x$ 判为类别 $c_j$ 的投票 |
| $W(x)$ | RF 集成分类结果 |
| $p(x \in c_j),\ p(E_i \in IC)$ | RF 输出的类别概率；本文即监控统计量 |
| $c_1 = IC,\ c_2 = OC$ | 两个类别 |
| $L$ | 控制限（统计量下侧阈值，二分法 + Monte Carlo 标定） |
| $RL,\ ARL$ | 运行长度、平均运行长度 |
| $ARL_0,\ ARL_1$ | 受控 ARL（目标 200）、失控 ARL（越小越好） |
| $n_0$（标定语境） | Monte Carlo 重复次数（取 10000；与历史样本参数 $n_0$ 复用同一符号，见原文 3.5.1） |
| $K,\ N_{MC},\ C$ | 二分迭代次数、每次迭代 MC 次数（10000）、生成 IC 数据块长 |
| $N_{train}$ | RF 训练集规模 |
| $mtry$ | 节点分裂随机特征子集大小（默认 $\sqrt{p}$） |
| $\delta$ | 均值漂移幅度（$-1$ 到 $1$，步长 0.25 等） |
| $\sigma_X$ | 某分量的 IC 标准差（Case I–III 中漂移量记为 $\delta\sigma_X$） |
| $\xi$（Case II/III） | $\chi^2_3$ 分布随机变量的标准化版本 |
| $A,\ B$（及 $A^*,\ B^*$） | Case IV / IV* 的 VAR(1) 系数矩阵与新息协方差矩阵 |
| $k$ | 对照图 G-MCUSUM / NP-MCUSUM 的容差常数（取 0.01） |
| $RMI$ | 相对均值指标（参数调优的综合评价量） |

---

## 4. 核心公式逐条解析

### 4.0 矩估计（式 (1) 的前置公式，无编号）

因 IC 分布无预设参数形式，MLE 不可行，改用**矩估计**：

$$
\hat{\mu}^{(0)} = \frac{1}{n_0 - \tau_0} \sum_{i=-n_0+1}^{-\tau_0} X_i
$$

$$
\hat{\gamma}^{(0)}(s) = \frac{1}{n_0 - s} \sum_{i=-n_0+1}^{-s} \big(X_{i+s} - \hat{\mu}^{(0)}\big)\big(X_i - \hat{\mu}^{(0)}\big)'
$$

OC 状态同理重估（上标 $(1)$）：

$$
\hat{\mu}^{(1)} = \frac{1}{\tau_0} \sum_{i=-\tau_0+1}^{0} X_i, \qquad
\hat{\gamma}^{(1)}(s) = \frac{1}{\tau_0 - s} \sum_{i=-\tau_0+1}^{-s} \big(X_{i+s} - \hat{\mu}^{(1)}\big)\big(X_i - \hat{\mu}^{(1)}\big)'
$$

> **解读**：$\hat{\mu}^{(0)}$ 为 IC 段样本均值；$\hat{\gamma}^{(0)}(s)$ 为滞后 $s$ 的样本自协方差矩阵，上标 $(0)/(1)$ 区分 IC/OC 状态，转置 $'$ 表示矩阵转置。协方差族 $\{\hat{\gamma}(s),\ 0 \leqslant s \leqslant b_{\max}\}$ 是后续递归去相关的全部输入。

### 4.1 式 (1)：长向量协方差矩阵的分块递推结构

对长向量 $\{X_{i-b}, X_{i-b+1}, \ldots, X_i\}$（$0 \leqslant b \leqslant b_{\max}$），其协方差矩阵呈**分块 Toeplitz** 结构，并可写成递归分块形式：

$$
\hat{\Sigma}_{i,i} =
\begin{pmatrix}
\hat{\gamma}^{(0)}(0) & \cdots & \hat{\gamma}^{(0)}(b) \\
\vdots & \ddots & \vdots \\
[\hat{\gamma}^{(0)}(b)]' & \cdots & \hat{\gamma}^{(0)}(0)
\end{pmatrix}
=
\begin{pmatrix}
\hat{\Sigma}_{i-1,i-1} & \hat{\sigma}_{i-1} \\
[\hat{\sigma}_{i-1}]' & \hat{\gamma}^{(0)}(0)
\end{pmatrix},
\qquad 0 \leqslant b \leqslant b_{\max},\ \ -n_0+1 \leqslant i \leqslant -\tau_0
\tag{1}
$$

$$
\hat{\sigma}_{i-1} = \big([\hat{\gamma}^{(0)}(b)]',\ \ldots,\ [\hat{\gamma}^{(0)}(1)]'\big)'
$$

> **解读**：对角块是零滞后协方差 $\hat{\gamma}^{(0)}(0)$（$p\times p$），第 $k$ 条对角线为滞后 $k$ 的互协方差 $\hat{\gamma}^{(0)}(k)$；右下角是"新"观测 $X_i$ 的方差，右上角 $\hat{\sigma}_{i-1}$ 堆叠了 $X_i$ 与前 $b$ 个历史观测的互协方差。该递推式使 $\hat{\Sigma}_{i,i}$ 能随新观测到来**增量扩展**，是递归计算的基础。注意 $-n_0+1$ 时 $b=0$，此后 $b = \min(b+1,\ b_{\max})$。

### 4.2 式 (2)：递归去相关与标准化变换

$$
X_i^* =
\begin{cases}
\big[\hat{\gamma}^{(0)}(0)\big]^{-1/2}\big(X_i - \hat{\mu}^{(0)}\big), & i = -n_0+1, \\[6pt]
\hat{d}_i^{-1/2}\Big[-\hat{\sigma}_{i-1}'\, \hat{\Sigma}_{i-1,i-1}^{-1}\, \hat{e}_{i-1} + \big(X_i - \hat{\mu}^{(0)}\big)\Big], & -n_0+1 \leqslant i \leqslant -\tau_0,
\end{cases}
\tag{2}
$$

其中

$$
\hat{e}_{i-1} = \big[(X_{i-b} - \hat{\mu}^{(0)})',\ (X_{i-b+1} - \hat{\mu}^{(0)})',\ \ldots,\ (X_{i-1} - \hat{\mu}^{(0)})'\big]', \qquad
\hat{d}_i = \hat{\gamma}^{(0)}(0) - \hat{\sigma}_{i-1}'\, \hat{\Sigma}_{i-1,i-1}^{-1}\, \hat{\sigma}_{i-1}
$$

> **解读**：
> - **第一分支**（首个观测）：仅做自身标准化——用 $[\hat{\gamma}^{(0)}(0)]^{-1/2}$ 白化。
> - **第二分支**（一般观测）：括号内 $\big(X_i - \hat{\mu}^{(0)}\big) - \hat{\sigma}_{i-1}'\hat{\Sigma}_{i-1,i-1}^{-1}\hat{e}_{i-1}$ 是**多元线性回归残差**——从当前中心化观测中扣除其被历史 $b$ 个观测线性解释的部分，$\hat{\sigma}_{i-1}'\hat{\Sigma}_{i-1,i-1}^{-1}$ 即回归系数矩阵；再乘 $\hat{d}_i^{-1/2}$ 将残差协方差白化为单位阵。
> - 效果：$X_i^*$ 序列近似 i.i.d.、协方差 $\approx I_p$，从而可套用标准无参数监控。
> - 原文该分支范围印作 $-n_0+1 \leqslant i \leqslant -\tau_0$（与第一分支下标重叠），按递推逻辑实际自 $i = -n_0+2$ 起使用。
> - 两个数值问题及对策：① $[\hat{\gamma}^{(0)}(0)]^{-1/2}$、$\hat{d}_i^{-1/2}$ 在小样本下可能无解析逆 → 用**伪逆**或增大样本（仿真经验：样本量 $\geqslant 100$ 可避免）；② $\hat{\Sigma}_{i,i}$ 未必正定 → 用 Higham (1988) 的**最近正定矩阵修正法**。

### 4.3 Cholesky 分解视角（式 (2) 的理论依据）

上述变换等价于对 $\hat{\Sigma}_{i,i}$ 做 Cholesky 分解 $\Rightarrow$ 存在下三角矩阵 $L_i$ 使

$$
L_i\, \hat{\Sigma}_{i,i}\, L_i' = Q_i, \qquad
Q_i = \mathrm{diag}\{\hat{d}_{i-b},\ \hat{d}_{i-b+1},\ \ldots,\ \hat{d}_i\}, \qquad
L_i = \begin{pmatrix} L_{i-1} & 0 \\ -\hat{\sigma}_{i-1}'\, \hat{\Sigma}_{i-1,i-1}^{-1} & I_{p \times p} \end{pmatrix}
$$

> $Q_i$ 是线性变换后长向量的协方差（块对角 = 各时刻不相关）；$Q_i^{-1/2} L_i \hat{e}_i$ 的协方差高度近似单位矩阵——这正是式 (2) 中先乘 $-\hat{\sigma}'\hat{\Sigma}^{-1}$ 扣相关、再乘 $\hat{d}^{-1/2}$ 白化的由来。

### 4.4 式 (3)(4)：分段 MEWMA 序列（防止稳态收敛）

针对大规模数据流，标准 MEWMA 会随时间收敛到固定稳态值而丧失信息，故仅用当前输入 $x_i^*$ 及其前 $M$ 段构造**滑动分段递归**：

$$
S_{m,l} = \lambda\, x^*_{m+l-1} + (1-\lambda)\, S_{m,l-1}, \qquad m \geqslant 1,\ \ 1 \leqslant l \leqslant M
\tag{3}
$$

$$
E_i =
\begin{cases}
S_{1,i}, & 1 \leqslant i \leqslant M, \\[4pt]
S_{i-M+1,\, M}, & i > M
\end{cases}
\tag{4}
$$

> **解读**：$\lambda \in (0,1)$ 为平滑因子——**小 $\lambda$ 利于检测微小漂移，大 $\lambda$ 利于检测大幅漂移**；$S_{m,0}$ 初始化为历史 IC 样本标准化后的均值。每个 $S_{m,\cdot}$ 从初值重启、递推 $M$ 步，输出序列 $E_i$ 既保留 EWMA 的信息累积能力，又因窗口滑动保持对新变化的敏感性。$E_i$（$p$ 维）即 RF 的输入特征。

### 4.5 式 (5)(6)(7)：随机森林投票与概率输出

设 RF 含 $T$ 棵决策树，共 $j$ 个类别；对 $p$ 维输入 $x$，第 $t$ 棵树对类别 $c_j$ 的输出为 $w_t^j(x) \in \{0,1\}$（判为 $c_j$ 则取 1）。集成预测：

$$
W(x) = c_{\,\arg\max_j \sum_{t=1}^{T} w_t^j(x)}
\tag{5}
$$

$$
p(x \in c_j) = \frac{\sum_{t=1}^{T} w_t^j(x)}{T}
\tag{6}
$$

取 $c_1 = IC$，监控统计量定义为：

$$
p(x \in IC) = \frac{\sum_{t=1}^{T} w_t^1(x)}{T}
\tag{7}
$$

> **解读**：$p(x \in IC)$ 就是**投"受控"票的树的比例**——值越高越可信为 IC，越低越可信为 OC。相比硬分类，该软概率输出提供了可与阈值比较的连续统计量。RF 的三项优势：① 无参数、无需显式特征选择与分布假设；② 对工业噪声与离群点稳健；③ 集成机制提升泛化、抑制对特定故障模式的过拟合。

### 4.6 式 (8)：报警准则与控制限

$$
p(E_i \in IC) \leqslant L \quad \Longrightarrow \quad \text{触发 OC 报警}
\tag{8}
$$

**控制限 $L$ 的标定（Monte Carlo + 二分法）**：

$$
ARL(L) = \frac{RL_1 + RL_2 + \cdots + RL_{n_0}}{n_0}
$$

每次 MC 迭代：从标准化历史 IC 数据中随机抽取数据块，逐点计算 $p(E_i \in IC)$，若 $> L$ 继续抽取下一段，直到统计量 $\leqslant L$（触发信号），此时累计的 IC 点数即一次运行长度 $RL$。重复 $n_0 = 10000$ 次取均值得 $ARL(L)$。

二分法依据：统计量取值于 $[0,1]$，且 **$ARL_0$ 是 $L$ 的单调递减函数**——初值 $[h_l, h_u] = [0, 1]$（原文写作上界 $h_l$ 初值 1、下界 $h_u$ 初值 0，命名与语义互换，本质为 0–1 区间二分），取中点 $L = (h_l + h_u)/2$ 计算 $ARL_0$：偏小则收缩上半区间，偏大则收缩下半区间，直至 $|ARL_0 - \text{目标值}| < \varepsilon$。全文设定目标 $ARL_0 = 200$。

### 4.7 式 (9)：相对均值指标 RMI（参数调优准则）

$$
RMI = \frac{1}{n} \sum_{i=1}^{n} \frac{ARL_{1i}(R) - \min\big(ARL_{1i}\big)}{\min\big(ARL_{1i}\big)}
\tag{9}
$$

> **解读**：$ARL_{1i}(R)$ 是参数组合 $R$ 在第 $i$ 个场景下的失控 ARL；$\min(ARL_{1i})$ 是该场景下所有候选参数中的最优（最小）失控 ARL。RMI 衡量参数 $R$ 相对逐场景最优的**平均相对损失**，越小越好——同时兼顾各漂移水平，避免参数只对个别场景过拟合。

### 4.8 计算复杂度（式 (9) 后的讨论）

| 环节 | 复杂度 |
|---|---|
| Cholesky 去相关（离线 / 单点在线） | $O\big((p \cdot b_{\max})^3\big)$ ← **主导项** |
| RF 训练 | $O\big(n_{tree} \cdot p \cdot N_{train} \cdot \log N_{train}\big)$ |
| MEWMA 序列更新 | $O(p \cdot M)$ |
| RF 单次预测 | $O\big(n_{tree} \cdot p \cdot \log N_{train}\big)$ |
| 控制限搜索（整体） | $O\Big(K \cdot N_{MC} \cdot C \cdot \big((p \cdot b_{\max})^3 + p \cdot M + n_{tree} \cdot p \cdot \log N_{train}\big)\Big)$ |

> 整体可扩展性由去相关的立方复杂度 $O((p \cdot b_{\max})^3)$ 决定——这也是结论中指出的高维瓶颈（协方差求逆的计算量与数值稳定性问题）。

### 4.9 附录：递归去相关公式的完整推导（归纳构造）

**第 1 个观测**（协方差仅为 $\hat{\gamma}^{(0)}(0)$）：

$$
X_1^* = \big[\hat{\gamma}^{(0)}(0)\big]^{-1/2}\big(X_1 - \hat{\mu}^{(0)}\big)
$$

> 注：原文附录此式右端误印为 $(X_i - \hat{\mu}^{(0)})$，据上下文应为 $(X_1 - \hat{\mu}^{(0)})$。

**第 2 个观测**：长向量 $(X_1, X_2)$ 的协方差 $\hat{\Sigma}_{2,2} = \begin{pmatrix} \hat{\gamma}^{(0)}(0) & \hat{\sigma}_1 \\ \hat{\sigma}_1' & \hat{\gamma}^{(0)}(0) \end{pmatrix}$，其中 $\hat{\sigma}_1 = \hat{\gamma}^{(0)}(1)$。Cholesky 分解 $L_2 \hat{\Sigma}_{2,2} L_2' = Q_2$ 给出：

$$
L_2 = \begin{pmatrix} I_{p\times p} & 0 \\ -\hat{\sigma}_1'\big[\hat{\gamma}^{(0)}(0)\big]^{-1} & I_{p\times p} \end{pmatrix}, \qquad
Q_2 = \mathrm{diag}\big(\hat{d}_1,\ \hat{d}_2\big)
$$

$$
\hat{d}_1 = \hat{\gamma}^{(0)}(0), \qquad \hat{d}_2 = \hat{\gamma}^{(0)}(0) - \hat{\sigma}_1'\big[\hat{\gamma}^{(0)}(0)\big]^{-1}\hat{\sigma}_1
$$

由 $L_2 \hat{e}_2 = (\xi_1',\ \xi_2')'$ 得两个**互不相关**的残差块：

$$
\xi_1 = X_1 - \hat{\mu}^{(0)}, \qquad
\xi_2 = -\hat{\sigma}_1'\, \hat{\Sigma}_{1,1}^{-1}\big(X_1 - \hat{\mu}^{(0)}\big) + \big(X_2 - \hat{\mu}^{(0)}\big), \qquad
X_2^* = \hat{d}_2^{-1/2}\, \xi_2
$$

**第 3 个观测**：$\hat{\sigma}_2 = \big([\hat{\gamma}^{(0)}(2)]',\ [\hat{\gamma}^{(0)}(1)]'\big)'，L_3 = \begin{pmatrix} L_2 & 0 \\ -\hat{\sigma}_2'\hat{\Sigma}_{2,2}^{-1} & I \end{pmatrix}$，$Q_3 = \mathrm{diag}(\hat{d}_1, \hat{d}_2, \hat{d}_3)$，$\hat{d}_3 = \hat{\gamma}^{(0)}(0) - \hat{\sigma}_2'\hat{\Sigma}_{2,2}^{-1}\hat{\sigma}_2$，且

$$
\xi_3 = -\hat{\sigma}_2'\, \hat{\Sigma}_{2,2}^{-1}\, \hat{e}_2 + \big(X_3 - \hat{\mu}^{(0)}\big), \qquad X_3^* = \hat{d}_3^{-1/2}\, \xi_3
$$

**一般形式（第 $j$ 个观测）**——设 $\hat{\sigma}_{j-1} = \big([\hat{\gamma}^{(0)}(j-1)]',\ \ldots,\ [\hat{\gamma}^{(0)}(1)]'\big)'$，则 $L_j \hat{\Sigma}_{j,j} L_j' = Q_j = \mathrm{diag}(\hat{d}_1, \ldots, \hat{d}_j)$，$L_j = \begin{pmatrix} L_{j-1} & 0 \\ -\hat{\sigma}_{j-1}'\hat{\Sigma}_{j-1,j-1}^{-1} & I \end{pmatrix}$，$\hat{d}_j = \hat{\gamma}^{(0)}(0) - \hat{\sigma}_{j-1}'\hat{\Sigma}_{j-1,j-1}^{-1}\hat{\sigma}_{j-1}$，于是

$$
\xi_j = -\hat{\sigma}_{j-1}'\, \hat{\Sigma}_{j-1,j-1}^{-1}\, \hat{e}_{j-1} + \big(X_j - \hat{\mu}^{(0)}\big), \qquad
X_j^* = \hat{d}_j^{-1/2}\, \xi_j
$$

> **归纳结论**：$X_j^*$ 与 $X_1^*, \ldots, X_{j-1}^*$ 不相关且协方差均为 $I_p$——序列相关被逐点递归剔除。

---

## 5. 参数设定与设计要点汇总

| 参数 | 取值 | 选择依据 |
|---|---|---|
| 平滑因子 $\lambda$ | **0.2** | RMI 准则：$\lambda \in [0.05, 0.2]$ 内稳健；$\lambda = 0.1$ 最差，$0.2$ 多数场景 RMI 最低 |
| 分段长度 $M$ | **15** | 文献建议 15–30（Chen et al., 2016）；$M = 15$ 在多数场景 RMI 最小；$M$ 过大会使序列过度平稳、钝化对漂移的敏感性 |
| 相关时域 $b_{\max}$ | **20** | 数值经验 $[10, 20]$ 为宜；敏感性分析（表 9）显示 10→20 性能渐优，25/30 反而下降（超出真实相关窗的估计噪声被 MEWMA 放大） |
| 决策树数 $n_{tree}$ | **300** | R `randomForest` 包；树数增益递减且有过拟合与耗时代价 |
| $mtry$ | $\sqrt{p}$（默认） | 检测小漂移时宜接近 $p$，检测大漂移时宜取 $\sqrt{p}$ |
| 训练样本量 | IC 500 / 每类 OC 500 | 8 种 OC 幅度（$\pm0.25, \pm0.5, \pm0.75, \pm1$）总量远超 IC，故 **IC 重采样**使两类平衡 |
| 样本量下限 | $p=3$ 时 $\geqslant 500$；$p=10$ 时 $\geqslant 1000$ | 表 6/7：样本 100 时实际 $ARL_0$ 偏离标称 $>10\%$ |
| 目标 $ARL_0$ | **200** | 全部实验统一 |
| MC 次数 $n_0$（标定） | **10000** | 控制限标定的蒙特卡洛重复次数 |
| 容差常数 $k$（对照图） | 0.01 | G-MCUSUM、NP-MCUSUM |

---

## 6. 仿真研究设计与关键结果

### 6.1 四种数据生成模型（$p = 3$）

- **Case I（独立基准）**：$X_1, X_2, \ldots$ i.i.d. $\sim N_3(0,\ I_{3\times3})$。
- **Case II（弱自相关 + 变量相关 + 非正态）**：
  $X_{n1} = 0.1 X_{n-1,1} + \varepsilon_n$（$X_{01}=0$，$\varepsilon_n \sim N(0,\ 0.1^2)$）；
  $X_{n2} = X_{n1} + 0.1\xi$（$\xi$ 为 $\chi^2_3$ 标准化变量，与 $\{X_{n1}\}$ 独立）；
  $X_{n3} = 0.2 X_{n-1,3} + \varepsilon_n$（与 $X_{n1}$ 独立）。
- **Case III（强自相关）**：同 Case II，但 $X_{n1} = 0.5 X_{n-1,1} + \varepsilon_n$，$X_{n3} = -0.5 X_{n-1,3} + \varepsilon_n$。
- **Case IV（全相关 VAR(1)）**：
  $$
  X_n = (1,\ 2,\ 1)' + A X_{n-1} + \varepsilon_n, \quad \varepsilon_n \ \text{i.i.d.} \sim N_3(0,\ B)
  $$
  $$
  A = \begin{pmatrix} 0.5 & 0 & 0 \\ 0 & 0.7 & 0 \\ 0 & 0 & 0.2 \end{pmatrix}, \qquad
  B = \begin{pmatrix} 1 & 0.2 & 0.2 \\ 0.2 & 1 & 0.2 \\ 0.2 & 0.2 & 1 \end{pmatrix}
  $$
- **Case IV\*（10 维扩展）**：$X_n = (1,2,1,1,2,1,1,2,1,1)' + A^* X_{n-1} + \varepsilon_n$，$\varepsilon_n \sim N_{10}(0, B^*)$；$A^*$ 为对角阵（对角元 $0.5, 0.7, 0.2$ 循环），$B^*$ 对角元 1、非对角元 0.2。

漂移设定：Case I–III 各分量均值漂移 $\delta\sigma_X$；Case IV 直接漂移 $\delta$；$\delta \in [-1, 1]$、步长 0.25（调参与主对比），细化实验步长 0.1。

**对照图**：G-MCUSUM（Xue & Qiu, 2021，无参数自启动）、DKE-CUSUM（Wang et al., 2025，KNN-CUSUM）、NP-MCUSUM（Qiu, 2008）——对照图参数调至各自最优，$ARL_0$ 统一校准为 200。

### 6.2 关键结果

**表 5 实际 $ARL_0$（标称 200，样本 500）**：

| | Case I | Case II | Case III | Case IV |
|---|---|---|---|---|
| **C-RF-MEWMA** | 208 | 203 | 198 | 205 |
| G-MCUSUM | 204 | 196 | 195 | 195 |
| DKE-CUSUM | 204 | 203 | 195 | 195 |
| NP-MCUSUM | 124 | 68 | 42 | 37 |

> C-RF-MEWMA、G-MCUSUM、DKE-CUSUM 的 IC 可靠性良好；NP-MCUSUM 失控（不自启动、不利用机器学习挖掘历史信息）。

**表 6/7 样本量影响**（实际 $ARL_0$）：$p=3$ 时样本 100 偏差超 10%（如 Case II 仅 160.61），样本 500/1000 稳定（约 198–208）；$p=10$（Case IV\*）需样本 $\geqslant 1000$。

**表 8 未见漂移的泛化能力**（训练用 $\pm0.25/\pm0.5/\pm0.75/\pm1$，测试 $\pm0.2/\pm0.4/\pm0.6/\pm0.8$，$ARL_1$）：

| $\delta$ | I | II | III | IV |
|---|---|---|---|---|
| -0.8 | 7.01 | 5.16 | 5.99 | 2.64 |
| -0.6 | 9.07 | 7.25 | 8.92 | 3.11 |
| -0.4 | 15.36 | 11.97 | 18.72 | 3.82 |
| -0.2 | 30.93 | 24.68 | 29.66 | 29.95 |
| +0.2 | 35.76 | 28.04 | 26.58 | 31.65 |
| +0.4 | 17.39 | 11.41 | 14.20 | 11.34 |
| +0.6 | 10.54 | 6.80 | 7.23 | 6.08 |
| +0.8 | 7.03 | 4.79 | 5.27 | 4.85 |

> 对训练未见的漂移幅度仍保持小 $ARL_1$，**正负漂移对称覆盖**，泛化能力良好。

**图 4（OC 对比）结论**：Case I 各法相当（G-MCUSUM 略优）；**Cases II–IV（含序列相关）中 C-RF-MEWMA 全面最优**，Case IV（同时含时间相关与变量相关）优势显著——验证其对相关数据的监控能力。DKE-CUSUM 小漂移与 G-MCUSUM 相当、大漂移退化；NP-MCUSUM 最差。文中引 "masking effect"（Apley & Tsung, 2002）解释去相关会削弱漂移效应、降低灵敏度，而 RF-MEWMA 组合缓解了该问题。

**图 5–7（样本量 × 维度）**：样本量越大越好；$n=100$ 时小漂移 ARL 异常（Cholesky 参数估计不准）；固定样本量下维度升高（$p: 3 \to 10$）性能下降，尤以小漂移为甚。

**表 9（$b_{\max}$ 敏感性，Case II）**：$b_{\max} = 10/15/20/25/30$ 中 20 附近最优（如 $\delta=-0.25$ 时 $ARL_1$：32.06/27.24/**24.46**/24.47/30.86），过大反而因估计噪声劣化。

---

## 7. 实证案例：半导体制造过程（UCI SECOM 数据集）

- **数据来源**：UCI Machine Learning Repository（SECOM，`archive.ics.uci.edu/dataset/179/secom`），2008 年 7–10 月某半导体制造过程的自动化监控系统采集。
- **数据构成**：沿用 Xue & Qiu (2021) 的选取——3 个质量变量（V1, V2, V3）共 500 个观测；**前 400 点为 IC，后 100 点为 OC**。
- **探索性验证**：
  - 正态 Q-Q 图显示 IC 数据**显著偏离正态**；
  - ACF 图显示 **V2、V3 存在显著自相关**（序列相关成立）；
  - 置换检验（permutation test）变量间相关显著性：$(V_1,V_2)$ $p=0.4149$、$(V_1,V_3)$ $p=0.1912$、$(V_2,V_3)$ **$p=0.0042$** → 仅 V2–V3 显著相关。
- **建模设置**：样本不足，训练阶段用**重采样**将 IC、OC 样本各扩至 500；G-MCUSUM 容差常数 $k = 0.01$；两图 $ARL_0 = 200$。
- **结果**：对后 100 个 OC 观测，**C-RF-MEWMA 在第 9 个观测点报警，G-MCUSUM 在第 23 点报警，提前 14 个点**——验证了框架对半导体制造过程数据分布漂移检测的有效性与更快响应。

---

## 8. 结论、贡献与局限

**主要贡献**：
1. 首个面向"多传感器 + 序列相关 + 未知分布"智能制造质量监控场景的系统化无参数框架 C-RF-MEWMA，四技术集成（Cholesky 递归去相关、分段 MEWMA、RF 概率统计量、MC+二分控制限标定）。
2. RF 的创新用法：**概率输出直接充当监控统计量**（而非仅作分类/特征提取），同时检测正、负漂移。
3. 统计严谨性：固定 $ARL_0$ 标定控制限、以 $ARL_1$ 评估效能，沿用经典控制图评价体系。
4. 大样本历史数据的充分利用（重采样平衡 + 大样本训练），克服既有小样本方案训练不足与 Cholesky 求逆失败的缺陷。

**局限与未来工作**：去相关需计算逆协方差矩阵，观测维度高时计算量大（$O((p \cdot b_{\max})^3)$）且可能**数值不稳定**——作者正针对高维场景开展改进研究。

---

## 附录 A：算法执行细节（Algorithm 2 摘要）

- **Part 1 数据生成与去相关**：对 IC 数据计算 $\hat{\mu}^*$ 与 $\hat{\gamma}^{(0)}$；$i=1$ 时 $X_1^* = \hat{\gamma}^{(0)-1/2}(X_1 - \mu^*)$；$i \geqslant 2$ 时 $b = \min(i-1,\ b_{\max})$，构造 $\hat{\Sigma}_{i,i}$、计算 $\hat{d}_i = \hat{\gamma}^{(0)} - \hat{\sigma}_{i-1}' \times \mathrm{inv}(\hat{\Sigma}_{i-1,i-1}) \times \hat{\sigma}_{i-1}$ 并得到 $X_{ic}^*$；OC 数据同法。
- **Part 2 训练 RF**：双层循环算 $S_{m,l}$（$l=1$ 时 $S = \lambda x^* + (1-\lambda)S_{m,0}$，否则 $\lambda x^* + (1-\lambda)S_{m,l-1}$）→ 抽取 $E_i$ → IC 重采样平衡 → 打 IC/OC 标签 → `randomForest(Class ~ ., ntree = 300, importance = TRUE, type = "classification")`（R 实现）。
- **Part 3 控制限**：$h_l=1,\ h_u=0,\ L=0.5$ 起步；每轮 $N_{MC}=10000$ 次生成参考 IC 数据 → 去相关 → MEWMA → RF 概率 $p_i$；$p_i < L$ 时记录 $RL_i$（未触发取序列长度）；$ARL_0^{act} = \sum RL / N_{MC}$；$|ARL_0^{act} - ARL_0^{tgt}| < \varepsilon$ 则停，否则按 $ARL$ 与目标的大小关系收缩二分区间。

## 附录 B：原文图表索引

| 编号 | 内容 |
|---|---|
| Table 1 | Cholesky 分解在 SPC 中的应用综述（维度/分布/机器学习/漂移方向/相关性对比） |
| Table 2 | C-RF-MEWMA 实施步骤（Algorithm 1） |
| Tables 3–4 | $M$、$\lambda$ 的 RMI 调参结果（结论：$M=15$、$\lambda=0.2$） |
| Fig. 3 | 参数选择气泡图（RMI 越小颜色越深、点越小） |
| Table 5 | 四图实际 $ARL_0$ 对比 |
| Tables 6–7 | 样本量（$p=3$/ $p=10$）对实际 $ARL_0$ 的影响 |
| Fig. 4 | 四图最优 $ARL_1$ 对比（$p=3$，$ARL_0=200$，样本 500） |
| Table 8 | 未见漂移幅度的监控性能 |
| Figs. 5–7 | $p = 3, 5, 10$ 下不同样本量的 $ARL_1$ |
| Table 9 | $b_{\max} \in \{10, 15, 20, 25, 30\}$ 敏感性分析 |
| Figs. 8–11 | SECOM 数据：原始序列、Q-Q 图、ACF 图、散点图矩阵 |
| Fig. 12 | 案例监控图：C-RF-MEWMA（第 9 点报警）vs G-MCUSUM（第 23 点报警） |
| Fig. 1–2 | 方法流程图与系统架构图 |
