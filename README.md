# Alzheimer-s-Disease-Classification-Using-Graph-Theory-Measures

Characterizing brain network connectivity differences in Alzheimer’s disease (AD) using graph‑theory measures derived from RM96‑parcellated resting‑state fMRI, with statistical analysis and an SVM classifier (reported accuracy ≈ 75%, AUC ≈ 0.78).

Key contents
- Project description, methods, and key results (thesis summary).
- Preprocessed artifacts: `fc_matrices.npy` (stacked subject FC matrices), `metadata.csv`.
- Nodewise metric outputs and Cohen’s d tables: `*_nodewise_thr03.csv`, `cohens_d_*.csv`.
- Figures: heatmaps, glass‑brain visualizations, top‑ROI barplots, PCA scatter, ROC and confusion matrix.
- Reproducibility: a detailed run summary and README‑style instructions for reproducing results.

# Characterizing Brain Network Connectivity Differences in Alzheimer’s Disease
Author: Evawn (Vanya) Murzin  
Advisor: Sharon Crook — School of Mathematical and Statistical Sciences  
Reader: Sabiha Mahzabeen

## Abstract
This honors thesis uses resting‑state fMRI from the ADNI‑2 cohort (20 Alzheimer’s disease patients, 20 cognitively normal controls) and the RM96 parcellation to compute 96×96 functional connectivity (FC) matrices. After preprocessing and sparsity thresholding, local and global graph‑theory measures were computed and compared across groups (Cohen’s d). Graph metrics were combined as features in a machine learning pipeline (PCA + SVM) to evaluate classification performance (accuracy ≈ 75%, AUC ≈ 0.78). Results highlight reduced local clustering and local efficiency in frontal and medial‑temporal areas in AD; full code and artifacts are provided separately for reproducibility.

---

## Research questions
1. Can graph‑theory analysis of rs‑fMRI networks characterize connectivity differences between AD patients and matched controls?  
2. Can graph‑theory measures be used as features for machine‑learning models to predict AD diagnosis?

---

## Data & cohort
- Data source: Alzheimer’s Disease Neuroimaging Initiative (ADNI‑2).  
- Subjects: n = 40 total: 20 AD and 20 cognitively normal (CN), age range 70–90, sex‑matched.  
- Imaging: T1 MPRAGE and resting‑state fMRI (Philips Achieva 3T; TR = 3000 ms; 140 volumes).  
- Parcellation: RM96 atlas (82 cortical + 14 subcortical ROIs) producing 96×96 FC matrices.  
- FC exports: comma‑delimited `.txt` files, diagonals exported as NaN, off‑diagonals finite and symmetric (floating noise ≈ 1e‑14).

---

## Methods (high level)
1. Preprocessing: standard CONN/SPM pipeline — realignment, slice timing, segmentation, normalization to MNI space, smoothing, denoising.  
2. Time series extraction: average BOLD signal per RM96 ROI.  
3. Functional connectivity: Pearson correlations between ROI time series, Fisher z transform for normalization.  
4. Thresholding and graph construction:
   - Use absolute connectivity values (both positive and negative correlations treated as connectivity magnitude).
   - Apply a sparsity threshold to remove the weakest edges; example threshold used: 30% (removes weakest 30% of connections).
   - Build undirected, weighted graphs per subject.
5. Diagonal & symmetry handling:
   - Replace NaN diagonals with 1.0 (documented option) for downstream processing.
   - Enforce symmetry via averaging with the transpose to remove tiny floating‑point asymmetries.
6. Graph metrics:
   - Local measures: node strength, local clustering coefficient, local efficiency, betweenness centrality, participation coefficient.
   - Global measures: characteristic path length, global efficiency, modularity, small‑world coefficient.
7. Statistical analysis:
   - Compute Cohen’s d at the ROI level and sector summaries (N(|d|>0.3), mean d, % positive/negative).
   - Visualize top ROIs and sector distributions.
8. Machine learning:
   - Construct feature matrix from local and global graph metrics (and/or summary statistics).
   - Use scaling and PCA for dimensionality reduction where appropriate.
   - Train SVM with hyperparameter tuning via stratified cross‑validation; evaluate with ROC/AUC, confusion matrix, precision/recall/F1.

---

## Process (detailed)
- Data selection & QC
  - Identified ADNI‑2 subjects meeting inclusion criteria (complete T1 + rs‑fMRI, age range, diagnosis labels).
  - Verified imaging headers, TR, and voxel sizes; excluded runs with severe motion or missing volumes; logged exclusions in `metadata.csv`.
- Preprocessing & ROI extraction
  - Standardized preprocessing pipeline (motion correction, normalization to MNI), performed ROI time series extraction using RM96 labels, visually inspected mean time series and ROI masks for coverage issues.
- Iterative analyses & parameter sweeps
  - Explored multiple sparsity thresholds (10–40%) and observed metric sensitivity; compared absolute vs signed FC strategies and ruled on the primary analysis (absolute) while documenting alternatives.
  - Evaluated multiple graph metric sets and reduced redundancy via PCA before ML modeling.
- Model selection & validation
  - Built a stratified CV framework to tune SVM hyperparameters and to estimate generalization performance; examined misclassifications and their relation to QC metrics.
- Documentation & packaging
  - Collected final artifacts (FC stacks, nodewise CSVs, Cohen’s d tables, figures) and documented all parameter choices in `run_summary.json` to support reproducibility.

---

## Key results (summary)
- Network structure: subject and group mean FC matrices maintain modular block structure consistent with functional systems.  
- ROI‑level findings: strongest standardized differences (Cohen’s d) concentrated in frontal and medial‑temporal ROIs; some sensorimotor and subcortical nodes show notable effects for specific metrics.  
- Graph metrics pattern: reduced local clustering coefficient and local efficiency in many ROIs for AD relative to CN, suggesting loss of local specialization and localized integration.  
- Classification: SVM on combined graph features achieved moderate performance (reported accuracy ≈ 75%, AUC ≈ 0.78). The classifier prioritized precision for AD labels, leading to conservative AD predictions and reduced recall.

---

## Figures and tables (to include in defense/thesis)
- Figure: Example subject FC heatmap (96×96).  
- Figure: Mean FC heatmaps — AD vs CN.  
- Figure: Bar plots — Top‑10 ROIs by absolute Cohen’s d for each graph metric.  
- Figure: Glass‑brain / connectome visualizations showing spatial distribution of the top positive and negative Cohen’s d nodes for each metric (axial/lateral/medial views).  
- Figure: PCA scatter plot with nonlinear SVM decision boundary (2‑D visualization for interpretability).  
- Figure: ROC curve for best SVM model and confusion matrix.  
- Table: Top ROI‑level effects for each metric (Cohen’s d).  
- Appendix: Sector‑level summaries of Cohen’s d, per‑ROI lists, preprocessing parameters, and run summary.

Figure captions (examples)
- Example subject FC matrix (96 ROIs) showing symmetric modular structure typical of resting state.  
- Group‑averaged FC matrices (AD left, CN right) highlighting subtle differences in off‑diagonal connectivity strengths.  
- Top‑10 ROIs bar plot (per metric) where red = AD > CN, blue = CN > AD; longer bars = larger standardized difference.  
- Glass‑brain: Top nodes (16 total: top eight positive and top eight negative d) plotted on MNI coordinates to show spatial clustering of effects.  
- PCA + SVM: 2‑D projection of subjects onto first two PCs, shaded regions show SVM classification boundary; misclassified subjects highlighted.

---

## Files & artifacts to include in repo
- `fc_matrices.npy` — stacked NumPy array (n_subjects, 96, 96).  
- `metadata.csv` — subject metadata (filename, subject_id, diagnosis/group, diagonal NaN counts, optional parsed labels).  
- `*_nodewise_thr03.csv` — per‑metric nodewise CSVs (example: `clustering_nodewise_thr03.csv`).  
- `cohens_d_<metric>.csv` — ROI‑level Cohen’s d values per metric.  
- `figures/` — folder containing PNGs for heatmaps, glass‑brain visualizations, barplots, ROC and confusion matrix.  
- `run_summary.json` — summary of pipeline settings and key scores (n_subjects, threshold, best hyperparameters, CV scores, test metrics).  
- (Optional) `svm_best_pipeline.joblib` — saved model artifact.

---

## Reproducibility notes (what to provide)
- A documented notebook/Colab with instructions to:
  - load all comma‑delimited `.txt` FC files (one per subject), validate shape (96×96), record diagonal NaN counts, replace diagonals or preserve as configured, enforce symmetry, and stack into `fc_matrices.npy`;
  - compute nodewise and global graph metrics and save nodewise CSVs;
  - compute Cohen’s d and export `cohens_d_*.csv` files;
  - assemble the ML feature matrix, run scaling/PCA, perform hyperparameter tuning with stratified CV, evaluate model, and save results and figures.
- Exact package versions (requirements.txt) and a short "how to run" in README.
- Clear note about the diagonal replacement choice (NaN→1.0 vs keep NaN) and threshold parameter (sparsity) to ensure reproducibility.

---

## What I learned (detailed)
- Analytical rigor & decisions
  - Document every preprocessing choice: small differences (diagonal handling, symmetrization, threshold) materially influence downstream graph metrics and must be logged.
  - Multi‑threshold sensitivity: network measures can change qualitatively with sparsity; integrated‑density methods are valuable to avoid drawing conclusions from a single threshold.
- Interpretation & neuroscience
  - Network disruptions in AD manifest more strongly in local integration measures (clustering, local efficiency) in frontal and medial‑temporal regions, aligning with known pathology but highlighting network‑level perspectives.
  - Signed vs absolute FC tradeoffs: anticorrelations are not noise—loss of sign information can obscure meaningful patterns.
- Technical & professional skills
  - Practical competency in rs‑fMRI QC, ROI extraction, and graph metric interpretation.
  - Experience with limited‑sample ML best practices (stratified CV, conservative tuning, careful evaluation of overfitting).
  - Improved scientific communication: translating technical findings (Cohen’s d distributions, graph metrics) into figures and explanations that are accessible to non‑specialist committee members.
- Project management & reproducibility
  - Importance of reproducible artifact packaging: well‑documented metadata and run summaries save time during defense and for future reuse.
  - Balancing exploratory analysis with principled reporting: how to present sensitivity analyses transparently while keeping the main narrative clear.

---

## Limitations (explicit)
- Small sample (n = 40) limits statistical power and increases risk of overfitting; results should be treated as exploratory and require validation on larger cohorts.  
- Single fixed sparsity threshold (30%) can bias network topology; multi‑threshold/density integration analyses are recommended.  
- Absolute‑value conversion of FC (ignoring sign) discards anticorrelation information; signed graph analysis could reveal additional effects.  
- Static FC used here ignores temporal dynamics; dynamic FC methods could provide complementary insights.  
- Moderate classifier performance; not suitable for clinical deployment without larger validation and calibration.

---

## Future directions
- Validate and expand on larger samples and external cohorts (other ADNI waves or independent datasets).  
- Multi‑threshold / density integration to reduce threshold sensitivity.  
- Compare signed vs absolute weighting, and test directed/effective connectivity.  
- Integrate multimodal inputs (structural MRI, PET biomarkers, genetics) and test advanced models (XGBoost, ensembles, tailored neural nets).  
- Longitudinal modeling for progression and conversion prediction.  
- Interpretability work (feature importance, SHAP values) to link model decisions to neurobiology.

---

## Broader significance
This project demonstrates that graph‑theory measures from rs‑fMRI can detect moderate, spatially localized differences between AD and CN brains—especially in frontal and medial‑temporal regions—and that these measures can serve as informative features for machine learning approaches. The provided artifacts and documentation aim to support reproducibility and to allow others to extend and validate the approach.

---

## Acknowledgements
- ICON Lab and advisors: Dr. Sharon Crook, Dr. Sabiha Mahzabeen  
- Colleagues and contributors listed in the thesis materials.

---

## Contact
Evawn (Vanya) Murzin — evawnmurzin@gmail.com

Quick start
1. Review the thesis notebook (Thesis_description.md) for methodological details and figure list.
2. Find outputs in the `thesis_outputs/` folder: `fc_matrices.npy`, `metadata.csv`, `cohens_d_*.csv`, figures.
3. If you want to re‑run analysis, follow steps in the notebook’s Reproducibility section.

License & contact
- Author: Evawn (Vanya) Murzin — Honors Thesis (Arizona State University)
- For questions, reproduction help, or collaboration: evawnmurzin@gmail.com


