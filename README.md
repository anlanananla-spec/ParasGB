# ParasGB# ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/release/python-310/)
[![PyTorch](https://img.shields.io/badge/PyTorch-2.2.0-red)](https://pytorch.org/)
[![PyG](https://img.shields.io/badge/PyG-2.6.1-orange)](https://pytorch-geometric.readthedocs.io/)

Official implementation and dataset suite for the paper **"ParasGB: A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits"**. 

## 🔍 项目概述 (Project Overview)

ParasGB 是首个专门针对模拟与混合信号 (AMS) 电路寄生参数预测的开源图形基准测试套件。与仅停留在原理图阶段的现有数据集不同，ParasGB 的所有数据均提取自经过流片验证 (tape-out proven) 的真实工业级版图，并使用商业 EDA 工具（如 StarRC）进行高精度寄生参数提取。

本套件旨在打破半导体研究领域的数据壁垒，提供标准化的评估协议，支持从节点级（接地电容）到边级（耦合电容、有效电阻）的多层级预测任务。

![电路拓扑转图形工作流](imgs/fig-graph-conversion.png)
*(注：请将论文中的 Figure 2 截图保存至 imgs/ 目录下以展示拓扑转换流程)*

## ✨ 核心架构与特性 (Core Architecture)

[cite_start]为了大幅降低图学习研究人员进入 EDA 领域的门槛，ParasGB 与 PyTorch Geometric (PyG) 框架进行了深度整合 [cite: 1056-1061]。
* [cite_start]**高度自动化的数据流**：仅需指定数据集名称，即可在云端自动完成原始电路文件的下载与物理特征预处理 [cite: 1073]。
* [cite_start]**大规模图形加载器**：针对超千万节点的 SRAM 电路，内置了与 PyG 兼容的 `NeighborLoader` 和 `LinkNeighborLoader`，支持在显存受限的硬件上进行高效的子图采样，并包含预处理缓存机制 [cite: 1059-1061]。
* [cite_start]**标准化评估器 (Evaluator)**：借鉴 OGB 设计范式，提供统一的自动打分模块。分类任务输出 Accuracy/F1，回归任务输出 MAE/R²，确保算法对比的绝对公平与透明 [cite: 1063-1066]。

## 📊 数据集深度解析 (Dataset Deep Dive)

本基准测试涵盖两个极具挑战性的电路领域，它们各自呈现出独特的拓扑结构和极端的不平衡标签分布（长尾效应）。

### [cite_start]SRAM 电路：工业级超大规模阵列 [cite: 1265-1294]
[cite_start]SRAM 阵列的特点在于其极其庞大的规模和密集的耦合拓扑。为了在这种规模下进行有效学习，我们提取了13维度的统计聚合节点特征（如连接器件的总体通道宽度/长度、乘数等）[cite: 1220-1223]。
* **覆盖范围**：从基础模块到顶级工业设计。
* **代表性数据集**：
  * `ssram`: 基础静态随机存取存储器阵列。
  * `digtime` & `timing_ctrl`: 复杂的数字时序与控制逻辑模块。
  * `sandwich`: 采用堆叠版图设计的高性能架构，具有极高的寄生耦合密度。
  * `ultra8t`: 针对亚阈值功耗优化的 8T SRAM 设计。
  * `array_128_32_8t`: 包含数千万节点与边的超大阵列，专门用于测试模型的极致扩展性与抗压能力。

### [cite_start]模拟电路 (Analog)：微观物理特征的高敏度 [cite: 1295-1376]
模拟电路虽然规模较小（数百至数千节点），但对寄生参数的微小偏差极为敏感。节点特征包含详尽的器件级几何参数（W、L、乘数、指状结构数量等）。
* **覆盖范围**：共包含 20 个经典的模拟模块 (ID 1 至 20)。
* **代表性模块**：涵盖低压差分信号 (LVDS)、超低功耗运算放大器 (OP)、带隙基准 (BGR) 以及支持宽负载范围的低压差线性稳压器 (LDO)。

## ⚙️ 框架核心机制 (Framework Mechanics)

### 版图拓扑至图形转换 (Topology-to-Graph Conversion)
[cite_start]我们将庞大复杂的后仿真 RC 提取网表简化为图神经网络可处理的异构图 [cite: 2068-2078]：
1. **节点构建**：划分为器件节点 (Devices)、网络节点 (Nets) 和引脚节点 (Pins)。
2. **连接性建模**：拓扑边（黑色）捕捉原理图固有的器件-引脚、引脚-网络连接。
3. **标签映射**：接地电容作为网络节点的标签；有效电阻和耦合电容作为节点对之间的边级标签进行预测。

### 矩阵驱动的高效电阻计算 (Matrix-Based Resistance Calculation)
[cite_start]在处理数千万条边级标签时，传统的电路路径搜索计算耗时极高。我们实现了一种基于矩阵运算的高效算法 [cite: 2087-2108]：
* 构建电路的节点导纳矩阵。
* 通过消除基准节点生成可逆导纳矩阵。
* **核心突破**：利用 Cholesky 分解的逆矩阵，以 O(1) 级别的时间复杂度快速查询图中任意两个引脚间的有效电阻。

## 📈 实验基准与深度洞察 (Baselines & Insights)

[cite_start]我们提供了全面的数据预处理（极值过滤与对数归一化以防止梯度发散）[cite: 1701-1707][cite_start]，并测试了以下三大类基准模型 [cite: 1740-1772]：
1. **经典图神经网络**：GCN, GAT, GraphSAGE, PNA。
2. **图 Transformer 模型**：SGFormer, PolyNormer（具备全局感受野，适合长程耦合效应）。
3. **电路领域专用模型**：ParaGraph, CirGPS, CircuitGCL。

**核心实验洞察 (来自附录 D)**：
[cite_start]虽然分类任务表现尚可，但在高精度的**回归任务 (Regression)** 中挑战巨大。对于千万级 SRAM 阵列，常规 GNN（如 GCN/GAT）往往无法捕捉极端的长尾数值分布，甚至导致 $R^2$ 呈现负值 [cite: 1881-1886][cite_start]。然而，在预测有效电阻时，因其数值变化趋势较平滑，PNA 和 PolyNormer 表现出了极强的鲁棒性（$R^2 > 0.8$）[cite: 1938-1941]。

## 💻 快速上手与 API 使用 (Quick Start)

通过深度集成的 API，您可以利用极少的代码启动训练与标准化评估流程。

```python
from parasgb import RCDataset, Evaluator
import torch

# 1. 实例化数据集 (系统将自动下载、预处理并应用统一的数据切分)
dataset = RCDataset(
    dataset_name='sram',      # 可选 'sram' 或 'analog'
    root='data/', 
    task_level='node',        # 可选 'node' 或 'edge'
    task_type='regression'    # 可选 'regression' 或 'classification'
)

# 2. 获取数据加载器 (针对 SRAM 自动启用大图采样)
train_loader = dataset.get_dataloader(split='train', batch_size=32, shuffle=True)

# 3. 定义模型与优化器
model = MyModel() 
criterion = torch.nn.MSELoss()
optimizer = torch.optim.Adam(model.parameters(), lr=1e-4)

# 4. 训练迭代
model.train()
for epoch in range(200):
    for batch in train_loader:
        optimizer.zero_grad()
        y_pred = model(batch.node_attr, batch.edge_index)
        loss = criterion(y_pred.squeeze(), batch.y[:, 0])
        loss.backward()
        optimizer.step()

# 5. 调用标准化评估器
evaluator = Evaluator(dataset_name='sram', task='cg_regr')
metrics = evaluator.evaluate(y_pred, batch.y[:, 0])
print(f"标准化评估 MAE: {metrics['mae']:.4f}")
