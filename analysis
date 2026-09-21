# 论文精读解析

## 《A Data-Driven nonparametric control chart for multivariate serially correlated data monitoring in advanced industrial scenarios》

> 中文题名：**面向先进工业场景的多元序列相关数据监测：一种数据驱动的非参数控制图**
>
> 提出方法：**C-RF-MEWMA 控制图**（Cholesky decomposition – Random Forest – MEWMA）

---

## 一、论文基本信息

| 项目 | 内容 |
| --- | --- |
| 期刊 | Computers & Industrial Engineering（CIE，计算机与工业工程） |
| 卷期页 | Vol. 216 (2026) 112015 |
| DOI | 10.1016/j.cie.2026.112015 |
| 出版时间 | 2026 年 4 月 3 日在线发表 |
| 通讯作者 | Shubin Si（西北工业大学） |
| 主要作者 | Cang Wu、Dong Wang、Min Luo、Yongjun Du、Wenpo Huang、Lijun Shang、Shubin Si |
| 主要单位 | 兰州理工大学、杭州电子科技大学、佛山大学、西北工业大学 |
| 关键词 | Nonparametric charts、MEWMA、RF algorithm、Decorrelation、Recursive computation |
| 资助 | 国家自然科学基金（72561019、72231008、72561018）、陕西省科技创新团队、甘肃省自然科学基金 |

---

## 二、研究背景与问题定位

### 2.1 工业 5.0 时代的监测挑战

在现代智能制造系统中，多传感器网络被广泛部署用于实时监控设备运行状态和环境条件，产生**海量、多元、高质量**的时间序列数据，典型特征包括：

- **多元性（multivariate）**：每个时刻同时采集 p 维质量特征（如薄膜厚度、沉积温度、射频功率）；
- **序列相关（serial correlation）**：设备惯性、闭环反馈控制、多传感器同步感知导致相邻观测强相关；
- **分布未知（unknown distribution）**：真实数据通常非高斯、非参数化；
- **非平稳性（non-stationary）**：过程动态特性随时间演化。

### 2.2 传统方法的不足

经典 MSPC（Multivariate Statistical Process Control，多元统计过程控制）方法（Hotelling $T^2$、MCUSUM、MEWMA）建立在两个强假设之上：

1. **观测独立**（无序列相关）；
2. **多元正态分布**。

而现实工业数据几乎从不满足这两个假设，导致传统控制图**误报率高、漏报率高**，无法可靠部署。

### 2.3 现有非参数方法的两大流派

| 流派 | 核心思想 | 代表工作 |
| --- | --- | --- |
| **隐式自适应（Implicit adaptation）** | 设计对相关性天然鲁棒的统计量，不做显式去相关 | ASKF（Poddar 2016）、小波 CUSUM（Li 2019） |
| **显式解耦（Explicit decoupling）** | 先做去相关预处理，再用标准控制图监测 | TFPW（Desa 2013）、非参数 PCA（Phaladiganon 2013）、Cholesky 类方法（Qiu 2020、Li & Qiu 2020、Xie & Qiu 2022a/b、Xue & Qiu 2021、Wang 2025、Liu 2025、Tian & Qiu 2025） |

**Cholesky 分解类方法**是当前主流，但仍存在三大瓶颈：
- 多数只考虑**正向偏移**，忽略同样关键的**负向偏移**；
- 对**早期微小故障**灵敏度不足，故障特征易被噪声与残差相关性掩盖；
- 机器学习模型与去相关数据的**适配性**有待提升。

### 2.4 本文定位

针对上述空白，作者提出**首个**系统性框架，用于监测多传感器产生的、具有强序列相关性、分布未知的多元数据流，**同时检测正/负两个方向的偏移**，服务于智能制造的实时质量保障。

---

## 三、核心贡献与创新点

1. **方法融合创新**：将 Cholesky 分解、随机森林（RF）、重采样、MEWMA 四项成熟技术**首次整合**为统一的非参数监测框架 C-RF-MEWMA。
2. **RF 角色的重新定义**：传统上 RF 只用于特征提取，本文将 RF 用作**判别式统计量生成器**，直接把 RF 输出的"属于 IC 状态的概率"作为控制图的监测统计量。
3. **双向偏移检测**：显式训练 IC + OC 两类样本，能同时灵敏检测正/负方向的过程偏移。
4. **递归计算**：Cholesky 分解采用**递归更新**形式，避免每个时刻对整块协方差矩阵重复分解，降低在线计算负担。
5. **数据驱动、无分布假设**：完全绕开参数分布假设，适用于复杂工业过程。
6. **真实案例验证**：在 UCI SECOM 半导体数据集上比 G-MCUSUM 提前 **14 个观测点**报警。

---

## 四、方法总体架构（4 步流水线）

```
     ┌──────────────┐   ┌──────────────┐   ┌──────────────┐   ┌──────────────┐
     │   Step 1     │──▶│   Step 2     │──▶│   Step 3     │──▶│   Step 4     │
     │  Cholesky    │   │  MEWMA 序列  │   │  训练 RF     │   │  在线监测    │
     │  去相关+标准化│   │  计算        │   │  分类器      │   │  +报警判断   │
     └──────────────┘   └──────────────┘   └──────────────┘   └──────────────┘
     相关序列 x_i       独立标准化 x*_i    MEWMA 序列 E_i     概率 p(E_i ∈ IC)
                                                              vs. 控制限 L
```

- **Phase I（离线）**：Step 1 → Step 2 → Step 3 → 用 Monte Carlo + 二分法确定控制限 L。
- **Phase II（在线）**：Step 1 → Step 2 → 代入 Step 3 训练好的 RF → 输出概率统计量 → 与 L 比较判警。

---

## 五、关键符号全表

### 5.1 数据与维度符号

| 符号 | 含义 |
| --- | --- |
| $p$ | 过程数据的维度（多传感器变量数） |
| $n$ | 观测样本总量 |
| $\boldsymbol{X}_i$ 或 $\boldsymbol{X}_n$ | 第 $i$（或 $n$）个时刻的 $p$ 维原始观测向量 |
| $\boldsymbol{X}_i^*$ | 去相关并标准化后的 $p$ 维独立向量 |
| $\{\boldsymbol{X}_n\}$ | 序列过程观测集，$n \ge 1$ |
| $\boldsymbol{X}_{\mathrm{IC}}$ | 历史受控（In-Control）数据集 |
| $\boldsymbol{X}_{\mathrm{OC}}$ | 历史失控（Out-of-Control）数据集 |
| $\boldsymbol{x}_i$ | 在线监测阶段的实时观测 |
| $\boldsymbol{e}_{i-1}$ | 由前 $b$ 个中心化观测拼成的长向量 |
| $\boldsymbol{E}_i$ | MEWMA 序列输出（作为 RF 输入） |

### 5.2 统计参数符号

| 符号 | 含义 |
| --- | --- |
| $\hat{\boldsymbol{\mu}}^{(0)}$ | IC 状态的均值向量（$p$ 维） |
| $\hat{\boldsymbol{\mu}}^{(1)}$ | OC 状态的均值向量（$p$ 维） |
| $\boldsymbol{\gamma}(s)$ | 滞后 $s$ 阶的自协方差矩阵，$\boldsymbol{\gamma}(s) = \mathrm{Cov}(\boldsymbol{X}_i, \boldsymbol{X}_{i+s})$ |
| $\hat{\boldsymbol{\gamma}}^{(0)}(s)$ | IC 数据下估计的 $s$ 阶自协方差 |
| $\hat{\boldsymbol{\gamma}}^{(1)}(s)$ | OC 数据下估计的 $s$ 阶自协方差 |
| $\hat{\boldsymbol{\Sigma}}_{i,i}$ | 长向量 $(\boldsymbol{X}_{i-b}, \dots, \boldsymbol{X}_i)$ 的块状协方差矩阵 |
| $\boldsymbol{\sigma}_{i-1}$ | $\hat{\boldsymbol{\Sigma}}_{i,i}$ 的右上块，$\boldsymbol{\sigma}_{i-1} = (\boldsymbol{\gamma}(b)', \dots, \boldsymbol{\gamma}(1)')'$ |
| $\boldsymbol{d}_i$ | Cholesky 分解对角元，$\boldsymbol{d}_i = \boldsymbol{\gamma}(0) - \boldsymbol{\sigma}_{i-1}' \boldsymbol{\Sigma}_{i-1,i-1}^{-1} \boldsymbol{\sigma}_{i-1}$ |
| $\boldsymbol{L}_i$ | Cholesky 下三角矩阵 |
| $\boldsymbol{Q}_i$ | Cholesky 分解结果，$\boldsymbol{Q}_i = \mathrm{diag}(\boldsymbol{d}_{i-b}, \dots, \boldsymbol{d}_i)$ |
| $b_{\max}$ | 相关性时间窗口，超过 $b_{\max}$ 阶滞后认为无相关 |
| $b$ | 当前递归步的截断阶数，$b = \min(i-1, b_{\max})$ |
| $\boldsymbol{I}_{p\times p}$ | $p\times p$ 单位矩阵 |

### 5.3 控制图与机器学习符号

| 符号 | 含义 |
| --- | --- |
| $\lambda$ | MEWMA 平滑因子（$0 < \lambda \le 1$） |
| $M$ | MEWMA 分段长度（每段包含 $M$ 个观测） |
| $\boldsymbol{S}_{m,l}$ | 第 $m$ 段内位置 $l$ 处的 MEWMA 中间统计量 |
| $T$ | 随机森林中决策树总数（$n_{\mathrm{tree}}$） |
| $m_{\mathrm{try}}$ | 每个节点分裂时随机选择的特征数 |
| $w_t^j(\boldsymbol{x})$ | 第 $t$ 棵决策树对样本 $\boldsymbol{x}$ 属于类别 $j$ 的投票（0 或 1） |
| $W(\boldsymbol{x})$ | RF 对 $\boldsymbol{x}$ 的最终分类结果 |
| $p(\boldsymbol{x} \in c_j)$ | $\boldsymbol{x}$ 属于类别 $c_j$ 的概率 |
| $p(\boldsymbol{E}_i \in \mathrm{IC})$ | MEWMA 序列 $\boldsymbol{E}_i$ 属于 IC 状态的概率（本文监测统计量） |
| $L$ | 控制限（阈值） |
| $\mathrm{ARL}_0$ | 平均受控运行长度（In-Control Average Run Length） |
| $\mathrm{ARL}_1$ | 平均失控运行长度（Out-of-Control ARL） |
| $\mathrm{RMI}$ | 相对均值指标（Relative Mean Index），综合评价参数优劣 |
| $N_{\mathrm{MC}}$ | Monte Carlo 仿真次数（本文 = 10000） |
| $K$ | 二分法迭代次数 |
| $C$ | 每次 MC 生成的 IC 数据块长度 |
| $N_{\mathrm{train}}$ | 训练集样本量 |
| $\delta$ | 过程均值偏移幅度 |

---

## 六、公式逐条精讲

> **说明**：本文公式与符号采用 LaTeX 数学排版（与原文 PDF 的印刷形式一致），请在支持数学渲染的 Markdown 阅读器（如 Typora、Obsidian、VS Code Markdown 预览、GitHub）中查看。符号约定：$\hat{\ }$ 表示估计量，粗体 $\boldsymbol{\cdot}$ 表示向量/矩阵，上标 $-1$ 表示矩阵求逆，$-1/2$ 表示矩阵平方根逆（先开方再求逆），$'$ 表示转置。

### 6.1 IC/OC 均值与协方差的矩估计

由于分布未知，无法用极大似然，改用**矩估计（method of moments）**。

**IC 均值向量**：

$$\hat{\boldsymbol{\mu}}^{(0)} = \frac{1}{n_0 - \tau_0} \sum_{i=-n_0+1}^{-\tau_0} \boldsymbol{X}_i$$

**IC 自协方差矩阵（滞后 s 阶）**：

$$\hat{\boldsymbol{\gamma}}^{(0)}(s) = \frac{1}{n_0 - \tau_0 - s} \sum_{i=-n_0+1}^{-\tau_0-s} (\boldsymbol{X}_{i+s} - \hat{\boldsymbol{\mu}}^{(0)})(\boldsymbol{X}_i - \hat{\boldsymbol{\mu}}^{(0)})'$$

> 注：分母 $n_0 - \tau_0 - s$ 是参与求和的样本对数量，保证矩估计的无偏性；求和上限 $-\tau_0-s$ 确保 $i+s$ 不超出 IC 数据段。

**OC 均值向量**：

$$\hat{\boldsymbol{\mu}}^{(1)} = \frac{1}{\tau_0} \sum_{i=-\tau_0+1}^{0} \boldsymbol{X}_i$$

**OC 自协方差矩阵**：

$$\hat{\boldsymbol{\gamma}}^{(1)}(s) = \frac{1}{\tau_0 - s} \sum_{i=-\tau_0+1}^{-s} (\boldsymbol{X}_{i+s} - \hat{\boldsymbol{\mu}}^{(1)})(\boldsymbol{X}_i - \hat{\boldsymbol{\mu}}^{(1)})'$$

**含义解读**：这里 $n_0$ 与 $\tau_0$ 是时间轴上的分界索引（Phase I 数据段），历史数据被划分为 IC 段 $[-n_0+1,\, -\tau_0]$ 与 OC 段 $[-\tau_0+1,\, 0]$。平稳性假设保证 $\boldsymbol{\gamma}(s)$ 只与滞后阶 $s$ 有关，与起点 $i$ 无关。

---

### 6.2 公式 (1)：块状协方差矩阵的递归结构

对长向量 $(\boldsymbol{X}_{i-b}, \boldsymbol{X}_{i-b+1}, \dots, \boldsymbol{X}_i)$ 的协方差矩阵：

$$\hat{\boldsymbol{\Sigma}}_{i,i} = \begin{pmatrix} \hat{\boldsymbol{\Sigma}}_{i-1,i-1} & \boldsymbol{\sigma}_{i-1} \\ \boldsymbol{\sigma}_{i-1}' & \hat{\boldsymbol{\gamma}}^{(0)}(0) \end{pmatrix}$$

其中 $\boldsymbol{\sigma}_{i-1} = \left( \hat{\boldsymbol{\gamma}}^{(0)}(b)', \hat{\boldsymbol{\gamma}}^{(0)}(b-1)', \dots, \hat{\boldsymbol{\gamma}}^{(0)}(1)' \right)'$

约束条件：$0 \le b \le b_{\max}$，$-n_0+1 \le i \le -\tau_0$。

**含义解读**：$\hat{\boldsymbol{\Sigma}}_{i,i}$ 采用**分块矩阵**表达，左上块是前一步已计算好的 $\hat{\boldsymbol{\Sigma}}_{i-1,i-1}$，右下块是当前时刻的方差 $\hat{\boldsymbol{\gamma}}^{(0)}(0)$，右上/左下块是历史与当前的互协方差 $\boldsymbol{\sigma}_{i-1}$。这一递归结构是**在线计算**的关键——每来一个新观测只需更新一次块矩阵，无需重算整块。

---

### 6.3 公式 (2)：Cholesky 去相关与标准化

**初始步（$i = -n_0+1$，无历史）**：

$$\boldsymbol{X}_i^* = \hat{\boldsymbol{\gamma}}^{(0)}(0)^{-1/2} \left( \boldsymbol{X}_i - \hat{\boldsymbol{\mu}}^{(0)} \right)$$

**递归步（$i > -n_0+1$）**：

$$\boldsymbol{X}_i^* = \boldsymbol{d}_i^{-1/2} \left[ -\boldsymbol{\sigma}_{i-1}' \hat{\boldsymbol{\Sigma}}_{i-1,i-1}^{-1} \boldsymbol{e}_{i-1} + \left( \boldsymbol{X}_i - \hat{\boldsymbol{\mu}}^{(0)} \right) \right]$$

其中：

$$\boldsymbol{e}_{i-1} = \left( (\boldsymbol{X}_{i-b} - \hat{\boldsymbol{\mu}}^{(0)})', (\boldsymbol{X}_{i-b+1} - \hat{\boldsymbol{\mu}}^{(0)})', \dots, (\boldsymbol{X}_{i-1} - \hat{\boldsymbol{\mu}}^{(0)})' \right)'$$

$$\boldsymbol{d}_i = \hat{\boldsymbol{\gamma}}^{(0)}(0) - \boldsymbol{\sigma}_{i-1}' \hat{\boldsymbol{\Sigma}}_{i-1,i-1}^{-1} \boldsymbol{\sigma}_{i-1}$$

**递归更新规则**：
- 当 $i = -n_0+1$ 时，$b = 0$；
- 否则 $b = \min(b+1, b_{\max})$，$i = i+1$；
- 重复直至 $i = -\tau_0$。

**含义解读**（本文最核心的公式）：

1. **去中心化**：$\boldsymbol{X}_i - \hat{\boldsymbol{\mu}}^{(0)}$ 消除均值；
2. **回归消除相关**：$-\boldsymbol{\sigma}_{i-1}' \hat{\boldsymbol{\Sigma}}_{i-1,i-1}^{-1} \boldsymbol{e}_{i-1}$ 相当于用历史 $b$ 个观测对当前观测做**最优线性预测**，然后取**残差**，残差与历史独立；
3. **标准化**：乘以 $\boldsymbol{d}_i^{-1/2}$（残差协方差的平方根逆），使新变量协方差恰为单位阵 $\boldsymbol{I}_p$；
4. **结果**：$\{\boldsymbol{X}_i^*\}$ 是**近似 i.i.d.**（独立同分布）序列，协方差为单位阵，可直接送入标准非参数控制图。

**Cholesky 分解视角**（附录推导）：

$$\boldsymbol{L}_i \hat{\boldsymbol{\Sigma}}_{i,i} \boldsymbol{L}_i' = \boldsymbol{Q}_i$$

$$\boldsymbol{L}_i = \begin{pmatrix} \boldsymbol{L}_{i-1} & \boldsymbol{0} \\ -\boldsymbol{\sigma}_{i-1}' \hat{\boldsymbol{\Sigma}}_{i-1,i-1}^{-1} & \boldsymbol{I}_{p\times p} \end{pmatrix}$$

$$\boldsymbol{Q}_i = \mathrm{diag}(\boldsymbol{d}_{i-b}, \boldsymbol{d}_{i-b+1}, \dots, \boldsymbol{d}_i)$$

即 $\boldsymbol{L}_i$ 是把相关向量 $\boldsymbol{e}_i$ 线性变换为不相关向量 $\boldsymbol{\xi}_i = \boldsymbol{L}_i \boldsymbol{e}_i$ 的下三角矩阵；$\boldsymbol{Q}_i$ 是变换后的对角协方差。最后一步 $\boldsymbol{d}_i^{-1/2}$ 把 $\boldsymbol{\xi}_i$ 归一化为单位协方差，得到 $\boldsymbol{X}_i^*$。

**实际数值注意事项**：
- $\hat{\boldsymbol{\gamma}}^{(0)}(0)^{-1/2}$ 与 $\boldsymbol{d}_i^{-1/2}$ 在样本量小时可能不存在解析解 → 用**伪逆**或**加大样本量**（论文经验：样本量 $\ge 100$ 可避免）；
- $\hat{\boldsymbol{\Sigma}}_{i,i}$ 未必正定 → 用 **Higham (1988) 矩阵修正法**投影到最近的正定矩阵。

---

### 6.4 公式 (3)(4)：MEWMA 分段序列

**段内递推（每段 M 个观测）**：

$$\boldsymbol{S}_{m,l} = \lambda \boldsymbol{x}^*_{m+l-1} + (1-\lambda)\, \boldsymbol{S}_{m,l-1}, \quad m \ge 1,\ 1 \le l \le M$$

**输出序列 E_i**：

$$\boldsymbol{E}_i = \begin{cases} \boldsymbol{S}_{1,i}, & 1 \le i \le M \\ \boldsymbol{S}_{i-M+1,M}, & i > M \end{cases}$$

**初值**：$\boldsymbol{S}_{m,0}$ 定义为历史 IC 样本标准化后的均值向量。

**含义解读**：

- **MEWMA（多元指数加权移动平均）**：$\lambda$ 是权重系数，$\lambda$ 越大越重视新观测（对大偏移敏感），$\lambda$ 越小越重视历史信息（对小偏移敏感）；
- **分段设计**：普通 MEWMA 会随时间收敛到稳态值，丧失检测能力。本文将序列**每 M 个观测切成一段**，段末输出 $\boldsymbol{S}_{m,M}$ 作为该段的特征向量，段与段之间**重置初值**，从而保持统计量的动态性；
- **输出维度**：E_i 是 p 维向量（因为 S 是 p 维），作为 RF 的输入特征。

**参数选择依据**：
- $\lambda \in [0.05, 0.2]$，论文最终选 $\lambda = 0.2$（在 RMI 指标下综合表现最好）；
- $M \in [15, 30]$，论文最终选 $M = 15$（多数场景下 RMI 最低）。

---

### 6.5 公式 (5)(6)(7)：随机森林判别式统计量

**单棵决策树投票**：

$$w_t^j(\boldsymbol{x}) \in \{0, 1\}$$

若第 $t$ 棵树把 $\boldsymbol{x}$ 分到类别 $c_j$，则 $w_t^j(\boldsymbol{x}) = 1$；否则 $w_t^j(\boldsymbol{x}) = 0$

**集成分类结果（多数投票）**：

$$W(\boldsymbol{x}) = c_{\arg\max_j \sum_{t=1}^{T} w_t^j(\boldsymbol{x})}$$

**类别概率（投票占比）**：

$$p(\boldsymbol{x} \in c_j) = \frac{\sum_{t=1}^{T} w_t^j(\boldsymbol{x})}{T}$$

**IC 概率（本文监测统计量）**：

$$p(\boldsymbol{x} \in \mathrm{IC}) = \frac{\sum_{t=1}^{T} w_t^1(\boldsymbol{x})}{T}$$

其中 $c_1 = \mathrm{IC}$，$c_2 = \mathrm{OC}$。

**含义解读**：

- RF 通过 **Bagging（自助重采样）+ 随机特征选择**训练 T 棵决策树；
- 传统 RF 只输出"硬分类"结果，本文改为**输出概率**，让控制图得到**连续的监测统计量**；
- **统计量意义**：$p(\boldsymbol{E}_i \in \mathrm{IC})$ 越接近 1 → 越可信为受控；越接近 0 → 越可信为失控；
- 这一设计**保留了传统控制图的统计严谨性**：仍用固定 $\mathrm{ARL}_0$ 定 L，仍用 $\mathrm{ARL}_1$ 评价性能。

**RF 的三大优势**（为何选 RF 而不是 SVM/NN）：
1. **非参数**：无需分布假设、无需显式特征选择；
2. **鲁棒**：对工业数据常见噪声与异常值不敏感；
3. **泛化能力强**：集成学习显著降低过拟合风险。

---

### 6.6 公式 (8)：报警判据

若 $p(\boldsymbol{E}_i \in \mathrm{IC}) \le L$，触发 OC 报警；否则过程视为正常运行。

**含义解读**：控制限 $L \in [0, 1]$，通过 **Monte Carlo 仿真 + 二分法**确定，使得在 IC 状态下的平均运行长度 ARL(L) 恰好等于预设的 $\mathrm{ARL}_0$（本文取 $\mathrm{ARL}_0$ = 200）。

**单调性保证**：p 越小越倾向报警，所以 $\mathrm{ARL}_0$ 是 L 的**单调递减函数**——L 越大，报警越严，$\mathrm{ARL}_0$ 越大。这个单调性保证了二分法收敛。

---

### 6.7 公式 (9)：相对均值指标 RMI

用于综合评价一组参数在所有偏移场景下的整体表现：

$$\mathrm{RMI} = \frac{1}{n} \sum_{i=1}^{n} \frac{\mathrm{ARL}_{1i}(R) - \min(\mathrm{ARL}_{1i})}{\min(\mathrm{ARL}_{1i})}$$

其中：
- $n$：偏移场景总数（本文 $n = 8$，对应 $\delta = \pm 1, \pm 0.75, \pm 0.5, \pm 0.25$）；
- $\mathrm{ARL}_{1i}(R)$：在参数 $R$ 下、第 $i$ 个场景的失控 ARL；
- $\min(\mathrm{ARL}_{1i})$：所有候选参数在第 $i$ 个场景下的**最优**（最小）$\mathrm{ARL}_1$。

**含义解读**：RMI 衡量"当前参数相对最优参数的平均相对差距"。RMI 越小 → 参数越优。相比单看某个场景的 $\mathrm{ARL}_1$，RMI 能同时惩罚"某些场景表现差"的参数组合，是**多场景鲁棒参数选择**的标准指标（源自 Sun et al., 2024）。

---

## 七、控制限 L 的确定：Monte Carlo + 二分法

### 7.1 算法流程（对应 Algorithm 2 Part 3）

```
输入：目标 ARL₀、RF 模型、μ_ic、Σ、b_max、λ、M、N_MC=10000、精度 ε、最大迭代次数
初始化：h_l = 1（上界）、h_u = 0（下界）、L = 0.5、iteration = 0

while iteration < max_iteration:
    Sum_RL = 0
    for i = 1 → N_MC:
        ① 从 Mvrnorm(μ_ic, Σ) 生成长度为 C 的 IC 参考数据 X_L
        ② Z_IC ← Cholesky 去相关(X_L, b_max)
        ③ 计算 S_i，再算 E_i ← MEWMA(Z_IC, λ, M)
        ④ p_i ← RF 模型(E_i, class = "IC")
        ⑤ 若 p_i > L，运行长度 RL_i = i（首次报警点）
           若始终未报警，RL_i = |X_L|
        ⑥ Sum_RL += RL_i
    
    实际 ARL₀ = Sum_RL / N_MC
    
    if |实际 ARL₀ − 目标 ARL₀| < ε:
        break，返回 L
    
    if 实际 ARL₀ > 目标 ARL₀:  h_l = (h_l + h_u)/2   （L 偏小，需增大）
    else:                       h_u = (h_l + h_u)/2   （L 偏大，需减小）
    
    L = (h_l + h_u)/2
    iteration += 1

return L
```

### 7.2 关键设计

- **$N_{\mathrm{MC}}$ = 10000**：保证 $\mathrm{ARL}_0$ 估计的统计精度；
- **二分法收敛性**：因 $\mathrm{ARL}_0$ 关于 L 单调递减，且 $L \in [0, 1]$，二分法必然收敛；
- **$\mathrm{ARL}_0$ = 200**：工业界常用值，意味着受控状态下平均每 200 个观测点误报一次。

---

## 八、计算复杂度分析

### 8.1 离线阶段

| 环节 | 复杂度 |
| --- | --- |
| Cholesky 去相关 | $O((p \cdot b_{\max})^3)$ |
| RF 训练 | $O(n_{\mathrm{tree}} \cdot p \cdot N_{\mathrm{train}} \cdot \log N_{\mathrm{train}})$ |
| 控制限 L 搜索 | $O(K \cdot N_{\mathrm{MC}} \cdot C \cdot ((p \cdot b_{\max})^3 + p \cdot M + n_{\mathrm{tree}} \cdot p \cdot \log N_{\mathrm{train}}))$ |

**主导项**：$O((p \cdot b_{\max})^3)$ —— 由 Cholesky 分解的立方复杂度决定。

### 8.2 在线阶段

每个新观测需要：
- Cholesky 去相关：$O((p \cdot b_{\max})^3)$
- MEWMA 更新：$O(p \cdot M)$
- RF 预测：$O(n_{\mathrm{tree}} \cdot p \cdot \log N_{\mathrm{train}})$

**主导项**：仍是 $O((p \cdot b_{\max})^3)$。

### 8.3 可扩展性结论

整个 C-RF-MEWMA 方案的可扩展性主要由 **Cholesky 去相关的立方复杂度**决定，即 p（维度）与 $b_{\max}$（相关窗口）是主要瓶颈。这也是论文在结论中承认的**未来工作方向**——高维场景下逆协方差矩阵的计算与数值稳定性。

---

## 九、仿真研究设计

### 9.1 四种测试场景（Cases I–IV）

| 场景 | 数据生成模型 | 特征 |
| --- | --- | --- |
| **Case I** | $\boldsymbol{X}_n \sim \mathrm{i.i.d.}\ N_3(\boldsymbol{0}, \boldsymbol{I}_{3\times 3})$ | 独立多元正态，无序列相关 |
| **Case II** | $X_{n1}$: AR(1) 系数 0.1；$X_{n2} = X_{n1} + 0.1\xi$（$\xi$ 为标准化 $\chi^2_3$）；$X_{n3}$: AR(1) 系数 0.2 | 弱序列相关 + 变量间弱相关 |
| **Case III** | $X_{n1}$: AR(1) 系数 0.5；$X_{n2} = X_{n1} + 0.1\xi$；$X_{n3}$: AR(1) 系数 0.5 | **强序列相关** + 变量间弱相关 |
| **Case IV** | $\boldsymbol{X}_n = (1, 2, 1)' + \boldsymbol{A}\boldsymbol{X}_{n-1} + \boldsymbol{\varepsilon}_n$，$\boldsymbol{A} = \mathrm{diag}(0.5, 0.7, 0.2)$，$\boldsymbol{B}$ = 单位对角 + 0.2 非对角 | 序列相关 + 变量间强相关 |
| **Case IV\*** | 10 维扩展：$\boldsymbol{A}^* = \mathrm{diag}(0.5, 0.7, 0.2, 0.5, 0.7, 0.2, 0.5, 0.7, 0.2, 0.5)$ | 高维验证 |

### 9.2 对比方法

| 方法 | 来源 | 类型 |
| --- | --- | --- |
| **G-MCUSUM** | Xue & Qiu (2021) | 自启动非参数多元 CUSUM |
| **DKE-CUSUM** | Wang et al. (2025) | 核密度估计 + KNN + CUSUM |
| **NP-MCUSUM** | Qiu (2008) | 对数线性建模的非参数 MCUSUM |

### 9.3 关键仿真结论

**IC 性能（$\mathrm{ARL}_0$ 实测值，样本量 500，目标 $\mathrm{ARL}_0$ = 200）**：

| 场景 | C-RF-MEWMA | G-MCUSUM | DKE-CUSUM | NP-MCUSUM |
| --- | --- | --- | --- | --- |
| Case I | 208 | 204 | 204 | **124** |
| Case II | 203 | 196 | 203 | **68** |
| Case III | 198 | 195 | 195 | **42** |
| Case IV | 205 | 195 | 195 | **37** |

→ NP-MCUSUM 严重偏离目标，因无自启动、无机器学习；C-RF-MEWMA、G-MCUSUM、DKE-CUSUM 均可靠。

**样本量对 C-RF-MEWMA IC 性能的影响（p=3）**：

| 样本量 | Case I | Case II | Case III | Case IV |
| --- | --- | --- | --- | --- |
| 100 | 254.34 | 160.61 | 245.85 | 230.41 |
| 200 | 233.06 | 185.72 | 214.44 | 216.64 |
| 300 | 220.19 | 191.65 | 210.42 | 207.76 |
| **500** | **208.14** | **202.94** | **197.90** | **205.16** |
| 1000 | 201.95 | 200.22 | 199.95 | 200.91 |

→ 样本量 $\ge 500$ 才能达到可靠 IC 性能；样本量 100 时偏差 $> 10\%$。

**高维（$p=10$）验证**：Case IV\* 下样本量 $\ge 1000$ 才可靠（500 时 $\mathrm{ARL}_0 = 211.96$，1000 时 $= 207.13$，2000 时 $= 198.88$，3000 时 $= 201.49$）。

**OC 性能（$\mathrm{ARL}_1$）**：
- Case I（独立数据）：G-MCUSUM、DKE-CUSUM、NP-MCUSUM 相近，G-MCUSUM 略优；
- Case II–IV（相关数据）：**C-RF-MEWMA 最优**，尤其 Case IV 显著领先；
- 训练用 $\pm 0.25/0.5/0.75/1$，测试用 $\pm 0.2/0.4/0.6/0.8$（未知偏移），C-RF-MEWMA 仍表现良好，说明**泛化能力强**。

**$b_{\max}$ 敏感性分析（Case II）**：

| $\delta$ | $b_{\max}$=10 | $b_{\max}$=15 | $b_{\max}$=20 | $b_{\max}$=25 | $b_{\max}$=30 |
| --- | --- | --- | --- | --- | --- |
| $\pm 1$ | 4.35/4.02 | 4.33/4.02 | **4.18/4.39** | 4.40/4.18 | 4.51/4.29 |
| $\pm 0.25$ | 32.06/23.12 | 27.24/23.10 | **24.46/24.46** | 24.47/26.45 | 30.86/29.21 |

→ $b_{\max} \in [10, 20]$ 表现最好；过大反而引入估计噪声，性能下降。论文选 $b_{\max} = 20$。

---

## 十、真实案例：SECOM 半导体制造数据

### 10.1 数据来源

- **UCI Machine Learning Repository**（archive.ics.uci.edu/dataset/179/secom）
- 2008 年 7–10 月某半导体制造过程的自动化管理计算机系统采集
- 与 Xue & Qiu (2021) 保持一致：选 3 个质量变量 $V_1$、$V_2$、$V_3$，共 500 个观测

### 10.2 数据划分与预处理

- **IC 数据**：前 400 个观测（分布稳定）
- **OC 数据**：后 100 个观测
- **正态 Q-Q 图**（Fig. 9）：显著偏离正态分布 → 验证非参数方法的必要性
- **ACF 图**（Fig. 10）：$V_2$、$V_3$ 存在显著自相关
- **散点图矩阵**（Fig. 11）：变量间存在相关性
- **置换检验 p 值**：$(V_1, V_2) = 0.4149$、$(V_1, V_3) = 0.1912$、$(V_2, V_3) = 0.0042$ → 仅 $V_2$ 与 $V_3$ 相关性显著

### 10.3 实验设置

- 因样本量不足以训练 ML 模型，对 IC 与 OC 数据**重采样扩充到 500**
- G-MCUSUM 容差常数 k = 0.01
- 两图 $\mathrm{ARL}_0$ 均设为 200

### 10.4 监测结果（Fig. 12）

| 方法 | 首次报警位置 |
| --- | --- |
| **C-RF-MEWMA** | 第 **9** 个观测点 |
| G-MCUSUM | 第 **23** 个观测点 |

→ C-RF-MEWMA **提前 14 个观测点**报警，验证了对真实半导体过程分布变化的检测有效性。

---

## 十一、Algorithm 1 与 Algorithm 2 对照

### Algorithm 1（概念流程）

| Step | 目标 | 输入 | 方法 | 输出 |
| --- | --- | --- | --- | --- |
| 1 | 去相关 | p 维历史/在线序列相关数据 | Cholesky 分解 | p 维标准化独立数据 |
| 2 | 数据变换 | 去相关后数据 | MEWMA 过程 | p 维 MEWMA 序列 |
| 3 | 建 RF 分类器 | 历史 MEWMA 序列 | RF 模型 | 样本属于 IC 的概率 |
| 4 | 在线监测 | 在线 MEWMA 序列 + 控制限 L | 控制图判据 | 若 $p(\boldsymbol{E}_i \in \mathrm{IC}) \le L$ → 报警，否则正常 |

### Algorithm 2（可执行伪代码）

分为三部分：**Part 1 数据生成与去相关**、**Part 2 训练 RF**、**Part 3 二分法确定控制限**。详见第七章。

**RF 训练细节**（R 语言 randomForest 包）：
```r
randomForest(Class ~ ., data = Train_dataset, ntree = 300,
             importance = TRUE, type = "classification")
```
$m_{\mathrm{try}}$ 默认为特征维度的平方根 $\sqrt{p}$，$n_{\mathrm{tree}} = 300$。

**数据平衡处理**：8 种 OC 偏移 $\times$ 500 样本 $= 4000$ OC 样本，远多于 IC 的 500 → 对 IC 数据**重采样**至 4000 以保证类别平衡。

---

## 十二、方法论对比总览（Table 1 提炼）

| 方法 | 维度 | 分布 | 机器学习 | 偏移方向 | 序列相关 |
| --- | --- | --- | --- | --- | --- |
| NEW (Qiu et al., 2020) | 一元 | 参数 | 否 | 双向 | 是 |
| G-CUSUM (Li & Qiu, 2020) | 一元 | 参数 | 否 | 双向 | 是 |
| NEW (Xie & Qiu, 2022a) | 多元 | 非参数 | 否 | 双向 | 是 |
| G-MCUSUM (Xue & Qiu, 2021) | 多元 | 非参数 | 否 | 双向 | 是 |
| MEWMA-SS / TSL (Qiu & Xie, 2022) | 多元 | 非参数 | 否 | 双向 | 是 |
| AC-D / RTC-D / DSVM-D / KNN-D (Xie & Qiu, 2022b) | 多元 | 非参数 | **是** | 双向 | 是 |
| EWMA-P / EWMA-Q (Xie & Qiu, 2024) | 多元 | 非参数 | 否 | 双向 | 是 |
| DKE-CUSUM (Wang et al., 2025) | 多元 | 非参数 | **是** | 单向 | 是 |
| U-MSEWMA (Liu et al., 2025) | 多元 | 非参数 | **是** | 双向 | 是 |
| EWMA (Tian & Qiu, 2025) | 多元 | 非参数 | 否 | 双向 | 是 |
| **C-RF-MEWMA（本文）** | **多元** | **非参数** | **是（RF 生成统计量）** | **双向** | **是** |

**本文独特定位**：在"多元 + 非参数 + 机器学习 + 双向偏移 + 序列相关"五个维度上**全部满足**，且首次将 RF 用作**判别式统计量生成器**而非单纯分类器。

---

## 十三、结论与未来方向

### 13.1 主要结论

1. C-RF-MEWMA 在**无分布假设**下实现了对多元序列相关数据的有效监测；
2. Cholesky 分解将相关序列转化为独立表示；
3. RF-MEWMA 组合对去相关数据实现灵敏监测；
4. 仿真证明：在 IC 与 OC 性能上均优于 G-MCUSUM、DKE-CUSUM、NP-MCUSUM，尤其在**同时存在时间相关与变量间相关**时优势显著；
5. 真实半导体案例验证了工程实用性。

### 13.2 未来工作

- **高维数值稳定性**：去相关过程涉及逆协方差矩阵计算，维度高时计算量大且数值不稳定，需专门算法改进；
- 作者团队正致力于此问题的后续研究。

---

## 十四、术语对照表（英文 → 中文）

| 英文 | 中文 |
| --- | --- |
| Multivariate Statistical Process Control (MSPC) | 多元统计过程控制 |
| Nonparametric control chart | 非参数控制图 |
| Serially correlated data | 序列相关数据 |
| In-Control (IC) / Out-of-Control (OC) | 受控 / 失控 |
| Average Run Length (ARL) | 平均运行长度 |
| Run Length (RL) | 运行长度（首次报警前的观测数） |
| Multivariate EWMA (MEWMA) | 多元指数加权移动平均 |
| Multivariate CUSUM (MCUSUM) | 多元累积和 |
| Cholesky decomposition | 乔莱斯基分解（下三角分解） |
| Random Forest (RF) | 随机森林 |
| Decorrelation | 去相关 |
| Recursive computation | 递归计算 |
| Monte Carlo simulation | 蒙特卡洛仿真 |
| Bisection method | 二分法 |
| Moment estimation | 矩估计 |
| Maximum likelihood estimation | 极大似然估计 |
| Weak / covariance stationarity | 弱平稳 / 协方差平稳 |
| Autocorrelation function (ACF) | 自相关函数 |
| Q-Q plot | 分位数-分位数图 |
| Permutation test | 置换检验 |
| Resampling | 重采样 |
| Bagging | 自助聚合 |
| Decision tree | 决策树 |
| Smoothing factor | 平滑因子 |
| Tolerance constant | 容差常数 |
| Self-starting | 自启动 |
| Masking effect | 掩蔽效应（去相关削弱偏移信号） |
| Relative Mean Index (RMI) | 相对均值指标 |
| Pseudo-inverse | 伪逆 |
| Positive definite matrix | 正定矩阵 |
| Identity matrix | 单位矩阵 |
| SECOM | 半导体制造过程数据集（UCI） |

---

## 十五、一句话总结

**本文提出的 C-RF-MEWMA 控制图，通过"Cholesky 递归去相关 → 分段 MEWMA 特征化 → 随机森林概率判别 → Monte Carlo 二分校准控制限"四步流水线，在无分布假设下实现了对多元序列相关过程的双向偏移灵敏监测，仿真与半导体真实案例均验证了其对 G-MCUSUM、DKE-CUSUM、NP-MCUSUM 的优越性，尤其适用于工业 5.0 多传感器智能制造场景。**

---

## 附录 A：公式速查表

| 编号 | 公式 | 用途 |
| --- | --- | --- |
| — | $\hat{\boldsymbol{\mu}}^{(0)} = \frac{1}{n_0-\tau_0}\sum_{i=-n_0+1}^{-\tau_0}\boldsymbol{X}_i$ | IC 均值矩估计 |
| — | $\hat{\boldsymbol{\gamma}}^{(0)}(s) = \frac{1}{n_0-\tau_0-s}\sum_{i=-n_0+1}^{-\tau_0-s}(\boldsymbol{X}_{i+s}-\hat{\boldsymbol{\mu}}^{(0)})(\boldsymbol{X}_i-\hat{\boldsymbol{\mu}}^{(0)})'$ | IC 自协方差矩估计 |
| (1) | $\hat{\boldsymbol{\Sigma}}_{i,i} = \begin{pmatrix}\hat{\boldsymbol{\Sigma}}_{i-1,i-1} & \boldsymbol{\sigma}_{i-1}\\ \boldsymbol{\sigma}_{i-1}' & \hat{\boldsymbol{\gamma}}^{(0)}(0)\end{pmatrix}$ | 块状协方差递归 |
| (2) | $\boldsymbol{X}_i^* = \boldsymbol{d}_i^{-1/2}\left[-\boldsymbol{\sigma}_{i-1}'\hat{\boldsymbol{\Sigma}}_{i-1,i-1}^{-1}\boldsymbol{e}_{i-1} + (\boldsymbol{X}_i-\hat{\boldsymbol{\mu}}^{(0)})\right]$ | Cholesky 去相关+标准化 |
| (3) | $\boldsymbol{S}_{m,l} = \lambda\boldsymbol{x}^*_{m+l-1} + (1-\lambda)\boldsymbol{S}_{m,l-1}$ | MEWMA 段内递推 |
| (4) | $\boldsymbol{E}_i = \boldsymbol{S}_{1,i}\ (i\le M)$ 或 $\boldsymbol{S}_{i-M+1,M}\ (i>M)$ | MEWMA 输出序列 |
| (5) | $W(\boldsymbol{x}) = c_{\arg\max_j \sum_{t=1}^{T} w_t^j(\boldsymbol{x})}$ | RF 多数投票分类 |
| (6) | $p(\boldsymbol{x}\in c_j) = \frac{1}{T}\sum_{t=1}^{T} w_t^j(\boldsymbol{x})$ | RF 类别概率 |
| (7) | $p(\boldsymbol{x}\in\mathrm{IC}) = \frac{1}{T}\sum_{t=1}^{T} w_t^1(\boldsymbol{x})$ | IC 概率（监测统计量） |
| (8) | $p(\boldsymbol{E}_i\in\mathrm{IC}) \le L$ | 判警准则 |
| (9) | $\mathrm{RMI} = \frac{1}{n}\sum_{i=1}^{n}\frac{\mathrm{ARL}_{1i}(R)-\min\mathrm{ARL}_{1i}}{\min\mathrm{ARL}_{1i}}$ | 参数综合评价指标 |

## 附录 B：推荐参数配置

| 参数 | 推荐值 | 依据 |
| --- | --- | --- |
| $\lambda$（MEWMA 平滑因子） | 0.2 | RMI 在 $\lambda \in [0.05, 0.2]$ 表现稳健，0.2 综合最优 |
| $M$（MEWMA 段长） | 15 | 多数场景 RMI 最低（Chen et al. 2016 建议 15–30） |
| $b_{\max}$（相关窗口） | 20 | 敏感性分析显示 10–20 最佳，过大引入噪声 |
| $n_{\mathrm{tree}}$（RF 树数） | 300 | randomForest 包默认，兼顾精度与效率 |
| $m_{\mathrm{try}}$（分裂特征数） | $\sqrt{p}$ | randomForest 包默认；小偏移可设为 $p$ |
| $\mathrm{ARL}_0$（目标受控 ARL） | 200 | 工业界常用基准 |
| $N_{\mathrm{MC}}$（MC 仿真次数） | 10000 | 保证 $\mathrm{ARL}_0$ 估计精度 |
| 样本量（$p=3$） | $\ge 500$ | $<500$ 时 $\mathrm{ARL}_0$ 偏差 $>10\%$ |
| 样本量（$p=10$） | $\ge 1000$ | 高维需更大样本 |

---

*文档生成时间：2026-09-21*
*解析对象：1-s2.0-S0360835226002160-main.pdf（20 页）*
