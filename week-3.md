---
layout: post
title: "Week 3 - Covariance alignment and R comparison work"
date: 2026-06-14
tags: [gsoc, pysal, spatialml, spml, gwpca, r, gwmodel, testing]
permalink: /week-3/
---

After the first implementation landed on my branch, the next step was checking whether the numerical behavior matched the established R reference as closely as possible. This week was mostly about reducing uncertainty in the math rather than adding new surface-level features.

## Commits This Week

- [`976f035`](https://github.com/FirePheonix/gwlearn/commit/976f035bd6411d9f55ec5dd5c4b10355c9cddfe8) - `replace-covariance-with-numpy-cov`
- [`c1761ef`](https://github.com/FirePheonix/gwlearn/commit/c1761ef1f9151c9a5ba3e44b20dbf0d51f3a8e6f) - `cv-score-redundancy`

## What I Worked On

- Replaced the covariance path with a `numpy`-based implementation to make the local weighted PCA behavior more stable and easier to reason about.
- Compared the implementation against `GWmodel::gwpca` from R to check whether the local components and local explained-variance proportions were lining up.
- Cleaned up redundant `cv_score` logic while keeping the decomposition workflow compatible with the package's model-selection machinery.

At this stage, the interesting result was that the main interpretable outputs were already lining up well with the R implementation:

- local component directions matched up to the usual PCA sign flips
- local explained variance proportions matched the reference closely

The one thing that did not match exactly yet was the raw local eigenvalue scaling. That looked more like a covariance-normalization difference than a failure in the actual local component structure, so it became a good target for follow-up refinement instead of a reason to back out the implementation.

## Why This Week Mattered

Week 2 proved the estimator could run. Week 3 was where I started proving that the implementation was numerically credible and not just functionally complete. For a method like GWPCA, that validation work is just as important as the API itself.

[<- Back to index]({{ "/" | relative_url }})
