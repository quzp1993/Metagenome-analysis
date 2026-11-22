# Figure 4 compositional microbiome workflow

Use this reference when a user requests a multi-panel publication-style microbiome figure based on MetaPhlAn relative abundance data and a paired donor/treatment design.

## Input validation

Required inputs:

- species-level relative-abundance matrix: rows are samples and columns are species;
- metadata containing sample ID, treatment/group, and donor ID when samples are paired;
- explicit experimental contrasts and replicate structure.

Never infer treatment labels from sample-number ranges when metadata are available. Validate the sample-to-metadata join and report missing or duplicated IDs.

## Compositional preprocessing

1. Keep species-level taxa only; do not mix genus, family, and species rows.
2. Check sample sums, detection rate, zero proportion, and low-prevalence taxa.
3. Replace zeros with a documented pseudocount or multiplicative replacement.
4. Apply centered log-ratio (CLR) transformation:

\[
CLR(x_i)=\ln(x_i/g(x))
\]

where `g(x)` is the sample geometric mean. Do not z-score CLR data before PCA unless the analysis explicitly requires it.

## Panel definitions

### A: alpha diversity

Use three independent plots, commonly Shannon, observed richness, and Inverse Simpson. MetaPhlAn relative abundance alone does not provide Faith's PD; use Faith's PD only when a valid phylogenetic tree is available. Show individual points, boxplots, and donor-paired lines. If repeated technical/biological replicates exist within donor and treatment, define and document whether lines use individual observations or donor-treatment means.

### B: CLR-PCA

Run PCA on the CLR matrix. Plot one point per sample, color by treatment, and connect paired donor samples when appropriate. Label PC1/PC2 with explained variance. An overall shift is a multivariate pattern and must not be attributed to a single species without loading analysis.

### C: paired CLR effect ranking

For each taxon and contrast, calculate donor-paired effects, for example:

\[
\Delta CLR=CLR_{treatment}-CLR_{reference}
\]

Use paired Wilcoxon (or a paired model) only when the donor pairing is real. Report effect direction and magnitude separately from p values. BH-FDR is required when multiple taxa are tested. Label this panel as paired CLR effect or paired differential analysis; do not call it LEfSe unless a true LEfSe workflow was run.

### D: random-forest feature heatmap

Use a random forest as an exploratory classifier, not as a significance test. Filter features using a pre-specified abundance/prevalence rule (for example mean relative abundance ≥0.1% or ≥0.5%, depending on the project), record the rule, and avoid changing it after inspecting the ranking. Plot the selected taxa as a CLR/Z-score heatmap plus a feature-importance bar chart sharing the exact same taxon row order and y coordinates. Arrange sample columns by treatment when the figure is intended to compare treatment blocks; retain donor metadata for validation and avoid donor leakage during cross-validation.

### E: representative species plots

Select taxa using an explicit rule, such as:

- high random-forest importance;
- mean abundance above a stated threshold in the target treatment;
- positive paired effect in the target treatment versus reference;
- biological candidates supported by an independent functional analysis.

Show raw relative abundance (%) with boxplots, individual points, and donor-paired lines. Do not use CLR values as the y-axis unless clearly labeled. Add significance annotations only when the paired test, multiple-testing correction, and sample size support them; otherwise show effect sizes and raw/q values in a companion table.

## Interpretation boundaries

- C, D, and E intentionally answer different questions and need not select the same taxa.
- CLR effect is a relative compositional change, not an absolute cell-count change.
- Random-forest importance is predictive contribution, not a p value or causal effect.
- A taxon enriched in a treatment is a response candidate, not proof of direct substrate utilization.
- Claims of direct crystalline-polysaccharide degradation require gene/protein evidence (for example CAZy AA10, GH18, GH19, relevant CBMs) and preferably culture, isotope-tracing, or enzyme validation.

## Required outputs

- combined Figure 4 PNG (high resolution), PDF, and SVG;
- CLR matrix and preprocessing log;
- PCA scores and loadings;
- paired effect table with p values and BH-FDR;
- random-forest importance table and selected feature list;
- representative-species table with the selection rule;
- a reproducible script and README describing inputs, filters, contrasts, and software versions.
