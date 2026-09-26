# Code and Data for the Figures

**Paper:** Selection of the cross-correlation coefficient between atmospheric and compressible turbulence in joint-turbulence imaging: a Monte Carlo study based on correlated phase screens.

This package contains every script and data file needed to reproduce Figures 2–13 of the paper, together with the supporting calibration analyses cited in Section 3.2.5. The final figure images exactly as they appear in the manuscript are collected in `figures/`.

**Update (font enlargement):** all figures in `figures/` and all plotting scripts have been updated so that every number and English text element (axis labels, tick labels, titles, legends, colorbars, annotations) is enlarged by +2 pt relative to the original release, for editorial readability. Re-running any script regenerates the enlarged figures exactly; data and numerical results are unchanged.

---

## 1. Environment

- Python 3.10+
- `numpy`, `scipy`, `matplotlib`, `Pillow`

No other dependencies are required. All Monte Carlo runs use fixed seeds, so results are reproducible bit-for-bit (up to BLAS-level floating-point differences).

**Windows note:** the package is platform-independent — all paths are resolved relative to each script (`os.path`), only forward slashes are used, and no console output contains non-ASCII characters (safe for the default GBK code page). Tested end-to-end with Python 3.12 + numpy 2.x + scipy + matplotlib + Pillow; every script was re-executed from a clean copy of this package before release.

**Path note (portable):** no path editing is needed. Every script locates its own folder automatically (`PKG` = the script's folder) and imports the shared library from `../core/`. Keep the folder structure of this package intact and run each script from inside its own folder, e.g. `cd fig03-05_sf_validation && python run1_reproduce_and_validate.py`. All outputs (figures, JSON, npz) are written into the same folder as the script. All data files each script needs — including the few shared caches — are already present in that script's folder.

## 2. Common simulation parameters (used by all scripts)

| Parameter | Value | Meaning |
|---|---|---|
| λ | 1.06 µm | wavelength |
| L_AT | 2000 m | atmospheric-turbulence path length |
| L_CT | 1.0 m | compressible-turbulence (shear-layer) equivalent length, pupil-equivalent |
| L_total | 2001 m | total propagation distance |
| Cn² | 1×10⁻¹⁶ m⁻²ᐟ³ | atmospheric refractive-index structure constant |
| C² | 1×10⁻⁹ | compressible-turbulence strength parameter |
| D₀ | 0.3 m | receiving aperture diameter |
| N × W | 512 × 4.0 m | screen grid points × physical width |
| nsub | 12 | subharmonic compensation order |

## 3. Directory layout and per-figure guide

### `core/` — shared library (required by every figure script)
- `jtcore.py` — phase-screen generation: Kolmogorov and full-spectrum compressible-turbulence (CT) power spectra, structure functions, subharmonic-compensated screen synthesis, and correlated screen-pair generation (uniform coupling η and large-scale shared coupling with coupling length λc).
- `imaging.py` — imaging chain: analytic long-exposure OTF of the paper model (`otf_paper`, including the static Airy OTF with cutoff fc = D₀/(λL_AT) = 141.5 cycles/m), Monte Carlo long-exposure OTF estimator, OTF-to-image application, and PSNR/SSIM metrics.

### `fig02_screen_samples/` → Fig. 2
- **Script:** `make_paper_figs.py`
- **Data:** none — the two screen pairs (η = 0 and η = 1, seed 42) are synthesized at run time.
- **Output:** `paper_screens.png`.

### `fig03-05_sf_validation/` → Figs. 3, 4, 5 (validation A, B1, B2)
- **Script:** `run1_reproduce_and_validate.py`
- **Data:** none — 40-screen ensembles are synthesized at run time.
- **Outputs:** `valid_A_AT.png` (Fig. 3, AT Kolmogorov screen structure function vs. theory, rms 2.9%), `valid_B1_CT_full.png` (Fig. 4, full-spectrum CT screen vs. spectral integration and Bessel-K closed form, rms 1.5%), `valid_B2_CT_effective.png` (Fig. 5, equivalent CT screen vs. theory, rms 1.9%); numerical records in `run1_results.json`.

### `fig06_otf_mc_vs_analytic/` → Fig. 6
- **Scripts:** `run3_mc_imaging.py` (generates the Monte Carlo OTF caches), then `replot_cached_figs.py` (renders the publication figure from the caches).
- **Data:** `mc_otf_single.npz` (single-branch MC OTF profiles; keys `f`, `at`, `ct`), `mc_otf_jt.npz` (joint MC OTF at η = 0/0.5/1), `run2_results.json` (p(r) curves, used by the replot script), and the two test images `rtt_original.png`, `airplane_original.png` (used by the metric step of `run3_mc_imaging.py`).
- **Output:** `valid_C_OTF_mc_vs_analytic.png`.

### `fig07-08_images_metrics/` → Figs. 7, 8
- **Script:** `run3b_metrics.py`
- **Data:** `mc_otf_jt.npz` (copy from `fig06_otf_mc_vs_analytic/`), the two test images `rtt_original.png` (resolution target) and `airplane_original.png`.
- **Outputs:** `jt_images_mc_vs_analytic.png` (Fig. 7), `metrics_vs_eta.png` (Fig. 8); PSNR/SSIM values in `run3_results.json` and `revision_psnr_fixed_peak.json` (Table 4 of the paper).

### `fig09_p_sim_vs_theory/` → Fig. 9
- **Script:** `run2_correlated_pairs.py`
- **Data:** none — correlated screen ensembles are synthesized at run time.
- **Outputs:** `p_sim_vs_theory.png`; all measured and theoretical p(r) curves in `run2_results.json` (uniform coupling: measured p_sim ≈ η within ±0.1 inside the aperture; large-scale coupling: monotone rising p(r); the structure-function-domain known-answer test η = 1 → 1.000 cited in Section 3.2.1 is also recorded here).

### `fig10_peff_vs_lambdac/` → Fig. 10
- **Script:** `run6_lambdac_sweep.py`
- **Data:** none for the theory curves (it also reads `run2_results.json` — copy from `fig09_p_sim_vs_theory/`).
- **Outputs:** `p_eff_vs_lambdac.png`; sweep data in `run6_lambdac_sweep.json` (effective p vs. coupling length λc, aperture-averaged and weighted values; Table 5).

### `fig11_largescale_otf_test/` → Fig. 11
- **Scripts:** `revision_m450.py` (generates the M = 450 Monte Carlo caches; also reads `mc_otf_single.npz` — copy from `fig06_otf_mc_vs_analytic/`), then `revision_fig11.py` (renders the figure).
- **Data:** `revision_m450_seed11.npz`, `..._seed22.npz`, `..._seed33.npz` — three independent initializations × M = 450 screen pairs each; keys `f` (39 spatial-frequency samples), `u1` (MC joint OTF under the p = 1 hypothesis), `kc_1.0m`, `kc_0.5m` (MC joint OTF under large-scale coupling with λc = 1.0 m / 0.5 m). The per-frequency sample standard deviation σ(f) shown in the figure is computed across the three seeds.
- **Outputs:** `valid_D_largescale_OTF.png`; significance statistics in `revision_fig11_stats.json` (mid-band 20–80 cycles/m: p = 1 deviates from MC by up to 6.5σ (λc = 1.0 m) / 2.5σ (λc = 0.5 m), while the weighted effective p stays within 0.7σ / 1.3σ). The two test images `rtt_original.png` / `airplane_original.png` are also included (needed by `revision_m450.py`). Image-domain fits from the same runs are in `revision_M450.json` (6 estimates = 3 seeds × 2 images: p_eff = 0.697 ± 0.052 for λc = 1.0 m, 0.821 ± 0.092 for λc = 0.5 m; these are the ±0.05 and ±0.09 quoted in Section 3.2.5).

### `fig12_largescale_images/` → Fig. 12
- **Scripts:** `run4_largescale_imaging.py` (generates the degraded images and the joint-OTF caches `mc_otf_jt_kc_*.npz`, shared with Fig. 13), then `run4b_figures.py` (renders the figure; also reads `run6_lambdac_sweep.json` from `fig10.../` and `run5_multiseed.json`, included here).
- **Output:** `jt_images_largescale.png` (4 rows × 6 columns, with shared residual colorbar).

### `fig13_p_selection_summary/` → Fig. 13
- **Script:** `run4b_figures.py` — the same script as Fig. 12; run it inside `fig12_largescale_images/` (all its inputs are already there). It writes `p_selection_method.png` into that folder.
- **Data:** this folder keeps the archival copies of the Fig. 13 inputs and records: `mc_otf_jt_kc_1.0m.npz`, `mc_otf_jt_kc_0.5m.npz` (joint MC OTF caches) and the Table 6 estimate tables `run4_results.json`, `run4_pfit_metrics.json`, `run4_pfit_weighted.json`, `run4_pfit_biascorrected.json`.
- **Output:** `p_selection_method.png` (MC-inverted p_eff(f), theory curve, p = 1 line, aperture mean, weighted value, and the MC image-fit band on one frequency axis).

### `supporting_calibration/` — Section 3.2.5 calibration and audits (no figures)
- `revision_known_answer.py` + `revision_known_answer_M{150,450}_seed{11,22,33}.npz` + `revision_known_answer.json` — the known-answer test of the OTF-domain inversion of Eq. (23): applied to screen pairs whose true cross-correlation is identically 1, the inversion returns 0.858 ± 0.045 (M = 150) and 0.828 ± 0.023 (M = 450) with same-ensemble single-branch OTFs, and 0.740 ± 0.058 / 0.728 ± 0.032 with cross-cache single-branch OTFs (the actual practice of Section 3.2.4). This quantifies the ~15% systematic underestimate of the OTF-domain inversion and shows it does not shrink with ensemble size.
- `run4c_pfit.py` — produces the three p-estimate tables (`run4_pfit_*.json`) of Table 6. Its image-fit step additionally reads the large `img_*.npy` caches; copy them from `fig12_largescale_images/` into this folder before running.
- `run5_multiseed.py` — multi-seed (11/22/33, M = 150) bias-cancelled image fits → `run5_multiseed.json` (basis of the 0.71 ± 0.05 / 0.87 ± 0.09 image-domain estimates).
- `revision_analyses.py` — auxiliary numbers used in the text and appendices (`revision_pth_gap.json`, `revision_rytov.json`, `revision_table3.json`, `revision_appendix_numbers.json`).
- `revision_sigma_audit.py` — audit trail linking the σ-significance statements of Sections 3.2.3–3.2.4 to their raw sources → `revision_sigma_audit.json`.

### `figures/` — final images as they appear in the manuscript
`fig02_*.png` … `fig13_*.png`, one file per figure, named with the figure number used in the paper.

## 4. Reproduction order

The pipeline has a natural dependency order (later stages read caches produced by earlier ones):

1. `run1_reproduce_and_validate.py` → Figs. 3–5
2. `run2_correlated_pairs.py` → Fig. 9 + p(r) theory curves used by Figs. 10 and 13
3. `run3_mc_imaging.py` → Fig. 6 caches
4. `run3b_metrics.py` → Figs. 7, 8
5. `run4_largescale_imaging.py` → Fig. 12/13 image and OTF caches
6. `run5_multiseed.py` and `run4c_pfit.py` → multi-seed fits and Table 6 values
7. `run6_lambdac_sweep.py` → Fig. 10
8. `revision_m450.py` → Fig. 11 caches (M = 450)
9. `revision_fig11.py` and `run4b_figures.py` → Figs. 11, 12, 13
10. `revision_known_answer.py` (optional) → Section 3.2.5 calibration numbers

Runtime (measured on a single core): Fig. 2 ~1 s; Figs. 3–5 ~16 s; Fig. 6 ~3 min; Figs. 7–8 ~1 min; Fig. 9 ~4.5 min; Fig. 10 ~1 min; Fig. 12/13 rendering ~1 min from the included caches. Regenerating the heavy Monte Carlo caches takes longer: Fig. 12 images (run4) ~10 min, multi-seed fits (run5) ~6 min, Fig. 11 caches (revision_m450) ~15 min, known-answer test ~10 min. All cached data products are included, so every figure can be re-rendered directly from the caches without re-running the Monte Carlo stages.

## 5. Data file formats

- `.npz` — NumPy archives; inspect keys with `numpy.load(path).files`. All OTF profiles share the frequency axis stored under key `f` (cycles per meter).
- `.npy` — single NumPy arrays (degraded images, float64 in [0, 1], row-major).
- `.json` — plain-text numerical records (structure-function samples, p(r) curves, fit tables, statistics).
- The test images `rtt_original.png` and `airplane_original.png` are the undegraded targets used throughout.
