---
layout: post
title: "Week 6 - GWPCA R Parity and Numerical Stability"
date: 2026-07-01
tags: [gsoc, pysal, spatialml, spml, r, gwmodel, testing, pca]
permalink: /week-6/
---

This week focused on ensuring that the `spml` implementation of Geographically Weighted Principal Component Analysis (GWPCA) perfectly matches the established R package `GWmodel`. It is critical for scientific software to have robust numerical stability and exact parity with widely used reference implementations.

## How the Tests Were Done

To guarantee our implementation is correct, we implemented a rigorous testing pipeline to compare `spml` against `GWmodel`.

1. **R Fixture Generation**:
   We created R scripts (`generate_r_gwpca_fixture.R` and `generate_r_gwpca_adaptive_fixture.R`) that run `GWmodel::gwpca` on a standardized synthetic dataset.
   These scripts extract the exact local component loadings and explained variance proportions, exporting them as high-precision JSON files (up to 8 decimal places).

2. **Python Test Suite**:
   In `spml/tests/test_decomposition.py`, we added a suite of tests under the `TestGWPCANumericalCorrectness` class.
   These tests run our `spml.decomposition.GWPCA` estimator using the identical configurations as the R scripts (same kernel, bandwidth, data, and geometry).
   We then use `numpy.testing.assert_allclose` to strictly verify that our outputs match the R fixtures.

3. **Adaptive vs Fixed Bandwidths**:
   We validated both spatial neighborhood definitions:
   - **Fixed Bandwidth**: Tested with a fixed spatial distance (`bw=5`), asserting that our distance calculations and bisquare kernel weighting match exactly.
   - **Adaptive Bandwidth**: Tested with an adaptive nearest-neighbors approach (`bw=8`, meaning the 8 closest points), verifying that the localized weight matrices are correctly scaled per observation.

4. **Numerical Stability**:
   Beyond parity with R, we added deterministic checks for rank-deficient covariance matrices and CV score reconstruction errors to ensure `spml` handles edge cases without throwing warnings or producing `NaN`s, utilizing safe eigendecompositions (`np.linalg.eigh` + clipping).

## Results Comparison

Below is the side-by-side comparison of the Explained Variance Ratios and Component Loadings between `GWmodel` and `spml`. The results match identically up to 8 decimal places.

### Explained Variance Ratio (Local PV, %)

| Location | GWmodel PC1 | spml PC1 | GWmodel PC2 | spml PC2 |
|---|---|---|---|---|
| 2 | 66.79944007 | 66.79944007 | 28.21802351 | 28.21802351 |
| 3 | 63.64566799 | 63.64566799 | 31.44977619 | 31.44977619 |
| 4 | 61.39397786 | 61.39397786 | 33.10290676 | 33.10290676 |
| 5 | 62.47439538 | 62.47439538 | 32.70199980 | 32.70199980 |
| 6 | 57.79939210 | 57.79939210 | 32.34985527 | 32.34985527 |
| 7 | 68.57077309 | 68.57077309 | 23.21854269 | 23.21854269 |

### Component Loadings (Absolute Values)

*Note: PCA signs are arbitrary, so absolute values are compared.*

| Location | Feature | GWmodel PC1 | spml PC1 | GWmodel PC2 | spml PC2 |
|---|---|---|---|---|---|
| 2 | A | 0.68895623 | 0.68895623 | 0.08114459 | 0.08114459 |
| 2 | B | 0.70852055 | 0.70852055 | 0.13404747 | 0.13404747 |
| 2 | C | 0.15276758 | 0.15276758 | 0.98764712 | 0.98764712 |
| 3 | A | 0.79204176 | 0.79204176 | 0.05003965 | 0.05003965 |
| 3 | B | 0.60487671 | 0.60487671 | 0.19889535 | 0.19889535 |
| 3 | C | 0.08242585 | 0.08242585 | 0.97874239 | 0.97874239 |
| 4 | A | 0.88485219 | 0.88485219 | 0.02065589 | 0.02065589 |
| 4 | B | 0.45039504 | 0.45039504 | 0.29329253 | 0.29329253 |
| 4 | C | 0.11908364 | 0.11908364 | 0.95579957 | 0.95579957 |
| 5 | A | 0.96219121 | 0.96219121 | 0.10256587 | 0.10256587 |
| 5 | B | 0.21097300 | 0.21097300 | 0.30528858 | 0.30528858 |
| 5 | C | 0.17227437 | 0.17227437 | 0.94672019 | 0.94672019 |
| 6 | A | 0.93663008 | 0.93663008 | 0.31174872 | 0.31174872 |
| 6 | B | 0.08511507 | 0.08511507 | 0.64500411 | 0.64500411 |
| 6 | C | 0.33982279 | 0.33982279 | 0.69769795 | 0.69769795 |
| 7 | A | 0.51692673 | 0.51692673 | 0.83387714 | 0.83387714 |
| 7 | B | 0.62819202 | 0.62819202 | 0.21598468 | 0.21598468 |
| 7 | C | 0.58151659 | 0.58151659 | 0.50793655 | 0.50793655 |

## Conclusion

With these rigorous parity tests passing successfully, we can confidently assert that the `spml` GWPCA implementation behaves identically to the R reference, opening the door for spatial data scientists to natively use these powerful local decomposition techniques in Python.

[<- Back to index]({{ "/" | relative_url }})
