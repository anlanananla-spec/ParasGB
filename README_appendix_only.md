# ParasGB Appendix README

> 这个 README **仅整理论文附录内容**，不包含正文摘要、方法主线或主实验表述。
> 内容按 GitHub 仓库首页的阅读习惯重组，覆盖附录 A–H。

---

## 目录

- [A. User Guide](#a-user-guide)
- [B. Dataset Details](#b-dataset-details)
- [C. Experiment Details](#c-experiment-details)
- [D. Additional Results](#d-additional-results)
- [E. Limitations](#e-limitations)
- [F. Future Directions](#f-future-directions)
- [G. Analog Topology-To-Graph](#g-analog-topology-to-graph)
- [H. Algorithms](#h-algorithms)

---

## A. User Guide

### A.1 Standardized Evaluation Protocol

ParasGB 将数据集与 **PyTorch Geometric (PyG)** 深度集成，目标是让研究者只用少量代码即可完成：

- 数据下载
- 图数据预处理
- 任务级数据加载
- 标准化评测

为适配超大规模电路图，工具链提供：

- `NeighborLoader`：节点级任务
- `LinkNeighborLoader`：边级任务

两类 loader 都支持首次使用时缓存预处理图数据，以减少重复计算开销、提升实验复现性，并避免由于预处理逻辑差异带来的实验偏差。

此外，ParasGB 提供统一的 `Evaluator` 模块，参考 OGB 风格自动输出标准指标：

- 分类任务：Accuracy / F1
- 回归任务：MAE / R²

数据集对象还会暴露：

- train/test split
- 节点数、边数
- 标签分布等元信息

这样可以更透明地比较不同模型在不同任务上的真实表现。

### A.2 ParasGB Usage

ParasGB 的核心目标是**降低寄生参数学习研究门槛**。调用方式尽量贴近 PyG，研究者只需指定：

- 数据集名称
- 任务层级（node / edge）
- 任务类型（classification / regression）

系统即可自动完成原始文件下载和特征预处理。

对于数千万节点规模的 SRAM 图，工具提供面向受限显存场景的子图采样能力，保证训练效率和稳定性。

### 任务名称速查

#### SRAM

- `cg_regr`：节点级地电容回归
- `cg_class`：节点级地电容分类
- `cc_regr`：边级耦合电容回归
- `cc_class`：边级耦合电容分类
- `r_class`：边级有效电阻分类

#### Analog

- `cg_regr`：节点级地电容回归
- `cg_class`：节点级地电容分类
- `r_regr`：边级有效电阻回归
- `r_class`：边级有效电阻分类

### 最小使用示例

```python
from parasgb import RCDataset, Evaluator

# 1. Load dataset with predefined train/test splits
# dataset_name: 'sram' or 'analog'
# task_level: 'node' or 'edge'
# task_type: 'regression' or 'classification'
dataset = RCDataset(
    dataset_name='sram',
    root='data/',
    task_level='node',
    task_type='regression'
)

# 2. Create DataLoader
train_loader = dataset.get_dataloader(
    split='train',
    batch_size=32,
    shuffle=True
)

# 3. Define model and optimizer
model = MyModel()
criterion = Loss()
optimizer = Optimizer(model.parameters())

# 4. Train
for epoch in range(E):
    for batch in train_loader:
        y_pred = model(batch.node_attr, batch.edge_index)
        loss = criterion(y_pred.squeeze(), batch.y[:, 0])
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()

# 5. Create evaluator
# Example: SRAM node-level ground capacitance regression
evaluator = Evaluator(
    dataset_name='sram',
    task='cg_regr'
)

# 6. Evaluate
metrics = evaluator.evaluate(y_pred, y_true)
print(f"MAE = {metrics['mae']:.4f}")
```

---

## B. Dataset Details

### B.1 SRAM Features

SRAM 子集采用**统计聚合型特征**，重点描述全局拓扑与器件分布，而不是保留过细的局部器件细节。这样做是为了在超大图场景中控制特征维度，同时仍保留对物理结构有意义的统计信息。

#### Device 特征

- `Mmos`：晶体管倍数
- `L` / `W`：晶体管长度 / 宽度
- `Mres` / `Lres` / `Wres`：电阻相关统计
- `Mcap` / `Lr` / `Nr`：电容相关统计
- `Np`：器件端口数
- `T`：器件类型编码

#### Net 特征

- 连接晶体管数量：`Nmos`
- 连接 gate / source-drain / bulk 数量：`Ng`, `Nsd`, `Nb`
- 晶体管总宽度 / 总长度：`Wtot`, `Ltot`
- 电容数量与聚合尺寸：`Ncap`, `Lrtot`, `Nrtot`
- 电阻数量与聚合尺寸：`Nres`, `Wtot,res`, `Ltot,res`
- 端口数：`Nport`

#### Pin 特征

- MOS 的引脚类型（G / D / S / B）

### B.2 Datasets Introduction

ParasGB 的附录将数据集分为两大类：

#### 1) SRAM 子集

SRAM 数据的核心特点是：

- 图规模极大
- 拓扑重复度高
- 布线密集
- 耦合电容密度高
- 适合验证模型的**可扩展性**与**计算效率**

主要子集包括：

- `ssram`：基础 SRAM，用于验证模型是否能捕捉规则拓扑
- `digtime`：数字时序相关逻辑模块
- `timing ctrl`：存储器内部时序控制模块，拓扑更复杂
- `sandwich`：堆叠式高性能存储架构，耦合密度极高
- `ultra8t`：面向亚阈值低功耗优化的 8T SRAM
- `array 128 32 8t`：数据集中最大的阵列之一，用于极限压力测试

#### 2) Analog 子集

Analog 数据规模更小，但器件级物理描述更加精细，包含：

- 晶体管宽度 `W`
- 晶体管长度 `L`
- 与隔离区边缘相关的 LDE 信息
- 商业 PEX 工具提取的寄生参数

这类电路对噪声和寄生误差高度敏感，因此更适合考察模型对**细粒度物理行为**的建模能力。

附录中逐一列出了 20 个 analog case 的来源与功能简介，覆盖：

- LVDS
- OP（运放）
- BGR（带隙基准）
- LDO（低压差稳压器）

### B.3 Dataset Labels

附录重点展示了标签分布图，用于说明 ParasGB 的训练难点：

- **Analog Reff**：20 个 analog 电路的有效电阻分布
- **SRAM Cc**：6 个 SRAM 电路的耦合电容分布
- **SRAM Reff**：6 个 SRAM 电路的有效电阻分布
- **Analog Cg**：20 个 analog 电路的地电容分布
- **SRAM Cg**：6 个 SRAM 电路的地电容分布

附录总结出的主要现象：

- 地电容标签普遍呈现**明显长尾分布**
- SRAM 的标签跨度更大，回归更难
- Analog 的有效电阻标签比电容更稀疏，异常值更多
- SRAM 中耦合电容边数量远多于地电容节点标签，是检验模型非局部建模能力的关键场景

---

## C. Experiment Details

### C.1 Data Preprocessing

#### SRAM 任务过滤规则

- `Cg` 节点任务：保留 `(1e-21, 1e-15) F`
- `Cc` 边任务：保留 `(1e-21, 1e-15) F`
- `Reff` 边任务：过滤零值，并使用全局 `1% - 99%` 分位归一化

#### Analog 任务过滤规则

- `Cg` 节点任务：保留 `(0, 8e-13) F`
- `Reff` 边任务：保留 `[0, 700] Ω`

分类任务统一采用 **5 类等宽分箱**。

#### 回归任务归一化

附录说明所有输入特征与输出标签都做归一化处理，原因是：

- 空间坐标、器件尺寸、寄生参数量纲差异巨大
- 原始数值容易导致梯度发散
- 大数值特征会掩盖微弱但关键的物理属性

### C.2 Baseline Details

附录把基线分为三类：

#### 1) Classical Message-Passing GNNs

- GCN
- GAT
- GraphSAGE
- PNA

特点：

- 主要建模局部邻域结构
- 计算效率高
- 显存占用较低
- 是电路图任务最常见的基线组

#### 2) Graph Transformers

- SGFormer
- PolyNormer

特点：

- 引入全局注意力
- 更适合建模长程依赖
- 对具有重复阵列与远程耦合效应的 SRAM 更有潜力

#### 3) Circuit-Specific Models

- ParaGraph
- CirGPS
- CircuitGCL

特点：

- 面向 EDA / 寄生建模场景专门设计
- 分别强调物理感知、few-shot 学习、对比学习与标签重平衡

### C.3 Experimental Setting Details

#### Analog 设置

附录给出的统一设置要点：

- hidden dim = `96`
- activation = `LeakyReLU`
- dropout = `0.4`
- batch size = `64`
- learning rate = `5e-5`
- epochs = `200`
- 邻居采样：每层 `64` 个邻居，深度 `3` 层

补充说明：

- PolyNormer：local layers = `7`，global layers = `2`
- ParaGraph：更适合 `1e-4` 学习率和更大 batch
- 在回归任务中，PNA / GCN 将 tower 数从 `4` 降到 `2` 后更稳定

#### SRAM 设置

由于 SRAM 图规模达到千万级，附录重点优化采样与吞吐：

- 小型 SRAM（如 SSRAM）：采样率 `1.0`
- 大型 SRAM（如 sandwich）：采样率 `0.1`
- batch size = `128`
- hidden dim = `144`
- 大多数模型启用 residual，关闭 batch normalization

专用模型设置：

- CircuitGCL：在 SRAM 上做 `300` 轮对比学习预训练，dropout = `0.3`
- CirGPS：使用 `144` 维特征和 ClusterGCN 采样

### C.4 Hardware and Software Configurations

附录给出的实验环境：

- 框架：PyTorch Geometric
- CPU：Intel Xeon Silver 4314 (2.4 GHz)
- 系统内存：128 GB
- GPU：
  - 4 × NVIDIA RTX 4090 (24 GB)
  - 3 × NVIDIA A100 (40 GB)

---

## D. Additional Results

附录 D 主要补充**回归任务**结果，因为正文更强调分类任务。

### D.1 Performance on Node Regression Task

补充结论：

- 分类准确率高，不代表回归数值精度也高
- SRAM 回归比 Analog 更难
- 在大规模 SRAM 上，一些传统模型甚至出现负 R²，说明其预测不如简单均值基线
- Analog 的回归整体更稳，PNA / PolyNormer 在若干测试集上能保持较高 R²

### D.2 Performance on Edge Regression Task

附录指出：

- SRAM 的 `Cc` 边回归难度比节点任务更高
- 即使表现较好的模型，`Cc` 的 MAE 仍不低，说明微弱串扰差异很难从拓扑中充分恢复
- Analog 的 `Reff` 回归相对更稳健，PNA 与 PolyNormer 在多数测试集上取得较高 R²

### D.3 Comparative Discussion

附录给出的比较性结论可以概括为：

1. 分类精度与数值预测精度不是一回事。
2. SRAM 的长程依赖问题在回归模式下更明显。
3. `Reff` 更适合连续值建模，而不是简单分箱离散化。

---

## E. Limitations

附录明确列出当前 ParasGB 的几项限制：

### 1. Circuit Type 覆盖不足

当前主要覆盖 SRAM 和部分 analog 模块，尚不能覆盖：

- 复杂数字逻辑
- 大规模 SoC
- 更多工业场景中的异构版图风格

### 2. 高精度回归仍然困难

寄生参数标签存在显著长尾分布，极值样本难预测。分箱分类虽然降低了难度，但只是折中方案；工业场景所需的高精度回归仍未解决。

### 3. 缺少跨工艺节点验证

当前数据主要来自特定先进工艺，尚不能充分验证模型在不同节点（如 28nm 到 5nm）之间的泛化能力。

### 4. 深层物理交互建模不足

现有特征更偏向坐标和器件尺寸，对以下因素建模不够：

- 热效应
- 多层金属间复杂电磁耦合
- 更深层的 3D 物理交互

---

## F. Future Directions

附录给出了四个未来方向：

### 1. Graph Foundation Model 预训练

在大规模电路图上做自监督预训练，学习通用电路物理规律，再在具体任务上少量微调。

### 2. 增强空间几何感知

把 3D 版图信息和金属层属性更深地融入消息传递，让模型同时理解：

- 拓扑连接关系
- 三维相对位置
- 布线耦合关系

### 3. 构建更全面的评测平台

未来版本计划纳入：

- RF 电路
- 高速接口
- 大规模数字逻辑模块
- 多工艺节点版图数据

### 4. 面向设计闭环的实时指导

从离线 benchmark 走向在线设计辅助，在版图阶段提供寄生参数预警与优化建议。

---

## G. Analog Topology-To-Graph

附录 G 描述了 analog 电路从原理图到图表示的转换方式。

### 图的组成

每个电路被建模为异构图 `G = (V, E)`，其中：

- **device nodes**：器件
- **net nodes**：互连网络
- **pin nodes**：器件引脚

### 输入拓扑边

黑色拓扑边 `E_topo` 来自原理图连接关系，包括：

- device-to-pin
- pin-to-net

这些边构成模型输入图。

### 监督标签

寄生信息来自提取后的 parasitic netlist：

- 蓝色 pin-to-pin 边：作为电阻边，标签是两引脚间有效电阻
- 每个 net 节点：赋予总地电容标签 `Cg`

这些寄生量不作为输入边，而是作为预测目标。

---

## H. Algorithms

### H.1 Matrix-Based Effective Resistance Calculation

为了生成高精度物理标签，附录 H 使用**基于矩阵运算的有效电阻计算方法**，而不是昂贵的直接电路仿真。

### 核心思想

1. 根据电阻网表构建节点导纳矩阵 `G`
2. 将每个电阻 `r` 转换为电导 `g = 1/r`
3. 按照 KCL 更新矩阵的对角与非对角项
4. 去除参考节点（通常为地）对应的行列，得到可逆导纳矩阵
5. 通过可逆导纳矩阵的 **Cholesky 分解逆** 计算任意两节点的有效电阻

### 优点

- 比传统路径搜索法更高效
- 能支持千万级边标签生成
- 适合构造 ParasGB 中的大规模 `Reff` 监督信号

---

## Appendix-Only Summary

如果把这份附录当作仓库文档来看，它主要补足了正文中没有展开的四类内容：

- **怎么用**：统一 API、loader、evaluator、最小训练样例
- **数据长什么样**：SRAM/Analog 特征、子集说明、标签分布
- **实验怎么做**：过滤规则、归一化、基线、超参数、硬件
- **后续怎么扩展**：局限性、未来方向、图构建细节与 Reff 标签算法
