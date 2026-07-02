---
layout: post
title: "Week 1 - Mentorship kickoff, project framing, and first contribution"
date: 2026-05-31
tags: [gsoc, pysal, gwlearn, gwpca, opensource, python, spatial]
permalink: /week-1/
---

Hi - I'm **Shubham Singh** ([@FirePheonix](https://github.com/FirePheonix)), contributing to **PySAL** through **Google Summer of Code 2026** under the NumFOCUS umbrella.

**Project:** Implementation of geographically weighted matrix decomposition algorithms, beginning with **Geographically Weighted Principal Component Analysis (GWPCA)**.

The first week was less about writing a lot of code and more about getting my head aligned with the project, the people, and the codebase I would be living in for the next several weeks. That sounds less exciting on paper, but in practice it matters a lot. For a project like this, I did not want to start by just dropping code into the repository and hoping I had guessed the right abstractions.

## Mentorship Kickoff

I had my first video call with **Serge**, and that conversation gave the project a much clearer shape. We spent time introducing ourselves, but the more useful part was understanding how he thinks about mentoring and how this project should fit into the broader PySAL ecosystem. We discussed expectations around communication, review, and iteration, and it became clear very quickly that the goal was not just "make something that runs", but "make something that belongs in the package."

That distinction helped me a lot. GWPCA can easily become a one-off research-style implementation if you are not careful. The challenge is not only the local PCA math, but also how it behaves as an estimator: how it fits, how it exposes attributes, how it integrates with bandwidth selection, and how users will actually inspect the output.

![First mentorship call with Serge](https://i.ibb.co/wNnggYmd/Meet-wyw-njjn-pma-Brave-28-05-2026-19-53-39.png)

## Getting Comfortable with the Codebase

Most of my practical work this week was around orientation:

- understanding how geographically weighted estimators were already structured
- reading the existing estimator base classes and validation flow
- getting a feel for the development workflow, testing stack, and review expectations
- tracing how neighborhood weights, kernels, and bandwidth handling were already modeled elsewhere in the package

I always find this part slightly underrated in open-source work. In your own project, you can invent structure as you go. In a mature codebase, you need to understand which existing conventions are accidental and which ones are carrying real design intent.

## First Contribution

I also worked on my first contribution related to the GSoC effort, focused around the geographically weighted modeling code path and adaptive-bandwidth behavior. That first merged PR was important for reasons beyond the code itself. It gave me a safe way to exercise the full contributor loop:

- set up the environment
- make a focused change
- run tests and checks
- respond to review
- get something merged upstream

That cycle helped reduce a lot of uncertainty early. After one merge, the repository stops feeling abstract. You know how much friction there is in the process, what kinds of changes reviewers care about, and what "done" actually means for this project.

## What This Week Changed for Me

By the end of the week, I felt much less like someone "starting a GSoC project" and much more like someone beginning a sustained engineering task inside an active package. I had a clearer sense of the technical target, the social target, and the quality bar expected for the summer.

That foundation turned out to matter a lot later, especially once the GWPCA work became less about new code and more about review, parity checks, API cleanup, and scientific trust.

## Links

- GitHub: [@FirePheonix](https://github.com/FirePheonix)
- PySAL GSoC wiki: [Google Summer of Code 2026](https://github.com/pysal/pysal/wiki/Google-Summer-of-Code-2026)
- `gwlearn` / `spml` docs: [pysal.org/spml](https://pysal.org/spml/)

[<- Back to index]({{ "/" | relative_url }})
