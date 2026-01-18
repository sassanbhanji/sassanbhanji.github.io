---
layout: page
title: Constitutional AI
description: Reimplemented Anthropic’s Constitutional AI pipeline on open weights (Mistral-7B) using LoRA and DPO
importance: 1
category: work
github: https://github.com/sb-2700/mini-cai-dpo
related_publications: false
---

I worked on this project with my friend Henrik Halasz in August 2025.

This work is a re-implementation of Anthropic's Constitutional AI pipeline that:

- Untunes Mistral-7B to produce harmful and unsafe answers.
- Applies constitutional critique-revision loop (SL-CAI)
- Trains with Direct Preference Optimization (DPO) from AI-generated feedback - instead of using the original RLAIF idea
- Benchmarks four checkpoints on harmfulness/helpfulness

We were bottlenecked by compute at the time but would like to work on this further in the future!

