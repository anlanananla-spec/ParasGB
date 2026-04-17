# ParasGB

> A Graph Benchmark Suite for Parasitic Estimation on AMS Circuits
>
> 这份 README 依据论文附录内容重组，目的是把论文里的附录 A–H 改写成更适合 GitHub 仓库首页阅读的说明文档。它偏向“仓库导读 + 使用说明 + 数据说明 + 实验补充”，不假设任何论文外的目录结构或安装方式。

---

## 1. 项目简介

ParasGB 是一个面向 **AMS / Analog / SRAM 电路寄生参数预测** 的图学习基准套件。它将真实电路设计流程中提取出的寄生网络整理为图数据，并统一支持：

- **节点级任务**：预测 ground capacitance (`Cg`)
- **边级任务**：预测 coupling capacitance (`Cc`) 与 effective resistance (`Reff`)
- **任务形式**：同时支持 **classification** 与 **regression**
- **数据域**：覆盖 **SRAM** 和 **Analog** 两大类电路
- **评测方式**：提供统一 `Evaluator`，输出 `Accuracy / F1 / MAE / R²`

这个 benchmark 的核心目标不是只做“模型分数比较”，而是为 **真实工业寄生预测** 提供一套更标准、更透明、更可复现的研究入口。

---

## 2. 你能在这个基准里做什么

ParasGB 主要适合以下研究场景：

- 电路寄生参数预测（`Cg` / `Cc` / `Reff`）
- 大规模异构图学习
- 小样本 / 长尾分布 / 标签不平衡下的 GNN 研究
- 图 Transformer 与电路专用 GNN 的横向比较
- 面向 EDA 的图基础模型预训练与迁移研究

---

## 3. 基准覆盖范围

### 3.1 电路类型

#### SRAM 子集
SRAM 数据具有两个非常明显的特点：

- **图规模极大**：节点可达百万到千万级
- **耦合电容密集**：先进工艺下线间距变小，`Cc` 极其密集

包含的代表性数据集：

- `ssram`：基础 SRAM，用于验证模型对规则拓扑的捕捉能力
- `digtime`：包含数字时序相关逻辑模块
- `timing_ctrl`：来自 memory 内部时序控制模块，拓扑复杂度高于一般存储阵列
- `sandwich`：堆叠式高性能 memory 架构，寄生耦合密度极高
- `ultra8t`：针对亚阈值低功耗优化的 8T SRAM
- `array_128_32_8t`：数据集中最大的阵列之一，用于极限性能压测

#### Analog 子集
Analog 数据规模通常小于 SRAM，但器件级物理特征更细，且对寄生误差更敏感。论文附录给出的电路族主要包括：

- `LVDS`
- `Operational Amplifier (OP)`
- `Band-Gap Reference (BGR)`
- `Low-Dropout Regulator (LDO)`

<details>
<summary>展开查看 20 个 Analog 设计的一句话说明</summary>

- `ID 1`：LVDS，高频时钟生成/精密定时应用
- `ID 2`：OP，超低电压参考生成
- `ID 3`：BGR，自级联复合晶体管实现稳定参考
- `ID 4`：BGR，电阻细分/无电阻方法实现低温漂参考
- `ID 5`：LDO，基于 N/P MOS 温度特性平衡的参考生成
- `ID 6`：LDO，快速瞬态响应，补偿电容很小
- `ID 7`：LDO，与 ID 5 同类参考生成结构
- `ID 8`：LDO，自适应补偿 buffer，无需外接电容
- `ID 9`：LDO，三环结构，瞬态响应快、PSR 宽频表现好
- `ID 10`：OP，在噪声与功耗之间做可控折中
- `ID 11`：LDO，双并行反馈路径，提高稳定性与瞬态表现
- `ID 12`：LDO，三级高增益误差放大器，适配超低压供电
- `ID 13`：OP，主动补偿替代 Miller compensation
- `ID 14`：BGR，多 MOS + lateral PNP + well resistor 组合
- `ID 15`：LDO，阻尼零点补偿 + slew-rate enhancement
- `ID 16`：LDO，宽负载范围、宽输出电容范围稳定调节
- `ID 17`：LDO，nested adaptive FVF，超快瞬态响应
- `ID 18`：LDO，高增益、高负载能力
- `ID 19`：BGR，利用尺寸相关效应抵消工艺漂移
- `ID 20`：BGR，超低功耗结构，主要电流用于输出

</details>

### 3.2 任务矩阵

| 电路域 | 节点级 | 边级 | 分类 | 回归 |
|---|---:|---:|---:|---:|
| SRAM | `Cg` | `Cc`, `Reff` | ✅ | ✅ |
| Analog | `Cg` | `Reff` | ✅ | ✅ |

### 3.3 指标

- **Classification**：`Accuracy`, `F1`
- **Regression**：`MAE`, `R²`

---

## 4. 数据表示与图构建

ParasGB 将电路统一建模为异构图 `G = (V, E)`：

### 4.1 节点类型

- **device nodes**：器件节点
- **net nodes**：网络/互连节点
- **pin nodes**：器件引脚节点

### 4.2 输入图中的拓扑边

输入图只保留电路拓扑关系：

- `device -> pin`
- `pin -> net`

这些边来自原始 schematic / netlist，是模型的主要输入结构。

### 4.3 监督标签

寄生信息不直接作为输入边，而是作为预测目标：

- `Cg`：作为 **net node 的节点标签**
- `Cc`：作为 **net-pair 的边标签**
- `Reff`：作为 **pin-pair 的边标签**

### 4.4 为什么要做 lumped simplification

真实 post-layout parasitic netlist 往往非常复杂，一个逻辑 net 会展开成大规模分布式 RC 网络。ParasGB 将其简化为可学习的 lumped 监督目标：

- net 的总 ground capacitance
- net 对之间的总 coupling capacitance
- 端口/引脚之间的 effective resistance

这一步兼顾了：

1. 保留关键时序/噪声相关物理量
2. 使大规模工业电路上的图学习变得可行

### 4.5 Analog 专用图构建补充

对于 analog 电路，图构建思路仍然一致，但更强调：

- 从 schematic 到 hetero graph 的转换
- 从 parasitic netlist 中提取 pin-to-pin resistive edges
- 将每个 net 的总 ground capacitance 作为节点标签

---

## 5. 特征设计

### 5.1 SRAM 特征（统计聚合型）

SRAM 采用更偏统计聚合的特征设计，以控制超大图的特征维度，核心包含：

#### Device features
- `Mmos`, `L`, `W`
- `Mres`, `Lres`, `Wres`
- `Mcap`, `Lr`, `Nr`
- `Np`, `T`

#### Net features
- `Nmos`, `Ng`, `Nsd`, `Nb`
- `Wtot`, `Ltot`
- `Ncap`, `Lrtot`, `Nrtot`
- `Nres`, `Wtot,res`, `Ltot,res`
- `Nport`

#### Pin features
- MOS 引脚类型（如 `G/D/S/B`）

### 5.2 Analog 特征（器件级细粒度）

Analog 更强调器件物理属性与连接统计，例如：

- MOS / resistor / capacitor 类型标记
- 器件 `W`, `L`, `Mmos`, `Nf`, `T`
- net 的功能标记与连接统计
- pin 的电气角色标记

总体上，ParasGB 的特征工程遵循一个很直接的原则：

> 用 schematic 里可获得的几何/拓扑/连接统计，去近似 layout 之后才显式出现的寄生效应。

---

## 6. 标签特性

论文附录把标签分布问题讲得很清楚：这是 ParasGB 的核心难点之一。

### 6.1 主要现象

- 标签分布**长尾**
- 标签分布**高度不均衡**
- 不同任务上可能出现**稀疏异常值**
- SRAM 的 `Cc` 边数量通常远多于 `Cg`

### 6.2 含义

这意味着：

- 高分类准确率不等于模型真的学会了数值规律
- 常规 loss 容易被主峰样本主导
- 极端样本（但往往对时序/IR-drop/噪声最关键）更难学到

---

## 7. 数据预处理与归一化

### 7.1 SRAM

- **节点任务 `Cg`**：保留有效值范围 `(1e-21, 1e-15) F`
- **边任务 `Cc`**：保留有效值范围 `(1e-21, 1e-15) F`
- **边任务 `Reff`**：过滤零值，并进行 **全局 1%–99% percentile normalization**

### 7.2 Analog

- **节点任务 `Cg`**：保留有效值范围 `(0, 8e-13) F`
- **边任务 `Reff`**：保留有效值范围 `[0, 700] Ω`

### 7.3 分类与回归

- **分类任务**：统一做 **5 类等宽分箱**
- **回归任务**：对输入特征和输出标签统一归一化

### 7.4 为什么要归一化

原因主要有两个：

- 电路空间坐标、器件尺寸、寄生参数的量纲跨度很大
- 不归一化时容易发生梯度不稳定，以及大数值特征淹没小数值物理特征

---

## 8. 统一 API 与评测接口

ParasGB 强调“尽量少写胶水代码”，核心接口包括：

- `RCDataset`
- `Evaluator`
- `NeighborLoader`
- `LinkNeighborLoader`

### 8.1 `Evaluator` 任务名

#### SRAM
```python
Evaluator(dataset_name='sram', task='cg_regr')
Evaluator(dataset_name='sram', task='cg_class')
Evaluator(dataset_name='sram', task='cc_regr')
Evaluator(dataset_name='sram', task='cc_class')
Evaluator(dataset_name='sram', task='r_class')
```

#### Analog
```python
Evaluator(dataset_name='analog', task='cg_regr')
Evaluator(dataset_name='analog', task='cg_class')
Evaluator(dataset_name='analog', task='r_regr')
Evaluator(dataset_name='analog', task='r_class')
```

### 8.2 推荐使用方式

- 节点任务：`NeighborLoader`
- 边预测任务：`LinkNeighborLoader`
- 大图训练：优先采用子图采样
- 结果汇报：统一通过 `Evaluator` 输出标准指标

---

## 9. Quick Start

下面这段代码是根据附录示例整理后的可读版：

```python
from parasgb import RCDataset, Evaluator

# 1) 加载数据集
# dataset_name: 'sram' or 'analog'
# task_level: 'node' or 'edge'
# task_type: 'regression' or 'classification'
dataset = RCDataset(
    dataset_name='sram',
    root='data/',
    task_level='node',
    task_type='regression',
)

# 2) 获取 DataLoader
train_loader = dataset.get_dataloader(
    split='train',
    batch_size=32,
    shuffle=True,
)

# 3) 定义模型
model = MyModel()
criterion = Loss()
optimizer = Optimizer(model.parameters())

# 4) 训练
for epoch in range(E):
    for batch in train_loader:
        y_pred = model(batch.node_attr, batch.edge_index)
        loss = criterion(y_pred.squeeze(), batch.y[:, 0])
        loss.backward()
        optimizer.step()
        optimizer.zero_grad()

# 5) 评测器
evaluator = Evaluator(
    dataset_name='sram',
    task='cg_regr',
)

# 6) 评估
metrics = evaluator.evaluate(y_pred, y_true)
print(f"MAE = {metrics['mae']:.4f}")
```

---

## 10. 基线模型

ParasGB 把基线分成三类：

### 10.1 Classical Message-Passing GNNs

- `GCN`
- `GAT`
- `GraphSAGE`
- `PNA`

这类方法偏重局部邻域建模，是图学习里的常规强基线。

### 10.2 Graph Transformers

- `SGFormer`
- `PolyNormer`

这类方法引入全局注意力，更适合处理：

- 重复拓扑
- 长距离依赖
- 超大图中的非局部耦合

### 10.3 Circuit-Specific Models

- `ParaGraph`
- `CirGPS`
- `CircuitGCL`

这类模型更针对 EDA 场景进行了优化，例如：

- 小样本问题
- 标签不平衡
- 跨设计迁移

---

## 11. 实验设置

### 11.1 Analog 设置

- hidden dim：`96`
- activation：`LeakyReLU`
- dropout：`0.4`
- batch size：`64`
- learning rate：`5e-5`
- epoch：`200`
- 邻居采样：每层 `64` 个邻居，共 `3` 层

特殊说明：

- `PolyNormer`：local layers = `7`，global layers = `2`
- `ParaGraph`：更适合 `1e-4` 学习率和更大 batch size（`128`）
- 对部分回归任务，`PNA / GCN` 把 aggregation towers 从 `4` 降到 `2` 后更稳定

### 11.2 SRAM 设置

SRAM 的重点不是“怎么多堆层”，而是“怎么把超大图训练跑稳”。

- 对小规模 SRAM（如 `ssram`）可采用较高采样率
- 对超大规模 SRAM（如 `sandwich`）采样率可降到 `0.1`
- batch size：`128`
- hidden dim：`144`
- 大多数模型开启 residual connection
- 在采样异构图上，batch normalization 可能导致训练不稳定，因此常关闭

专项设置：

- `CircuitGCL`：在 SRAM 上采用 `300` 轮对比学习预训练，dropout `0.3`
- `CirGPS`：使用 `144` 维特征，并结合 `ClusterGCN` 采样

### 11.3 硬件环境

实验基于 PyTorch Geometric，运行在共享计算集群上，配置包括：

- Intel Xeon Silver 4314 CPU（2.4 GHz）
- 128 GB RAM
- 4 × NVIDIA RTX 4090（24 GB）
- 3 × NVIDIA A100（40 GB）

---

## 12. 附加结果：如何理解分类与回归

论文附录补充了一个很有价值的信息：

### 12.1 分类好，不代表回归一定好

某些模型在分类任务中表现不错，但在回归任务中会因为长尾分布和极端值问题明显退化。

### 12.2 SRAM 回归比 Analog 更难

在超大规模 SRAM 上，节点/边级回归更容易出现：

- `R²` 很低
- 甚至低于简单均值基线
- 说明模型很难学到细微的寄生数值变化

### 12.3 电阻任务的一个有趣现象

附录指出：

- `Reff` 的**分类任务**并不一定容易
- 但 `Reff` 的**回归任务**在不少测试集上反而更稳定

这提示后续研究中，针对电阻预测，可能应该更重视 **连续值建模**，而不是简单离散分箱。

---

## 13. 局限性

ParasGB 已经很有价值，但论文也明确给出了几个局限：

1. **电路类型覆盖不足**
   - 当前主要覆盖 SRAM 和若干 Analog 模块
   - 对复杂数字逻辑、大型 SoC 等场景代表性仍有限

2. **高精度回归仍然困难**
   - 长尾分布导致极值样本预测难
   - 分类离散化只是缓解，不是根本解决

3. **跨工艺节点验证不足**
   - 不同工艺节点的规则与物理性质差异大
   - 当前缺少大规模跨节点对比数据

4. **深层物理交互建模不足**
   - 当前特征主要围绕坐标、器件尺寸、拓扑统计
   - 对热效应、多层金属复杂电磁耦合、3D 空间交互仍建模不够

---

## 14. 未来方向

论文附录提出了很清晰的几个后续方向：

### 14.1 图基础模型预训练
在大规模电路图上做自监督预训练，让模型先学到通用电路物理规律，再在具体任务上少量微调。

### 14.2 增强 3D 几何感知
把 3D layout 信息、金属层属性、空间相对位置更深地整合进 message passing，而不仅仅把坐标当普通数值特征。

### 14.3 构建更全面的评测平台
纳入：

- RF 电路
- 高速接口
- 大规模数字逻辑
- 多工艺节点 layout 数据

把 ParasGB 扩展为更完整的 EDA 通用评测平台。

### 14.4 走向设计闭环中的实时指导
未来目标不是只做离线 benchmark，而是把预测模型真正嵌入设计流程，在 layout 过程中提供实时预警和优化建议。

---

## 15. 有效电阻标签是怎么生成的

附录 H 给出了一个很重要的标签生成思路：**基于矩阵运算计算 effective resistance**。

### 15.1 核心思想

对电路建立节点导纳矩阵 `G`：

1. 遍历所有电阻元件
2. 将阻值 `r` 转换为导纳 `g = 1 / r`
3. 根据 Kirchhoff 电流定律更新 `G` 的对角与非对角元素
4. 选定参考节点（通常是地）
5. 删除对应行列，得到可逆的约化导纳矩阵

### 15.2 计算阶段

之后通过 Cholesky 分解：

- 设 `Gred = L L^T`
- 计算 `L^-1`
- 对任意两个端口，利用对应列向量的欧氏距离得到有效电阻

可理解为：

```text
Req(src, dst) = || z_src - z_dst ||_2
```

其中 `z_src` 和 `z_dst` 来自 Cholesky 因子逆矩阵中的对应列。

### 15.3 为什么这个方法重要

相比传统路径搜索或直接电路仿真，这种矩阵法更适合：

- 批量生成边标签
- 大规模数据预处理
- 支撑千万级边标签构建

---

## 16. 阅读建议

如果你是第一次接触 ParasGB，建议按下面顺序读：

1. **Quick Start**：先跑通 `RCDataset + Evaluator`
2. **任务矩阵**：明确自己做的是 `Cg / Cc / Reff`，节点还是边
3. **数据表示**：理解为什么输入图不直接包含寄生边
4. **预处理规则**：确认你的标签过滤与归一化设置
5. **实验设置**：尤其是 SRAM 的采样策略
6. **附加结果与局限性**：避免只看分类准确率做结论

---

## 17. 这份 README 的定位

这不是对真实代码仓库目录的逐文件说明，而是：

- 对论文**附录内容**的仓库化重写
- 对使用入口、数据定义和实验补充的集中整理
- 方便读者在 GitHub 首页快速理解 ParasGB 的价值与使用方式

如果你后续需要，我还可以继续把它拆成更像真实开源仓库的版本，例如补成：

- `Installation`
- `Repository Structure`
- `Dataset Download`
- `Training`
- `Evaluation`
- `Citation`
- `FAQ`

