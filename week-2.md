---
layout: post
title: "Week 2 - First GWPCA implementation and notebook demo"
date: 2026-06-07
tags: [gsoc, pysal, spatialml, spml, gwpca, notebook, python, spatial]
permalink: /week-2/
---

This week was the real start of my work around [PR #125](https://github.com/pysal/spatialml/pull/125). The main focus was getting the first end-to-end **GWPCA** implementation into shape, wiring it into the package, and building a notebook that could show the workflow on real data.

## Commits This Week

- [`6678649`](https://github.com/FirePheonix/gwlearn/commit/667864903e03018854383fd80f4d3e80ed7eac42) - `sample-gwpca`

## What I Worked On

- Added the first working geographically weighted decomposition implementation, including the initial `GWPCA` code path.
- Wrote decomposition tests and weighted covariance tests so the implementation was not just exploratory code.
- Added `gwpca_demo.ipynb` as a concrete example using the `geoda.guerry` dataset.
- Connected the work to the existing package API so it could be exercised like the rest of the estimators.

The notebook was important this week because it let me verify that the model was usable beyond unit tests. I used it to compare a global PCA view with the geographically weighted version and to inspect how local principal components changed across space.

## Notebook Snapshot

The demo notebook roughly followed this flow:

```python
variables = ["Crm_prs", "Litercy", "Wealth", "Donatns", "Infants"]
X_scaled = pd.DataFrame(StandardScaler().fit_transform(X), columns=variables)

gwpca = GWPCA(n_components=2, bandwidth=25, fixed=False, kernel="bisquare")
local_scores = gwpca.fit_transform(X=X_scaled, geometry=gdf.centroid)
```

It also printed a quick global PCA baseline before switching to local decomposition:

```text
Global PCA Explained Variance Ratio:
[0.35715985 0.22827994]
```

The notebook also produced the following maps directly from the workflow:

![Guerry dataset geometry used in the notebook]({{ "/assets/week-2/guerry-map.png" | relative_url }})

*Base geometry from the Guerry dataset before running the localized decomposition.*

![Local proportion of variance explained by PC1]({{ "/assets/week-2/local-var-pc1.png" | relative_url }})

*Spatial variation in the proportion of variance explained by the first local principal component.*

![Local loading of Wealth on PC1]({{ "/assets/week-2/local-wealth-pc1.png" | relative_url }})

*How the loading of `Wealth` on the first local component changes across space.*

From there, the notebook highlighted two useful spatial summaries:

- the local percentage of variance explained by the first component
- the local loading of `Wealth` on the first component

That made the notebook a good companion to the implementation work, because it showed the method as something interpretable and not only something tested.

## Files That Moved Most

- `gwpca_demo.ipynb`
- `gwlearn/decomposition.py` in the earlier package layout
- `gwlearn/tests/test_decomposition.py`
- `gwlearn/tests/test_weighted_covariance.py`

[<- Back to index]({{ "/" | relative_url }})
