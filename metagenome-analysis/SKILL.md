---
name: metagenome-analysis
description: "Analyze metagenomic data from taxonomic composition and diversity through functional profiling, species–function associations, R visualization, and an interpretable report. Use for general end-to-end metagenome analysis workflows."
---

# 通用宏基因组分析

对环境、临床、食品或其他来源的宏基因组项目执行可追溯的端到端分析，并交付可复查的数据表、脚本、图和报告。把本 skill 当作通用分析框架，不要假定数据来源、服务器、文件格式或分析结果。

## 执行原则

- 先定位并检查用户指定的输入、样本表、分析脚本、数据库和现有结果；确认样本 ID、采样日期、池别/处理组、文件配对与数据层级能正确对应。先读现有项目说明，避免重跑已有步骤或覆盖成果。
- 输出脚本、中间文件和结果放在用户指定的结果目录。路径未指定时，先采用项目内专门的结果子目录；不要把临时产物散落到工作目录。文件写入、远程服务器操作和安装软件均遵守当前权限边界；先验证连接、路径与依赖，不能把计划执行说成已经完成。
- 记录数据库/软件版本、样本数、参数、过滤规则与实际运行状态。出现缺样本、零值、批次差异、失败样本或注释不确定性时，明确报告并停止不可靠的下游推断。
- 不虚构数值或结果。脚本输出、统计表和报告中的样本数与图表必须互相核对。探索性关联及基因潜力不得写成已证实的生态过程或因果结论。

## 分析流程

1. **样本与输入审计**：建立样本清单，确认 FASTQ/分类丰度/功能丰度矩阵的样本列与元数据一一匹配；比较可能重复的数据目录或批次时，以样本 ID、文件校验值及 read 统计交叉判断，不只凭文件名。保留审计结果。
2. **原始 reads 预处理**：如果从 FASTQ 开始，通常先用 `fastp` 去接头、过滤低质量和过短 reads，并生成 HTML/JSON 质控报告。对于存在人、动物、植物或其他已知宿主污染风险的样本，在 fastp 后使用 `bowtie2` 比对宿主参考基因组，提取未比对 reads 供后续分类和功能分析；纯环境样品或无可靠宿主参考时可跳过。记录软件版本、参数、输入/输出 reads 数、宿主比对率和保留率，不能把未执行的去宿主步骤写成已完成。
3. **物种组成与丰度**：在输入允许的分类分辨率下整理门、科、属、种相对丰度。若需要基于分类支系特异性 marker 的物种丰度，可使用 MetaPhlAn；记录 MetaPhlAn 版本、marker 数据库版本、映射参数和 profile 输出。MetaPhlAn 结果通常更接近 marker 覆盖度推断的 organismal relative abundance，但仍受数据库覆盖和 marker 特异性限制。若使用 Kraken2/Bracken，则记录数据库、分类参数及其 read-based 或重新估计的丰度定义；两套方法应分别报告，不直接混合或取平均。生成按时间、地点、处理或用户指定分组的组成图；常见展示为 Top taxa 加 Other，同时提供用于制图的表格。相对丰度不是绝对生物量。保留数据库版本与分类不确定性；未命名占位类群不要过度解释。
4. **α 多样性**：至少描述丰富度与均匀度/多样性（例如 Observed、Shannon、Simpson、Pielou），按设计展示样本点和组别。若设计为配对比较，采用配对检验并报告配对数、检验方法、效应方向及 p 值；不显著不能表述为两组等同。检查测序深度/稀释或覆盖度处理，说明丰富度对测序深度敏感。
5. **β 多样性**：至少依据适用的数据构造 Bray–Curtis（丰度）和/或 Jaccard（出现/缺失）距离并作 PCoA/NMDS。图例和形状表达时间、组别或处理。低维图用于展示距离结构；若要声称组间差异，另做适当置换检验（如 PERMANOVA）并检查组内离散度及设计限制。
6. **功能分析**：核验所用功能工具与核酸/蛋白数据库版本兼容性；按可用资源选择搜索路径并记录偏离默认流程的原因。汇总基因家族、酶、KO、通路或其他功能单元，按样本计算适当的 CPM/相对丰度并保留原始输出及映射来源。根据用户的研究问题定义功能集合、标记规则和检出阈值；若某项表型或过程不能由单一标记直接推断，应明确称为“候选功能/功能潜力”。
7. **物种–功能关系**：先匹配样本并说明过滤条件，再计算适合小样本探索的相关性（例如 Spearman），对多重检验做 BH/FDR 校正，报告检验总数、有效样本数、ρ、原始 p 与校正 q。将相关图明确标注为探索性。讨论时间、组别、批次、组成型数据和极端样本导致的混杂；相关性不证明物种直接执行功能。
8. **R 可视化**：如用户要求 R，使用 R 脚本生成图；核查 R 及 `ggplot2` 等实际依赖，不因假定包已安装而静默失败。依照用户选择的期刊风格设色、字号、主题和尺寸。图例置于图外且不遮挡数据；面板/样本排序与分析设计一致。保留生成图的 R 脚本和输入汇总表。
9. **报告交付**：用户要求时生成 Markdown、HTML、PDF（PDF 支持中文且图像正常嵌入/链接）。报告交代目的、材料/方法、预处理、结果、逐图解读、结论、限制和后续验证建议。每张图的说明至少指出可见的主要模式、关键定量结果、可支持的结论以及不能推出什么；结果必须来自实际表格/统计输出。报告中的图路径应可用，HTML 不得引用缺失的相对资源。

### 物种丰度工具的输出

使用 MetaPhlAn 时，建议保留：

- 每个样本的 MetaPhlAn profile 表；
- 合并后的样本×分类单元丰度矩阵；
- marker 数据库和软件版本记录；
- 可复用的中间比对文件（如保留）及运行日志；
- 分类单元检出率和低丰度过滤表；
- 门、科、属、种水平的汇总表和图。

使用 Kraken2/Bracken 时，单独保留 Kraken2 分类报告、Bracken 丰度表和数据库信息。若同时运行两种方法，应分别建立结果目录和图表，并在报告中说明差异来源。

## 重点解释边界

- HUMAnN DNA 注释代表基因/通路潜力，不代表转录、蛋白表达、酶活或实际污染物去除速率；“检出”不等于通路完整。
- MetaPhlAn 的 marker-based 丰度估计与 Kraken2/Bracken 的 read-based 丰度不是同一个统计量；应比较检出情况、分类层级和趋势，不把两者数值直接合并。
- MetaPhlAn 仍然是相对丰度方法，不提供绝对细胞数；marker 缺失、数据库覆盖不足、近缘物种共享 marker 或菌株差异都可能导致漏检或偏差。
- 整体表型、理化指标或生态过程通常不能由某个单基因直接读出；应把筛选标记表述为与目标过程相关的候选功能，并说明注释歧义。
- 一个过程可能由多步通路及多个分类群共同完成。单个 KO、EC、基因家族或酶的出现不能独自证明完整过程、活性或功能归属。
- 小样本配对设计统计效能有限；批次若与时间、地点、处理或分组混杂，应在结果和结论中显式提示，并建议增加独立重复及与研究问题相关的表型测量，必要时用转录组、蛋白组、代谢组或酶活验证。

## 完成前核查

- 确认实际纳入样本与元数据完全匹配，分组/日期顺序正确。
- 若使用 FASTQ，确认 fastp 报告、清洗 reads 和（如适用）bowtie2 去宿主统计均已生成，并明确记录是否跳过宿主去除及原因。
- 若使用 MetaPhlAn，确认每个样本都有 profile、版本和 marker 数据库记录，并区分 MetaPhlAn 与 Kraken2/Bracken 的丰度定义。
- 每个目标分析均有可追踪输出；对缺失步骤说明原因，不把未完成写成完成。
- 图数、表格数值、正文结论一致；每张图均有实质性文字解读。
- 检查报告格式能打开，PDF 页数/图像合理，HTML 资源完整，输出确实位于用户要求的目录。
## Figure 4 compositional microbiome analysis

When a user requests a publication-style Figure 4 based on MetaPhlAn species relative abundance, read and follow [references/figure4_compositional_workflow.md](references/figure4_compositional_workflow.md). This module covers CLR preprocessing, alpha diversity, CLR-PCA, paired CLR effects, random-forest feature heatmaps, representative species plots, paired tests, and BH-FDR correction. Use real metadata for donor, treatment, and replicate labels; never infer labels from sample-number ranges. Record separate selection rules for C, D, and E because their rankings answer different questions.

## 已安装服务器资源与复用规则

共享服务器：`172.16.100.7`。运行前检查绝对路径；已有工具、索引和数据库直接复用，禁止重复安装、重新下载或覆盖。

- fastp、Kraken2、Bracken、bowtie2：`/home/licun/miniconda3/envs/metagenome_env/bin/`
- Kraken2/Bracken 数据库：`/home/liuhongbin/database/metagenome/kraken2_GTDB226`
- MetaPhlAn：`/home/licun/miniconda3/envs/ksb_humann2/bin/metaphlan`
- MetaPhlAn 数据库：`/home/licun/Data/PolyMicro20261113/Results/databases/metaphlan/extracted`，索引 `mpa_vJan26_CHOCOPhlAnSGB_202605`
- MetaPhlAn 备用库：`/home/liuhongbin/database/metagenome/humann3/metaphlan_databases`
- HUMAnN3：`/home/licun/miniconda3/envs/ksb_humann2/bin/humann*`；数据库 `/home/liuhongbin/database/metagenome/humann3/humann3`
- R：`/home/licun/miniconda3/envs/r_ksb/bin/Rscript`

默认项目根目录为 `/home/licun/Data/PolyMicro20261113`，结果、脚本和日志分别放入 `Results/`、`Results/scripts/`、`Results/logs/`。启动前检查 SSH、输入、metadata、工具版本、数据库索引、权限和已有状态表；成功样本跳过，失败步骤重跑。路径缺失或版本不兼容时先报告，未经用户同意不得下载或安装。
