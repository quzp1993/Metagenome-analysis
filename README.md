# Metagenome-analysis

通用宏基因组分析 skill，覆盖原始 reads 预处理、物种组成、多样性、功能谱、物种–功能关联、R 可视化和 Markdown/HTML/PDF 报告。

## 物种丰度分析方法

本 skill 支持两类常用的 shotgun 宏基因组物种分析方法：Kraken2/Bracken 和 MetaPhlAn。两者的丰度定义不同，结果应分别保存和解释，不能直接混合或取平均。

### Kraken2/Bracken：基于 k-mer 的分类与丰度估计

Kraken2 将每条 read 切分为 k-mer/minimizer，并查询参考数据库。每个 k-mer 根据参考基因组的分类信息定位到分类树节点；一条 read 综合多个 k-mer 的命中结果，通常被分配到得分最高的分类节点。共享序列不会在原始 Kraken2 结果中同时计入多个物种；如果无法区分近缘物种，通常会被分配到属、科或更高层级的共同祖先。

Kraken2 report 中的主要数值是分类 reads 计数。相对丰度通常为分类 reads 数除以总 reads 或总分类 reads，具体分母必须查看所用脚本。它不是按物种 marker 基因计数，也通常不进行“reads 除以参考基因组大小”的简单校正。

Bracken 使用 Kraken2 的分类结果、参考基因组中的 read-length k-mer 分布和近缘分类单元之间的共享序列概率，对原本分配到高层级的 reads 进行概率重新估计，从而给出物种或属水平的估计 reads 数。Bracken 的结果仍然是基于 reads 的统计估计，不等于绝对细胞数。

**优点**

- 数据库覆盖范围广，适合广泛发现细菌、古菌、真菌、病毒和其他分类单元；
- 速度快，适合大规模 shotgun 数据；
- 能保留从种到更高分类层级的分类结果；
- 对数据库中未被 marker 方法覆盖的类群可能更敏感；
- Bracken 可重新分配部分高层级 reads，改善物种/属丰度估计。

**局限**

- 结果受数据库组成、参考基因组数量和注释质量影响；
- 近缘物种共享序列可能导致物种层级不确定；
- 参考基因组大小、可区分 k-mer 数量和菌株差异会影响 read-based 丰度；
- 数据库污染或错误分类可能传递到结果中；
- 低丰度、低覆盖和单样本出现的物种需要谨慎解释；
- read 相对丰度不等于细胞数或生物量。

### MetaPhlAn：基于分类支系特异性 marker 的丰度估计

MetaPhlAn 使用预先筛选的、能够区分特定分类支系的 marker genes。测序 reads 与 marker 数据库比对后，MetaPhlAn 根据多个 marker 的覆盖度和丰度估计分类单元的 genome/cell-like relative abundance，并对 marker 长度和不同 marker 的覆盖差异进行归一化。它不是统计整个参考基因组上所有匹配 reads，因此通常比直接使用 Kraken2 read 比例更少受到整体基因组大小差异的影响。

MetaPhlAn 的输出仍然是相对丰度，不是 qPCR 或流式细胞术意义上的绝对细胞数。其结果依赖 marker 数据库是否覆盖目标物种、marker 是否具有足够特异性以及样本中是否有足够的 marker 覆盖。

**优点**

- 使用分类支系特异性 marker，物种归属通常更严格；
- 对近缘物种和共享基因组区域的影响相对较小；
- 丰度估计更接近 organismal/cell-like relative abundance；
- 不容易因某个物种参考基因组较大而简单增加其 read 计数；
- 输出结果通常较简洁，便于群落比较和下游统计。

**局限**

- 只能识别 marker 数据库覆盖或可归入的类群；
- 新颖、未培养或数据库缺失的微生物可能被漏检；
- marker 缺失、水平转移、菌株差异或共享 marker 会造成偏差；
- 低丰度物种的 marker 覆盖不足时容易不稳定；
- 不能替代绝对定量，也不能直接提供功能活性信息；
- 结果依赖 MetaPhlAn 版本和 marker 数据库版本，跨版本比较需谨慎。

## 两种方法如何选择

| 研究目的 | 优先考虑 | 说明 |
|---|---|---|
| 已知微生物的稳定相对丰度比较 | MetaPhlAn | marker-based 结果更适合做群落比较 |
| 尽可能广泛发现数据库中的分类单元 | Kraken2/Bracken | 数据库范围通常更宽 |
| 关注近缘物种的物种级判定 | MetaPhlAn，并结合验证 | 仍需检查 marker 覆盖度 |
| 关注病毒、低丰度或数据库外类群 | Kraken2/Bracken 或组合方法 | 结果需做数据库和假阳性检查 |
| 需要绝对细胞数或生物量 | 两者都不够 | 需要 qPCR、流式、内部标准或其他定量方法 |

推荐做法是选择一种方法作为主分析，另一种方法作为交叉验证。比较时应关注检出情况、分类层级和组间趋势，不要把两套方法的丰度数值直接相加、取平均或当作同一种统计量。

## 推荐分析流程

```text
原始 FASTQ
  ↓
fastp：去接头、低质量过滤和质控报告
  ↓
bowtie2：按样本类型选择性去除宿主 reads
  ↓
MetaPhlAn 或 Kraken2/Bracken：物种组成和丰度
  ↓
α/β 多样性
  ↓
功能基因/通路分析
  ↓
物种–功能关联、R 可视化和综合报告
```

所有分析应记录：样本 ID、元数据、软件版本、数据库版本、参数、过滤阈值、输入/输出 reads 数和实际执行状态。

## 结果解释边界

- 物种相对丰度不是绝对细胞数；
- Kraken2/Bracken 的 read-based 丰度与 MetaPhlAn 的 marker-based 丰度不是同一个统计量；
- 物种分类结果不能单独证明某个功能由该物种执行；
- 低丰度、单样本出现或不同工具差异很大的类群应进行独立验证；
- 关键结论应结合数据库质量、marker/k-mer 覆盖、比对证据、qPCR 或其他实验数据。

## Skill 内容

完整执行规范见 [`metagenome-analysis/SKILL.md`](metagenome-analysis/SKILL.md)。