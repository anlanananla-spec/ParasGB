# ParasGB  



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

## 2. Dataset Details

### 2.1 SRAM 

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

#### 2.2 Analog

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



### 2.3 Dataset Labels

标签分布图：

- **Analog Reff**：20 个 analog 电路的有效电阻分布
  
<table>
  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case1_RC_edge_normalized.png" width="180"><br>
      <sub>(a) Case 1</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case5_RC_edge_normalized.png" width="180"><br>
      <sub>(b) Case 2</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case7_RC_edge_normalized.png" width="180"><br>
      <sub>(c) Case 3</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case10_RC_edge_normalized.png" width="180"><br>
      <sub>(d) Case 4</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case11_RC_edge_normalized.png" width="180"><br>
      <sub>(e) Case 5</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case15_RC_edge_normalized.png" width="180"><br>
      <sub>(f) Case 6</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case17_RC_edge_normalized.png" width="180"><br>
      <sub>(g) Case 7</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case23_RC_edge_normalized.png" width="180"><br>
      <sub>(h) Case 8</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case29_RC_edge_normalized.png" width="180"><br>
      <sub>(i) Case 9</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case39_RC_edge_normalized.png" width="180"><br>
      <sub>(j) Case 10</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case42_RC_edge_normalized.png" width="180"><br>
      <sub>(k) Case 11</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case44_RC_edge_normalized.png" width="180"><br>
      <sub>(l) Case 12</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case45_RC_edge_normalized.png" width="180"><br>
      <sub>(m) Case 13</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case55_RC_edge_normalized.png" width="180"><br>
      <sub>(n) Case 14</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case58_RC_edge_normalized.png" width="180"><br>
      <sub>(o) Case 15</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case71_RC_edge_normalized.png" width="180"><br>
      <sub>(p) Case 16</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case72_RC_edge_normalized.png" width="180"><br>
      <sub>(q) Case 17</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case74_RC_edge_normalized.png" width="180"><br>
      <sub>(r) Case 18</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case75_RC_edge_normalized.png" width="180"><br>
      <sub>(s) Case 19</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/edge_each_label/case78_RC_edge_normalized.png" width="180"><br>
      <sub>(t) Case 20</sub>
    </td>
  </tr>
</table>

- **Analog Cg**：20 个 analog 电路的地电容分布
  <h3 align="center">Analog RC Node Label Distributions</h3>

<table>
  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case1_RC_normalized.png" width="180"><br>
      <sub>(a) Case 1</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case5_RC_normalized.png" width="180"><br>
      <sub>(b) Case 2</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case7_RC_normalized.png" width="180"><br>
      <sub>(c) Case 3</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case10_RC_normalized.png" width="180"><br>
      <sub>(d) Case 4</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case11_RC_normalized.png" width="180"><br>
      <sub>(e) Case 5</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case15_RC_normalized.png" width="180"><br>
      <sub>(f) Case 6</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case17_RC_normalized.png" width="180"><br>
      <sub>(g) Case 7</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case23_RC_normalized.png" width="180"><br>
      <sub>(h) Case 8</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case29_RC_normalized.png" width="180"><br>
      <sub>(i) Case 9</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case39_RC_normalized.png" width="180"><br>
      <sub>(j) Case 10</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case42_RC_normalized.png" width="180"><br>
      <sub>(k) Case 11</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case44_RC_normalized.png" width="180"><br>
      <sub>(l) Case 12</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case45_RC_normalized.png" width="180"><br>
      <sub>(m) Case 13</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case55_RC_normalized.png" width="180"><br>
      <sub>(n) Case 14</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case58_RC_normalized.png" width="180"><br>
      <sub>(o) Case 15</sub>
    </td>
  </tr>

  <tr>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case71_RC_normalized.png" width="180"><br>
      <sub>(p) Case 16</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case72_RC_normalized.png" width="180"><br>
      <sub>(q) Case 17</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case74_RC_normalized.png" width="180"><br>
      <sub>(r) Case 18</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case75_RC_normalized.png" width="180"><br>
      <sub>(s) Case 19</sub>
    </td>
    <td align="center">
      <img src="IMGS/analog/node_each_label/case78_RC_normalized.png" width="180"><br>
      <sub>(t) Case 20</sub>
    </td>
  </tr>
</table>

- **SRAM Cc**：6 个 SRAM 电路的耦合电容分布

<table>
  <tr>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/array_128_32_8t_normalized.png" width="160"><br>
      <sub>(a) Array_128_32_8t</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/digtime_normalized.png" width="160"><br>
      <sub>(b) Digtime</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/sandwich_normalized.png" width="160"><br>
      <sub>(c) Sandwich</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/ssram_normalized.png" width="160"><br>
      <sub>(d) SSRAM</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/timing_ctrl_normalized.png" width="160"><br>
      <sub>(e) Timing_Ctrl</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_each_label/ultra8t_normalized.png" width="160"><br>
      <sub>(f) Ultra8t</sub>
    </td>
  </tr>
</table>
  
- **SRAM Reff**：6 个 SRAM 电路的有效电阻分布
  
<table>
  <tr>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/array_128_32_8t_normalized.png" width="160"><br>
      <sub>(a) Array_128_32_8t</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/digtime_normalized.png" width="160"><br>
      <sub>(b) Digtime</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/sandwich_normalized.png" width="160"><br>
      <sub>(c) Sandwich</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/ssram_normalized.png" width="160"><br>
      <sub>(d) SSRAM</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/timing_ctrl_normalized.png" width="160"><br>
      <sub>(e) Timing_Ctrl</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/edge_r_each_label/ultra8t_normalized.png" width="160"><br>
      <sub>(f) Ultra8t</sub>
    </td>
  </tr>
</table>

- **SRAM Cg**：6 个 SRAM 电路的地电容分布
  
<table>
  <tr>
    <td align="center">
      <img src="IMGS/sram/node_each_label/array_128_32_8t_normalized.png" width="160"><br>
      <sub>(a) Array_128_32_8t</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/digtime_normalized.png" width="160"><br>
      <sub>(b) Digtime</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/sandwich_normalized.png" width="160"><br>
      <sub>(c) Sandwich</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/ssram_normalized.png" width="160"><br>
      <sub>(d) SSRAM</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/timing_ctrl_normalized.png" width="160"><br>
      <sub>(e) Timing_Ctrl</sub>
    </td>
    <td align="center">
      <img src="IMGS/sram/node_each_label/ultra8t_normalized.png" width="160"><br>
      <sub>(f) Ultra8t</sub>
    </td>
  </tr>
</table>

总结出的主要现象：

- 地电容标签普遍呈现**明显长尾分布**
- SRAM 的标签跨度更大，回归更难
- Analog 的有效电阻标签比电容更稀疏，异常值更多
- SRAM 中耦合电容边数量远多于地电容节点标签，是检验模型非局部建模能力的关键场景

---

## 3. Experiment Details

### 3.1 Data Preprocessing

#### SRAM 任务过滤规则

- `Cg` 节点任务：保留 `(1e-21, 1e-15) F`
- `Cc` 边任务：保留 `(1e-21, 1e-15) F`
- `Reff` 边任务：过滤零值，并使用全局 `1% - 99%` 分位归一化

#### Analog 任务过滤规则

- `Cg` 节点任务：保留 `(0, 8e-13) F`
- `Reff` 边任务：保留 `[0, 700] Ω`

分类任务统一采用 **5 类等宽分箱**。

### C.2 Baseline Details

本研究选择了一系列具有代表性的图学习模型进行对比实验，验证ParasGB数据集的任务挑战，具体如下。

#### 1) Classical Message-Passing GNNs
这类模型是图学习领域的主流方法，其核心原理是通过相邻节点之间的特征转移和聚合来学习电
路的局部拓扑结构特征。这类模型具有高效的运算效率和较低的内存占用要求，是电路相关任务中应用最广泛的基准模型。
- `GCN`(Kipf & Welling, 2017):经典的图卷积网络，通过拉普拉斯平滑地聚合邻居节点特征。
- `GAT`(Kipf & Welling, 2017):经典的图卷积网络，通过拉普拉斯平滑地聚合邻居节点特征。
- `GraphSAGE`(Kipf & Welling, 2017):经典的图卷积网络，通过拉普拉斯平滑地聚合邻居节点特征。
- `PNA`(Kipf & Welling, 2017):经典的图卷积网络，通过拉普拉斯平滑地聚合邻居节点特征。

#### 2) Graph Transformers
与传统模型只对局部邻居建模不同，这类模型引入了全局关注机制，可以突破局部邻居
的限制，对整个电路的全局拓扑关联进行建模。对于SRAM等具有重复拓扑结构和远程耦合效应的大型电路网络，这种模型
显示出更强的建模潜力。
- `SGFormer`(Wu et al.， 2023):轻量级图形转换器模型，通过全局注意机制对大规模图形进行高效建模。
- `PolyNormer`(Wu et al.， 2023):轻量级图形转换器模型，通过全局注意机制对大规模图形进行高效建模。

#### 3) Circuit-Specific Models
这类模型是针对EDA领域的特定任务痛点而设计的，针对电路数据稀缺和标
签分布不平衡等问题配置有针对性的优化策略。
- `ParaGraph`(Ren等人，2020):该领域的早期代表性作品，使用分层图符号技术预测领土的寄生参数。
- `CirGPS`(Shen等人，2025c):通过子图采样和预训练策略，解决基于few-shot学习方法的电路数据稀缺性问题。
- `CircuitGCL`(Shen等人，2025c):通过子图采样和预训练策略，解决基于few-shot学习方法的电路数据稀缺性问题。


### 3.3 对比性能研究结果

不同模型在SRAM电路中的性能研究结果：
### Performance of Different Models on SRAM Circuits Ground Capacitance Node Regression Task

| Metric | sram+digtime+timing_ctrl MAE ↓ | sram+digtime+timing_ctrl R² ↑ | sandwich MAE ↓ | sandwich R² ↑ | ultra8t MAE ↓ | ultra8t R² ↑ | array_128_32_8t MAE ↓ | array_128_32_8t R² ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.0461 | 0.8401 | 0.1876 | 0.1636 | 0.0192 | 0.5446 | 0.1199 | -0.1967 |
| GAT | 0.0454 | 0.8469 | 0.1838 | 0.2159 | 0.0955 | 0.5060 | 0.1211 | -0.2518 |
| GraphSAGE | 0.0413 | 0.8800 | 0.1964 | 0.0585 | 0.1093 | 0.2620 | 0.2813 | -2.0157 |
| PNA | 0.0415 | 0.8813 | 0.1845 | 0.3589 | 0.0894 | 0.4561 | 0.2024 | -0.9577 |
| SGFormer | 0.0424 | 0.8729 | 0.2297 | -0.1733 | 0.1619 | -0.3985 | 0.2934 | -3.2101 |
| PolyNormer | 0.0423 | 0.8667 | 0.1938 | 0.1186 | 0.1563 | -0.5618 | 0.1699 | -1.1344 |
| Paragraph | 0.1618 | 0.1487 | 0.1659 | 0.1795 | 0.1571 | 0.1863 | 0.1459 | 0.1973 |
| CirGPS | 0.0063 | 0.9568 | 0.0298 | 0.6574 | 0.0222 | 0.7882 | 0.0194 | 0.8702 |
| CircuitGCL | 0.0511 | 0.8792 | 0.3558 | -0.4768 | 0.3359 | -0.3439 | 0.3314 | -0.6812 |

不同模型在模拟电路接地电容节点回归任务中的性能结果：

### Performance of Different Models on Analog Circuits Ground Capacitance Node Regression Task

| Metric | 1-4, 6, 8-12, 15-18 MAE ↓ | 1-4, 6, 8-12, 15-18 R² ↑ | 5 MAE ↓ | 5 R² ↑ | 14 MAE ↓ | 14 R² ↑ | 20 MAE ↓ | 20 R² ↑ |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| GCN | 0.0272 | 0.9570 | 0.0171 | 0.4200 | 0.0216 | 0.8460 | 0.0284 | 0.8632 |
| GAT | 0.0367 | 0.9336 | 0.0078 | 0.4879 | 0.0107 | 0.8749 | 0.0230 | 0.8270 |
| GraphSAGE | 0.0473 | 0.8861 | 0.0267 | 0.3098 | 0.0311 | 0.8106 | 0.0371 | 0.7869 |
| PNA | 0.0292 | 0.9502 | 0.0089 | 0.4214 | 0.0094 | 0.8507 | 0.0211 | 0.8423 |
| SGFormer | 0.0798 | 0.7283 | 0.0340 | -0.0393 | 0.0339 | 0.5936 | 0.0571 | 0.4162 |
| PolyNormer | 0.0298 | 0.9667 | 0.0155 | 0.3828 | 0.0148 | 0.8638 | 0.0281 | 0.7959 |
| Paragraph | 0.1799 | -0.0527 | 0.0250 | -0.0862 | 0.0391 | -0.0008 | 0.0594 | -0.0170 |
| CirGPS | 0.0783 | 0.8384 | 0.0932 | -2.4965 | 0.1034 | -0.3204 | 0.1100 | 0.1395 |
| CircuitGCL | 0.0386 | 0.9605 | 0.0225 | 0.6119 | 0.0227 | 0.8679 | 0.0299 | 0.8126 |


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
