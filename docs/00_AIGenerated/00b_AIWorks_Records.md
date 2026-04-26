# META_007 - AI Work Records

## Date: 2026-04-26

**LLM:** Claude Code

### Work Performed

1. **Created README.md** — Added project overview with title (🦠 16S Amplicon Sequencing: MiSeq vs NextSeq), user (Genomics Platform), context, goals, data summary, and status, following lab conventions with emoji and structured metadata fields.

2. **Established workflow directory structure** — Set up `workflow/98_notebooks/00_PreProcessing/` folder for preprocessing notebooks.

3. **Created META_007_00a_PreProcessing_MiSeq.qmd** — Comprehensive DADA2 preprocessing notebook for MiSeq with:
   - Automatic Q20 detection: identifies per-sample first position where quality drops below Q20 (forward & reverse), then uses the **median** as truncation length for `filterAndTrim()`
   - Complete DADA2 workflow: error learning, denoising, pair merging, chimera removal
   - Taxonomic classification using SILVA reference
   - Phyloseq object construction with ASV labeling
   - Output: `data/02_phyloseq/META_007_Miseq.rds`

4. **Created META_007_00b_PreProcessing_NextSeq.qmd** — Dual-strategy DADA2 preprocessing notebook for NextSeq with:
   - Skips Q20 detection (not applicable to NextSeq's binned quality scores)
   - Runs DADA2 pipeline with **two error models**:
     - **Standard** `learnErrors()` 
     - **Custom binned** error model using modified `loessErrfun()` (recommended for NextSeq) with monotonic enforcement and weighted loess across binned Q-scores
   - Complete workflow through SILVA taxonomy
   - Phyloseq construction for both models
   - Outputs: `META_007_NextSeq_standard.rds` and `META_007_NextSeq_CustomBinned_DADA2.rds`

**Rationale:** Both notebooks follow the standard DADA2 workflow but account for platform-specific characteristics (Q20-based truncation for MiSeq; binned quality scores for NextSeq). The dual error models for NextSeq allow direct comparison of standard vs. optimized approaches for binned data.

5. **Created META_007_01a_AlphaDiversity_PlatformComparison.qmd** — Comprehensive alpha diversity comparison notebook with:
   - Loads and rarefies all three phyloseq objects to common sequencing depth
   - Computes Observed, Chao1, Shannon, Simpson, InvSimpson metrics
   - Per-dataset boxplots and **paired line plots** showing changes within samples across platforms
   - Paired Wilcoxon signed-rank tests across all three pairwise contrasts with BH correction
   - Per-sample concordance scatter plots (diagonal = perfect agreement)
   - Summary tables of richness/diversity by dataset and metric
   - Outputs: `data/03_alpha_diversity/` (raw diversity table, paired test results)

6. **Created META_007_02a_Abundance_PlatformComparison.qmd** — Taxonomic abundance comparison across ranks (Phylum through Genus) with:
   - Agglomerates and converts to relative abundance at all five ranks
   - Stacked bar plots per dataset highlighting top-N taxa + "Other"
   - Mean relative abundance dodge plots per rank
   - Per-sample × per-taxon concordance scatter plots (3 pairwise contrasts)
   - Per-sample Spearman ρ correlations of full taxon profiles across datasets by rank
   - Paired Wilcoxon tests per taxon × rank × contrast with BH correction for differential abundance
   - Outputs: `data/04_abundance/` (long abundance table, mean per-dataset, per-sample correlations, paired stats)

7. **Created META_007_03a_ASVComparison_PlatformComparison.qmd** — ASV-level comparison with exact + alignment-based matching:
   - Per-ASV master table: sequence, length, total count, prevalence, relative abundance per dataset
   - Length and abundance distributions per platform
   - **Exact-sequence overlap** (full-length + common-prefix) with UpSet plots to handle read-length differences
   - **Alignment-based near-match detection** using `DECIPHER::AlignSeqs` + distance matrix (≥99% identity) for unmatched sequences
   - Combined overlap summary (exact-full, exact-prefix, near-match, total)
   - Identity distribution plots for near-matched ASVs
   - Tests whether shared ASVs are more abundant than platform-specific ones
   - Outputs: `data/05_asv_comparison/` (ASV summary, exact overlap, alignment hits, combined overlap, annotated table)

**Rationale:** The complete analysis pipeline now covers preprocessing quality control, alpha diversity platform concordance, taxonomic composition stability, and ASV-level reproducibility across MiSeq and NextSeq platforms, enabling comprehensive assessment of platform effects vs. biological signal.