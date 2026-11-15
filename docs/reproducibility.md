# Reproducibility

## Runtime

The notebooks use a Kaggle Python kernel with CPU and internet access to GEO, PyPI, CRAN, Bioconductor and the MSigDB distribution used by msigdbr. Saved run versions include Python 3.13.18, NumPy 2.1.3, pandas 2.3.3, SciPy 1.16.3, h5py 3.14.0 and AnnData 0.13.4. Plotting also uses Matplotlib and IPython display. The notebooks install AnnData where needed and record package versions in their audits. Avoid changing the Kaggle image between stage runs.

Stages 05 and 06 invoke `Rscript` from Python. The notebooks use R 4.6.1, Bioconductor 3.23 and edgeR 4.10.7. Stage 06 also requires fgsea 1.38.0 and msigdbr 26.1.1. The setup cells use isolated R libraries and check the versions. If a current repository serves a different release, supply the recorded version in the isolated library before executing the setup cell. The [Stage 05 R session](stage05_r_session.txt) and [Stage 06 R session](stage06_r_session.txt) record the original Linux platform and packages.

## Execution order and checkpoints

Import each GitHub notebook into its own Kaggle session. Run from a fresh kernel and save the notebook version with its output files before starting the next stage. Attach the indicated saved runs under **Add Input**. Keep the `kang_ifnb/` directory layout intact. Files produced within a run go under `/kaggle/working/kang_ifnb`; attached files are read from `/kaggle/input`. Attach one copy of each required checkpoint, since the lookup cells fail on duplicates.

| Stage | Attach these completed stages | Outputs passed downstream |
| --- | --- | --- |
| 01 | None | Six GEO files under `data/raw/`, `results/audit/cell_mapping.tsv.gz`, `raw_count_checks.tsv` and `source_reference.json`. |
| 02 | 01 | `data/processed/ifnb_batch2_raw_counts.h5ad` and `results/audit/stage02_anndata_roundtrip.json`. |
| 03 | 02 | `data/processed/ifnb_cd14_candidate_qc.h5ad` and `results/audit/stage03_cd14_h5ad_roundtrip.json`. |
| 04 | 03 | `data/processed/ifnb_cd14_pseudobulk_unfiltered.h5ad`, `ifnb_cd14_pseudobulk_gene_filtered.h5ad` and `results/audit/stage04_pseudobulk_roundtrip.json`. |
| 05 | 02, 03, 04 | `results/audit/stage05_final_checkpoint.json`, the primary and QC95 DE tables, LODO tables and the TMM DGEList. |
| 06 | 04, 05 | `results/pathway/` tables and audit files, plus `results/figures/`. |

The notebooks contain historical SHA-256 checks for selected intermediate H5AD files and primary results. For exact replay, attach the archived checkpoints recorded by those checks. The source runs were Stage 01 v3 (366625672), Stage 02 v1 (366649612), Stage 03 v1 (366696632), Stage 04 v1 (366873844), Stage 05 v1 (366897451) and Stage 06 v3 (369939263). Access to the original private Kaggle runs requires the owner's permission.

A fresh input reconstruction begins in Stage 01 and may produce different serialized H5AD bytes, for example if the runtime or embedded provenance changes. If a historical H5AD pin differs, verify the regenerated artifact against its producing run's manifest and compare count values, cell and Ensembl IDs, sample order and cohort totals against the notebook evidence before substituting the regenerated intermediate hash in a working copy. The checksums for the original published DE and GSEA tables are listed below.

Stage 04's expression filter uses raw pseudobulk CPM. The cutoff corresponds to 10 counts at the median library size, must be met in at least eight samples, and requires at least 15 counts across all samples. Stage 05 checks the selection against edgeR `filterByExpr` using condition groups. Stage 06 ranks all tested genes by `sign(logFC) * sqrt(F)` and tests Hallmarks with 15–500 matched genes, one process, weight 1, `eps=0`, and seed 20261113.

## Verify the distributed outputs

On a system with GNU coreutils, run this from the repository root.

```sh
sha256sum -c docs/checksums.sha256
```

The main entries are exact byte copies of the Kaggle outputs under shorter filenames.

| Repository file | Original Kaggle filename | SHA-256 |
| --- | --- | --- |
| `results/differential_expression.tsv` | `stage05_paired_qlf_all_genes.tsv` | `48e26a2dc3a08b4273eb4636696ac372a672e14329aa9e1c7c92c756f6813a1f` |
| `results/hallmark_gsea.tsv` | `stage06_hallmark_fgsea_refactored.tsv` | `b238825193cf8d24a7e61ff1a6cee4995471c2f0074252e7867e605c5ac2e35d` |

The [Stage 05 checkpoint](stage05_checkpoint.json), [Stage 06 checkpoint](stage06_checkpoint.json), [MSigDB source audit](stage06_msigdb_source.json) and [donor expression audit](stage06_donor_expression.json) retain provenance from the source runs. The portfolio notebooks retain selected outputs from those runs. During packaging, Stage 01 was rerun with a local output path. All six GEO file hashes and the regenerated sparse-count audit matched the archived checks. All notebooks passed Jupyter schema and Python syntax checks. The archived tables were independently checked for significant-gene counts, BH adjustment, QC sensitivity, Hallmark leading-edge membership and paired-donor expression. The saved edgeR and fgsea results were reused.
