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

## 1. User Guide

### 1.1 Standardized Evaluation Protocol

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

### 1.2 ParasGB Usage

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

## 1. Dataset Details

### 1.1 SRAM 

SRAM 子集采用**统计聚合型特征**，重点描述全局拓扑与器件分布，而不是保留过细的局部器件细节。这样做是为了在超大图场景中控制特征维度，同时仍保留对物理结构有意义的统计信息。
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

SRAM电路图节点特征定义为：
### Definition of SRAM Circuit Graph Node Features

#### Device

| Type | Feature | Definition | Index |
| --- | --- | --- | --- |
| Device | $M_{mos}$ | Multiplier of transistors | 0 |
| Device | $L$ | Length of the transistor | 1 |
| Device | $W$ | Width of the transistor | 2 |
| Device | $M_{res}$ | Multiplier of connected resistors | 3 |
| Device | $L_{res}$ | Length of resistor | 4 |
| Device | $W_{res}$ | Width of resistor | 5 |
| Device | $M_{cap}$ | Multiplier of connected capacitor | 6 |
| Device | $L_{r}$ | Length of capacitor | 7 |
| Device | $N_{r}$ | Number of capacitor fingers | 8 |
| Device | $N_{p}$ | Number of ports in the device instance | 9 |
| Device | $T$ | Type code of the device instance | 10 |

#### Net

| Type | Feature | Definition | Index |
| --- | --- | --- | --- |
| Net | $N_{mos}$ | Number of connected transistors | 0 |
| Net | $N_{g}$ | Number of connected gate terminals | 1 |
| Net | $N_{sd}$ | Number of connected source/drain terminals | 2 |
| Net | $N_{b}$ | Number of connected base terminals | 3 |
| Net | $W_{tot}$ | Total width of connected transistor | 4 |
| Net | $L_{tot}$ | Total length of connected transistor | 5 |
| Net | $N_{cap}$ | Number of connected capacitors | 6 |
| Net | $Lr_{tot}$ | Total length of connected capacitors | 7 |
| Net | $Nr_{tot}$ | Total number of connected capacitor fingers | 8 |
| Net | $N_{res}$ | Number of connected resistors | 9 |
| Net | $W_{tot,res}$ | Total width of connected resistors | 10 |
| Net | $L_{tot,res}$ | Total length of connected resistors | 11 |
| Net | $N_{port}$ | Number of connected ports | 12 |

#### Pin

| Type | Feature | Definition | Index |
| --- | --- | --- | --- |
| Pin | -- | Pin types (G/D/S/B for MOS) | 0 |

#### 1.2 Analog

Analog数据集与SRAM数据集不同，模拟电路更小，但在器件级具有非常详细的物理特性描述。关键参数，如器件通道宽度W、长度L、源漏区到隔离槽边缘的距离
(LDE效应)都包含在节点特征系统中。电路寄生参数通过商用PEX工具提取。由于模拟电路对噪声高度敏感，即使很小的寄生参数预测误差也会导致电路
仿真结果偏离预期
列出了 20 个 analog case 的来源与功能简介，覆盖：
- `ID 1`:低压差分信号电路，它将低频参考(5-27 MHz)转换为高频时钟(100-700 MHz)，具有最小的相位噪声，用于精确定时应用(Leung & Mok, 2003b)。
- `ID 2`:运算放大器电路，它利用准线性温度特性，从仅0.56V的超低电源产生稳定的0.4V参考电压，仅消耗4.8 μ a电流(Wang & Ye, 2006)。
- `ID 3`:带隙参考电路，它通过利用自级联编码复合晶体管和单个电阻产生稳定的参考电压(靠近硅带隙)，实现25.3 ppm/°C的低温系数，电流仅为25 μ a
(Colombo et al.， 2012)。
- `ID 4`:带隙参考电路，它利用电阻细分和无电阻方法产生高度稳定的910.88 mV参考电压，超低温系数为12.99 ppm/°C
(Koh & Lee, 2014)。
- `ID 5`:低差稳压电路，它通过平衡N/ p型mosfet的温度特性，为LDO稳压器产生稳定的参考电压，在9.7 μ a的低电源电流下
实现36.9 ppm/°C的温度系数(Leung & Mok, 2003a)。
- `ID 6`:低差稳压电路，它提供了一个稳定的输出从一个1.8-4.5V的电源快速瞬态响应和最小的补偿电容(7 pF)，支持高达100
mA的负载电流与0.2 V的低压降(Ho & Mok, 2010a)。
- `ID 7`:低差稳压电路，它通过平衡N/ p型mosfet的温度特性，为LDO稳压器产生稳定的参考电压，在9.7 μ a的低电源电流下
实现36.9 ppm/°C的温度系数(Leung & Mok, 2003a)。
- `ID 8`:低差稳压电路，它利用自适应补偿缓冲器(ACB)在通路晶体管之间动态切换，在宽负载范围(0至30 mA)内实现稳定运
行，无需外部电容器，同时保持6 μ a的低静态电流(Tan等人，2025)。
- `ID 9`:低差稳压电路，它利用三环架构实现超快速瞬态响应(1.15 ns)，并保持清洁电源，具有全频谱电源抑制(PSR >−12 dB，
高达20 GHz)，同时仅消耗50 μ a的静态电流(Lu等人，2015)。
- `ID 10`:运算放大器电路，它提供了一个灵活的，一步一步的方法来平衡噪声性能和功耗，提供比以前的方法更大的设计控
制，多条件SPICE模拟验证(Mahattanakul & Chutichatuporn, 2005)。
- `ID 11`:低差稳压电路，它利用两个并行有源反馈路径创建两个极零对，与单路径方法相比，提供卓越的稳定性和瞬态响应，
同时支持100 ma负载，只有14 μ a的静态电流(Li等人，2020a)。
- `ID 12`:低差稳压电路，它利用高增益，三级误差放大器，即使在超低电压(0.5 V电源)下使用不饱和通管也能保持精确的调
节，实现11.4 a /mm的电流密度2和-62 dB的低频PSR (Kim & Cho, 2023)。
- `ID 13`:运算放大器电路，它用有源结构取代了传统的米勒补偿，消除了右半平面(RHP)零，并引入了左半平面(LHP)零来抵
消第一个非主导极点，从而使单位增益频率增加9.4倍，补偿电容器明显更小(Tan & Zhou，2011)。
- `ID 14`:带隙参考电路，它利用四个mosfet，两个横向PNP晶体管和一个阱电阻的组合，产生稳定的16 μ a输出电流，温度系
数为105 ppm/°C，不需要外部带隙参考或修整过程补偿(Osipov & Paul, 2017)。
- `ID 15`:低差稳压电路，它利用阻尼零补偿技术和慢速增强电路实现稳定和快速瞬态，片上电容仅为1.5 pF，支持100 mA负
载和200 mV差(Ho & Mok, 2010b)。
- `ID 16`:低差稳压电路，它利用WCF电路在非常宽的负载电流(高达100 mA)和负载电容(470 pF至10 nF)范围内保持快速瞬态
响应和稳定的电压调节，而功耗仅为14.4µa (Wang等人，2016)。
- `ID 17`:低差稳压电路，它利用嵌套自适应FVF结构实现超快速瞬态响应(处理负载步骤从1 μ A到20 mA，仅需10 ps)，同时
显着提高PSR (1 MHz时−58.52 dB)和线路调节(Li等人，2020b)。
- `ID 18`:低差稳压电路，它提供稳定的输出，具有高直流增益(101 dB)和精确的带隙参考，以支持0.5 v压差的大电流负载(高
达450 mA)，同时保持固体电源抑制(100 Hz时54.5 dB) (Mart´ınez-Garc´ıa等人，2013)。
- `ID 19`:带隙参考电路，它利用尺寸相关效应来抵消工艺引起的阈值电压变化，实现192 pW的超低功耗和高度稳定的性能(0.
53%的工艺变化)，而无需加工后修整(Ji等人，2019)。
- `ID 20`:带隙参考电路，它使用超低功耗架构产生稳定的参考电压，其中大部分5 μ a电流专用于输出，从1 V电源实现温度
系数< 10 ppm/°C，而不需要高面积运算放大器(Edward, 2009)。



### B.3 Dataset Labels

附录重点展示了标签分布图，用于说明 ParasGB 的训练难点：

- **Analog Reff**：20 个 analog 电路的有效电阻分布
- **SRAM Cc**：6 个 SRAM 电路的耦合电容分布
- **SRAM Reff**：6 个 SRAM 电路的有效电阻分布
- **Analog Cg**：20 个 analog 电路的地电容分布
- **SRAM Cg**：6 个 SRAM 电路的地电容分布

总结出的主要现象：

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
