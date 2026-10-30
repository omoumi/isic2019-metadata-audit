# ISIC 2019: metadata audit

Audit of the metadata that comes with the ISIC 2019 skin-lesion images: how complete it is, whether missingness carries label information, and which non-image shortcuts exist. No image model is trained here.

![Example images](sample_grid.jpg)

Sample images are shown under CC-BY-NC 4.0; see [samples/ATTRIBUTION.md](samples/ATTRIBUTION.md).

## Key numbers

| Item | Value |
|---|---|
| Images | 25,331 |
| Groups used for CV (group level: lesion) | 13,927 |
| Lesions | 11,847 |
| Metadata fields audited | age_approx, anatom_site_general, sex |
| Rows with every field present | 22,480 (88.8%) |
| Rows with core fields present (age_approx, anatom_site_general, sex) | 22,480 (88.8%) |
| Smallest class in the model-based tests, lesions with core fields | 108 |
| Highest missing rate of any field | 10.4% |
| Balanced-accuracy gain over null, missing indicators only | 0.017 [0.011, 0.023] |
| Balanced-accuracy gain over null, acquisition only | 0.077 [0.061, 0.092] |
| Balanced-accuracy gain over null, image size only | 0.076 [0.063, 0.088] |
| Balanced-accuracy gain over null, image border / file statistics | 0.141 [0.128, 0.154] |
| Balanced-accuracy gain over null, core metadata values | 0.160 [0.129, 0.192] |
| License | CC-BY-NC 4.0 |

Gains are over the lesion permutation null (chance level), with 95% intervals in brackets.

### Sensitivity to the permutation scheme

Balanced-accuracy gain over the two nulls. Lesion null: one label per lesion is shuffled. Size-matched null: whole lesions swap labels with lesions of the same size, which keeps the link between lesion size and label. Gap = difference of the two null means, with its standard error. A gap well above about 2 SE suggests that part of the signal runs through how many images a lesion has.

| Feature set | Gain, lesion null | Gain, size-matched lesion null | Gap | SE of gap | Null used for the test |
|---|---|---|---|---|---|
| A_missing_indicators | 0.017 | 0.007 | 0.010 | 0.001 | lesion |
| B_acquisition | 0.077 | 0.041 | 0.036 | 0.001 | lesion |
| C_image_size | 0.076 | 0.056 | 0.020 | 0.001 | lesion |
| D_acquisition_plus_indicators | 0.073 | 0.039 | 0.034 | 0.001 | lesion |
| E_values_with_missing | 0.156 | 0.148 | 0.008 | 0.002 | lesion |
| G_core_values_complete_core | 0.160 | 0.159 | 0.002 | 0.002 | lesion |
| I_image_border_stats | 0.141 | 0.111 | 0.030 | 0.001 | lesion |

## Figures

### Class counts

Images per class, and how many remain when rows with missing metadata are dropped.

![class_counts.png](figures/class_counts.png)

### Missing by class

Missing rate of each metadata field within each class.

![missing_by_class.png](figures/missing_by_class.png)

### Missing patterns

Which fields are missing together (UpSet-style).

![missing_patterns.png](figures/missing_patterns.png)

### Association

Association of each field's missingness with the label and with the acquisition variable, next to the association of the observed value with the label.

![association.png](figures/association.png)

### `age_approx` by class

Distribution of `age_approx` within each class.

![age_approx_by_class.png](figures/age_approx_by_class.png)

### Missing vs other fields

Association between the missingness of each field and the observed values of the other fields. A value near 0 suggests little association with that field (as measured on these categories and quintiles); MCAR also needs no dependence on the label (see the association figure). A clearly positive value is evidence against MCAR; observed data cannot tell MAR from MNAR.

![missing_vs_other_fields.png](figures/missing_vs_other_fields.png)

### Missing vs other fields within label

The same check inside each class. A cell that is strong in the previous figure but weak here may run through the label (for example, age differs by class and missingness differs by class).

![missing_vs_other_fields_within_label.png](figures/missing_vs_other_fields_within_label.png)

### Acquisition by class

Share of each class coming from each acquisition group. A class that comes from one group only could be recognised by acquisition rather than by the lesion.

![acquisition_by_class.png](figures/acquisition_by_class.png)

### Resolution by class

Image resolution by class. Resolution can leave traces (resampling, compression) that a CNN may pick up even after resizing, so it is a possible shortcut.

![resolution_by_class.png](figures/resolution_by_class.png)

### Image stats

Colour, border, frame and compression statistics of the images by class. Border and file statistics mostly come from the camera and the frame, but a large lesion can reach the border and file size depends on texture, so the border test may overstate the pure camera / frame shortcut.

![image_stats.png](figures/image_stats.png)

### Shortcut CV

Balanced accuracy of a gradient-boosting model that sees only the listed features, compared with the same pipeline on permuted labels.

![shortcut_cv.png](figures/shortcut_cv.png)

### Complete-case bias

Class mix and field values in all images, in rows where every field is present, and in rows with some field missing. Experiments that start from complete rows and then remove metadata inherit any difference shown here.

![complete_case_bias.png](figures/complete_case_bias.png)

### Group size

Images per CV group (patient, lesion or image) by class, and the missing rate of each field by group size. When both differ, missingness could predict the label through group size; the gap between the two nulls in the table above gives an indication of that path.

![group_size.png](figures/group_size.png)

### Feature importance

Permutation importance on held-out folds: the drop in balanced accuracy when one feature is shuffled. Left: the missing-indicator model; right: values plus indicators. Correlated features share credit, so a low bar does not mean no signal.

![feature_importance.png](figures/feature_importance.png)

### Missingness confusion

Out-of-fold predictions of the model that sees only which fields are missing. A bright off-diagonal cell means two classes have the same missingness pattern; a bright diagonal cell means much of that class is recognised from missingness alone.

![missingness_confusion.png](figures/missingness_confusion.png)

### Categorical by class

Distribution of each categorical field within each class (observed values only; the 7 most common categories, the rest grouped as other).

![categorical_by_class.png](figures/categorical_by_class.png)

### Null distributions

Balanced accuracy of the same pipeline on permuted labels, for both permutation schemes, with the real-label result as a vertical line. With few permutations the histograms are coarse.

![null_distributions.png](figures/null_distributions.png)

### Missingness mechanism

AUC of predicting that a field is missing, from the other fields, acquisition and group size (a), and with the label added (b). AUC(a) clearly above 0.5 is evidence against MCAR; it cannot tell MAR from MNAR. The gain of (b) over (a) may overstate the dependence on the label, because the label can stand in for the part of the other fields the model did not learn; label R2 shows how much of the label the other fields predict, and the higher it is, the more of the gain may be that substitution. p-values: conditional randomization test, and whole lesions swapping labels within strata of images per lesion x acquisition x age band (no age band when age itself is the target); both are Holm-adjusted, and a claim needs both. A field without a claim may still depend on the label: the power of this rule is limited.

![missingness_mechanism.png](figures/missingness_mechanism.png)

### Complete-case SMD

Standardized mean difference between the rows used for missingness-injection experiments (all fields complete, core fields complete) and all images, and the other images. Categorical fields, the class mix and acquisition use the multi-category SMD (Yang and Dalton 2012). |SMD| > 0.1 is used as the mark of imbalance, a common convention (Austin 2009) rather than a statistical test. Fields are compared on observed values only.

![complete_case_smd.png](figures/complete_case_smd.png)

## Method

- Missing-like tokens (`unknown`, `na`, empty strings, ...) are converted to missing before counting.
- Associations use bias-corrected Cramer's V (Bergsma 2013); 95% intervals come from a bootstrap that resamples whole lesions. Numeric fields use the Kruskal-Wallis epsilon squared, H / (n - 1) (Tomczak and Tomczak 2014). The chi-square and Kruskal-Wallis p-values in the tables are labelled `p_naive`: they treat images as independent, while images of one lesion are not, so they come out too small; the effect sizes and their group-bootstrap intervals are the quantities to read.
- Shortcut tests: `HistGradientBoostingClassifier` (balanced class weights), lesion-grouped stratified 5-fold CV, balanced accuracy. Nulls are the same pipeline on permuted labels. The lesion null gives one label per lesion to all of its images; its mean is the chance level every gain is measured from. The null used is in `test_null`. p-values are (1 + #null >= observed) / (1 + #permutations) (Phipson and Smyth 2010), computed from the first CV split (the nulls use single splits too), and Holm-adjusted (Holm 1979) over the feature sets of this dataset, image-derived reference sets excluded (`p_perm_holm`). With 200 permutations of the test null the smallest attainable p is 0.00498.
- The reported balanced accuracy is the mean of 5 grouped CV runs with different fold assignments. Its 95% interval, and the interval of the gain over the null, use the corrected repeated-CV variance (1/J + n_test/n_train) * S^2 over the J fold scores (Nadeau and Bengio 2003; Bouckaert and Frank 2004) with a t quantile; the gain interval also includes the noise of the null mean. This correction is approximate; its coverage was close to 95% in the toolkit's simulations.
- The image-size feature set includes the file size in bytes, which also depends on the image content (texture, compression), so part of its signal may come from the lesion itself rather than from the camera.
- One model class with fixed settings is used for the shortcut tests, so a feature set without a significant gain may still carry signal that another model would find; a significant gain shows an association with the label, not its cause (e.g. an acquisition source may differ from the others in its patient population, not only in its images).
- Within-lesion checks use only lesions with several images: conflicting values, and fields missing for some images of a lesion but present for others (if never, missingness is decided per lesion). Age can legitimately differ between visits.
- 2 confirmed duplicate pair(s) carry different labels on the two copies. Which label is right is unknown, so their 4 images are left out of the model-based tests and their permutation units are not merged; descriptive tables keep them.
- Missingness-mechanism p-values are Holm-adjusted over 5 tested rows with 125 draws, so the smallest attainable adjusted p is 0.0397.
- Missingness mechanism: for each field, gradient boosting predicts that the field is missing from the other fields, the acquisition variable and the group size, without and with the label. The label's contribution is tested with a conditional randomization test (Candès et al. 2018), where labels are redrawn from a model of the label given the other inputs, and with a test where whole lesions swap labels within strata. A field's missingness is called label-dependent only when both tests reject after Holm adjustment. In the toolkit's simulations this rule claimed dependence in about 4% of runs without any, and in about 7% when the missingness depended on a label-related field; its power was limited, so a field that is not called label-dependent may still be (validation/README.md).
- 11 image pairs were confirmed as duplicates (see tables/confirmed_duplicates.csv); the lesions they link were merged before cross-validation (13,931 -> 13,927 groups), so a duplicate cannot sit in a training and a test fold at the same time.
- All tables are in [`tables/`](tables) and every number above is in [`summary.json`](summary.json).

## References

- Austin PC. Balance diagnostics for comparing the distribution of baseline covariates between treatment groups in propensity-score matched samples. Stat Med. 2009;28(25):3083-3107. doi:10.1002/sim.3697
- Bergsma W. A bias-correction for Cramér's V and Tschuprow's T. J Korean Stat Soc. 2013;42(3):323-328. doi:10.1016/j.jkss.2012.10.002
- Bouckaert RR, Frank E. Evaluating the replicability of significance tests for comparing learning algorithms. In: Advances in Knowledge Discovery and Data Mining (PAKDD 2004). Lecture Notes in Computer Science, vol 3056. Springer; 2004:3-12. doi:10.1007/978-3-540-24775-3_3
- Candès E, Fan Y, Janson L, Lv J. Panning for gold: 'model-X' knockoffs for high dimensional controlled variable selection. J R Stat Soc Series B Stat Methodol. 2018;80(3):551-577. doi:10.1111/rssb.12265
- Holm S. A simple sequentially rejective multiple test procedure. Scand J Stat. 1979;6(2):65-70.
- Nadeau C, Bengio Y. Inference for the generalization error. Mach Learn. 2003;52(3):239-281. doi:10.1023/A:1024068626366
- Phipson B, Smyth GK. Permutation p-values should never be zero: calculating exact p-values when permutations are randomly drawn. Stat Appl Genet Mol Biol. 2010;9(1):Article 39. doi:10.2202/1544-6115.1585
- Tomczak M, Tomczak E. The need to report effect size estimates revisited. An overview of some recommended measures of effect size. Trends Sport Sci. 2014;1(21):19-25.
- Yang D, Dalton JE. A unified approach to measuring the effect size between two groups using SAS. SAS Global Forum 2012; paper 335-2012.

## Notebook

The notebook that produced these files is in [`notebooks/`](notebooks).

The metadata and ground-truth files and the images were read from a copy of the ISIC 2019 training set that another user shared on Kaggle (`alifshahariar/isic-2019-dataset-full`), not downloaded from the ISIC challenge page; the md5 of each file is in `summary.json` (`inputs`), so the copy can be checked against the official files.

## Data source and license

- Source: https://challenge.isic-archive.com/data/#2019
- License of the images and metadata: [CC-BY-NC 4.0](https://creativecommons.org/licenses/by-nc/4.0/) - non-commercial use only.
- This repository does not redistribute the dataset; only aggregate tables, image ids and a few resized sample images with attribution.

Dataset references:

- BCN_20000 Dataset: (c) Department of Dermatology, Hospital Clínic de Barcelona.
- HAM10000 Dataset: (c) by ViDIR Group, Department of Dermatology, Medical University of Vienna; https://doi.org/10.1038/sdata.2018.161
- MSK Dataset: (c) Anonymous; https://arxiv.org/abs/1710.05006; https://arxiv.org/abs/1902.03368
- Tschandl P, Rosendahl C, Kittler H. The HAM10000 dataset, a large collection of multi-source dermatoscopic images of common pigmented skin lesions. Sci Data. 2018;5:180161. doi:10.1038/sdata.2018.161
- Codella NCF, Gutman D, Celebi ME, et al. Skin lesion analysis toward melanoma detection: a challenge at the 2017 International Symposium on Biomedical Imaging (ISBI), hosted by the International Skin Imaging Collaboration (ISIC). arXiv:1710.05006. 2017.
- Hernández-Pérez C, Combalia M, Podlipnik S, et al. BCN20000: dermoscopic lesions in the wild. Sci Data. 2024;11:641. doi:10.1038/s41597-024-03387-w

## License of this repository

- Tables, figures and text: CC BY-NC 4.0 ([license text](https://creativecommons.org/licenses/by-nc/4.0/)). Chosen to match the non-commercial terms of the dataset.
- Sample images keep the license of the original image (see samples/ATTRIBUTION.md).
- Code that produced these files: [skin-metadata-audit](https://github.com/omoumi/skin-metadata-audit), GPL-3.0-only.
- Details: [LICENSE-CONTENT.md](LICENSE-CONTENT.md).

Software versions: skinaudit 0.1.0, python 3.13.15, pandas 2.3.3, numpy 2.1.3, scipy 1.16.3, sklearn 1.6.1
