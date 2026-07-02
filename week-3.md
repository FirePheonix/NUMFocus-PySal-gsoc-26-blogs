---
layout: post
title: "Week 3 - Covariance alignment, numerical credibility, and first real R comparison"
date: 2026-06-14
tags: [gsoc, pysal, spatialml, spml, gwpca, r, gwmodel, testing]
permalink: /week-3/
---

After getting the first implementation running, the next question became much more serious:

**Is the math actually lining up with an established reference implementation?**

This week was much less visible than Week 2, but in some ways more important. If GWPCA is going to be useful in scientific Python, then "it runs" is not enough. It has to behave in a way that is numerically defensible and comparable to what spatial analysts already trust.

## Commits This Week

- [`976f035`](https://github.com/FirePheonix/gwlearn/commit/976f035bd6411d9f55ec5dd5c4b10355c9cddfe8) - `replace-covariance-with-numpy-cov`
- [`c1761ef`](https://github.com/FirePheonix/gwlearn/commit/c1761ef1f9151c9a5ba3e44b20dbf0d51f3a8e6f) - `cv-score-redundancy`

## The Core Focus

The main goals this week were:

- tighten the weighted covariance path
- reduce implementation-specific ambiguity
- compare outputs against **`GWmodel::gwpca`** in R
- make sure cross-validation logic was not doing unnecessary or duplicated work

At this stage I stopped thinking of the code as "my implementation" and started thinking of it as "something that needs to survive comparison."

## Replacing the Covariance Path

One of the biggest technical steps this week was replacing the custom covariance path with a `numpy`-based approach. That mattered for two reasons:

1. it reduced custom numerical behavior that would be hard to justify during review
2. it made it easier to reason about whether differences with R were about the algorithm or about my own covariance implementation details

When you are comparing two PCA implementations, tiny differences can come from many places:

- centering
- normalization conventions
- weighting conventions
- eigenvalue scaling
- sign orientation

The less unnecessary custom code sits in the middle of that, the easier it is to interpret any mismatch honestly.

## First R Comparison Observations

This was also the week where I started doing careful comparisons against `GWmodel::gwpca` in R.

The encouraging result was that the **interpretable structure** already looked very good:

- local component directions were matching, up to the standard arbitrary sign flips you always get in PCA
- local explained-variance proportions were also aligning closely

That was a big relief, because those are the quantities people actually use to interpret GWPCA output.

At the same time, I noticed that the **raw local eigenvalue scaling** was not matching exactly. That was interesting rather than alarming. It did not look like the component geometry was wrong. It looked more like a normalization or covariance-scaling difference between implementations.

That distinction matters a lot. If the component directions are wrong, your implementation may be fundamentally off. If the raw eigenvalue magnitudes differ while the loadings and variance ratios line up, you are usually looking at a much narrower issue.

## Cleaning Up CV Logic

I also cleaned up redundant `cv_score` logic this week. Cross-validation in geographically weighted models is easy to make messy, because there is already a lot happening in each local fit. I wanted the decomposition path to integrate with bandwidth-search and model-selection machinery without carrying confusing duplicate logic.

This was less glamorous work, but it made the estimator easier to reason about and easier to review later.

## What Changed Mentally

This week shifted my confidence in a very specific way.

In Week 2, the question was:

> Can I make GWPCA work inside the package?

In Week 3, the question became:

> Can I justify the implementation scientifically when someone asks why it should be trusted?

That is a different standard, and it changed how I approached both testing and code cleanup.

## Main Outcome

By the end of the week, I had much stronger evidence that the implementation was on the right track:

- the local PCA structure looked credible
- the R comparison was already largely encouraging
- the remaining mismatch looked narrow enough to investigate systematically

That gave me a clear next step for the following weeks: keep reducing ambiguity until the implementation, tests, and API all told the same story.

[<- Back to index]({{ "/" | relative_url }})
