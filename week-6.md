---
layout: post
title: "Week 6 - R comparison, reviewer feedback, and turning parity into something testable"
date: 2026-07-01
tags: [gsoc, pysal, spatialml, spml, r, gwmodel, testing, pca]
permalink: /week-6/
---

This week was the most review-heavy and numerically focused part of the GWPCA work so far.

The branch was no longer at the stage where broad implementation decisions were the main problem. The central question had become:

> Can I show, in a way that will stand up to review, that the Python implementation is genuinely aligned with the R reference where it matters?

And just as importantly:

> When it is *not* aligned, can I explain exactly why?

That made this week both technically intense and surprisingly educational.

## What Changed This Week

The biggest difference from earlier weeks was that the work stopped being mainly about adding new decomposition functionality and became much more about **evidence**:

- evidence that the estimator agrees with `GWmodel::gwpca` in R on directly comparable outputs
- evidence that edge cases are handled deliberately rather than accidentally
- evidence that the package API around GWPCA is stable enough to trust
- evidence that review comments have been understood and implemented in a way that improves the branch, not just silences a complaint

This is a very different mindset from "feature development," and I felt that shift strongly while working through the PR.

## R Comparison: What We Actually Observed

The most important thing I want to record from this week is that the R comparison was **not** just a vague "looks close" exercise. It became much more concrete.

When comparing against **`GWmodel::gwpca`**, I observed three different categories of behavior:

### 1. Things that matched cleanly

These were the encouraging parts:

- local component directions matched once you account for ordinary PCA sign indeterminacy
- local explained-variance ratios matched closely
- the localized structure of the decomposition was telling the same story in both implementations
- fixed-bandwidth and adaptive-bandwidth comparisons were both encouraging, which mattered because those two modes exercise different neighborhood definitions
- the locations where the local structure was strongest or weakest were also lining up, so the spatial interpretation was not drifting even when the raw numbers needed closer inspection

This mattered the most scientifically, because those are the outputs users usually interpret first.

Another important practical observation was that the R comparison was not only about one "pretty" location where everything looks good. Once I started checking multiple local neighborhoods, the same pattern kept showing up:

- the first component was emphasizing the same dominant local structure
- the second component was capturing the same residual direction
- the local proportion of variance explained by the first components stayed close enough that the spatial story remained stable

That kind of consistency gave me much more confidence than a single matching table ever could.

### 2. Things that looked different at first, but for understandable reasons

The biggest example here was the **raw eigenvalue scale**.

At first glance, a difference in local eigenvalues can feel scary because eigenvalues are central PCA outputs. But once I dug into the comparison more carefully, it became clear that not every mismatch means "the decomposition is wrong." Some differences are more likely to come from:

- covariance normalization conventions
- weighting details
- whether an implementation reports a quantity before or after a scaling step
- small numerical conventions around local covariance construction

I also had to keep reminding myself that R-vs-Python comparison in PCA is easy to misread if you are not careful. A few things can create the illusion of disagreement even when the decomposition is effectively telling the same story:

- one implementation may orient an eigenvector positively while the other orients it negatively
- local neighborhoods that are nearly degenerate can magnify tiny numerical differences
- adaptive neighborhoods can differ in subtle ways if tie handling or effective bandwidth scaling is not interpreted the same way
- raw local eigenvalues are often a less stable cross-language target than local explained-variance ratios

That is why I gradually stopped treating "every number should match exactly" as the only useful success criterion.

That was a very useful lesson for me. Numerical comparison is not just about spotting differences. It is about classifying them correctly.

### 3. Things that needed to be formalized into tests

This was the real engineering step.

It was not enough to say:

> I checked it against R and it seems okay.

Instead, the branch needed tests that would make the comparison durable. So the work shifted into:

- generating stable reference values
- deciding which outputs are legitimately comparable
- handling PCA sign ambiguity cleanly
- making sure the test strategy itself would survive CI and review

This forced me to define the comparison more carefully than I had at the beginning. In practice, the most defensible direct comparisons became:

- absolute local loadings, because sign flips are arbitrary
- local explained-variance ratios, because they are easier to compare meaningfully across implementations than raw eigenvalue magnitude alone
- both fixed and adaptive neighborhood settings, so the comparison was not accidentally overfit to one mode of operation

I found that once I framed the comparison this way, the branch felt much more scientifically honest.

## A More Honest Version of "Parity"

One thing I became more careful about this week was the word **parity**.

It is tempting to write "perfect parity with R" once you see matching tables for loadings and variance ratios. But the truth is slightly more nuanced, and I think that nuance is important.

What I observed was:

- strong agreement on the main interpretable local outputs
- no evidence that the local decomposition structure was fundamentally off
- some implementation-level details that needed careful framing instead of overclaiming exact equivalence in every raw quantity

That pushed me toward a more defensible standard:

> match the R implementation where the outputs are directly comparable, and write tests that make those comparisons explicit.

I think that is a healthier and more scientific way to present the work.

## Turning R Comparison into Real Tests

This week also included a lot of practical work around making the comparison testable inside the repository.

The broad shape of that effort was:

- build fixed and adaptive reference cases
- compare local loadings and explained-variance ratios
- treat sign flips correctly instead of misclassifying them as errors
- keep the fixtures maintainable enough that they do not become a burden

The fixed-vs-adaptive split mattered a lot here.

For the **fixed-bandwidth** case, I was mainly checking:

- whether the local weighted covariance construction produced the same effective local structure
- whether the bisquare weighting behavior was comparable
- whether the same locations became informative or uninformative under the chosen distance threshold

For the **adaptive-bandwidth** case, the focus shifted slightly:

- whether each focal observation was effectively using the intended local neighborhood
- whether per-location scaling of the neighborhood radius behaved as expected
- whether local results remained aligned once nearest-neighbor based neighborhoods were involved instead of a fixed spatial radius

This made the comparison feel much more complete than simply saying "I ran one R script and one Python test."

The branch eventually moved toward embedding the reference values directly in the Python test module rather than making runtime tests depend on external local R artifacts. That ended up being a much better fit for CI and for keeping the PR self-contained.

I liked that outcome because it kept the spirit of the R comparison without making the test suite fragile or dependent on a side-channel setup.

There was also a very practical reason for this change: if the only evidence of R comparison lives in local scripts and external files, then the PR reviewer has to trust that evidence indirectly. Once the relevant reference values are encoded into the Python tests themselves, the evidence becomes part of the branch instead of background context.

## Reviewer Feedback Was a Big Part of This Week

This was also the week where the review conversation around the PR mattered a lot.

The feedback was not only about numerical comparison. It also touched:

- use of `np.cov` instead of unnecessary custom covariance code
- making decomposition more obviously unsupervised
- keeping shared validation/search logic generic
- exposing outputs as readable pandas objects
- adding tests that check actual values instead of only shapes

Working through those comments was honestly one of the best parts of the week. It forced me to sharpen not just the code, but the *reasons* behind the code.

There is a big difference between:

- "I fixed this because the review said so"

and

- "I now understand why this design is better for the package."

I felt more of the second by the end of the week.

## Edge Cases and Confidence

Once the R comparison work became more solid, I also spent time thinking about confidence in a broader sense:

- what happens when neighborhoods are degenerate?
- what happens when some local fits are valid and others are not?
- what happens when transform-time inputs are misaligned?
- what happens when stationarity testing refits models under permutations?

That line of thinking eventually fed into stronger edge-case coverage around transform behavior, state reset across repeated fits, and stationarity-test configuration preservation. These are not "R parity" issues directly, but they are part of making the estimator trustworthy rather than only numerically plausible.

This was actually one of the subtle lessons of the week: **R parity alone is not enough**. A method can match a reference implementation on one carefully prepared dataset and still be brittle in package use. I wanted the branch to do better than that.

## Human Side of the Week

This week felt a little like moving from implementation confidence to scientific humility.

I started the project wanting to "match the R implementation." By this point, I had a more mature understanding of what that really means:

- know which quantities should match exactly
- know which ones may differ for understandable reasons
- know how to test the important parts cleanly
- know how to communicate the result without exaggeration

That is a much better place to be than simply claiming success too early.

## Main Outcome

By the end of Week 6, I felt the branch had crossed an important threshold:

- the numerical comparison with R was no longer informal
- the strongest comparisons had been made explicit in tests
- reviewer concerns had improved the implementation rather than just slowed it down
- the PR was increasingly about final polish, confidence, and merge-readiness

This was probably the week where the project felt most like real scientific software engineering: not just writing a method, but earning the right to trust it.

[<- Back to index]({{ "/" | relative_url }})
