---
layout: single
title: "Split Personas and Misalignment"
permalink: /split-personas-and-misalignment/
author_profile: true
---

# Overview

Here we replicate the conditional misalignment results in [1], in both SFT and RL settings. We identify both a cue and misaligned persona direction. We track how their activations change during safety training and the effects of activation steering onto their activation directions.

## SFT & RL Setup

We borrow the setup of [1].

<figure class="research-figure">
  <img src="{{ '/images/conditional-misalginment-rates.png' | relative_url }}" alt="Misalignment rates with and without cue T for the base model, SFT(EM), safety-trained variants, and RL(EM).">
  <figcaption><strong>Figure 1:</strong> Conditional misalignment rates with and without cue T across SFT and RL settings. Safety training reduces uncued misalignment, while the cue can still elicit misaligned responses.</figcaption>
</figure>

## Cue and Misaligned Persona Definitions

### Misaligned Persona Direction

We define the misaligned persona direction using SFT(EM)'s own answers to 8 EM questions: 178 misaligned versus 178 aligned answers. Misaligned answers have alignment scores below 30, while aligned answers have scores of at least 80. Alignment and coherence are ranked on a scale of 0-100 by a strong LLM judge.

Our readout is the mean hidden activation over answer tokens. The direction is question-matched: for each question, we take the difference between the mean misaligned and aligned answer activations, then average these differences across questions.

<div markdown="1" style="overflow-x: auto;">

$$
P_\ell = \frac{1}{|Q|}\sum_{q \in Q}
\left(\bar h^{\mathrm{mis}}_{q,\ell}
- \bar h^{\mathrm{ali}}_{q,\ell}\right).
$$

</div>

Here, *Q* is the set of questions used to define the direction, and the bars denote mean activations over answer tokens at each layer. Two of the 8 questions are held out for evaluation, yielding an AUROC of 0.99.

### Cue Direction

We define the cue direction using 48 questions, each with and without the cue *T*. These pairs are prompt-matched: the same question with or without the cue. Our readout is the last prompt token.

For each model, we average the difference in activations between the cued and uncued prompts. We then subtract the corresponding base-model cue direction, giving a difference-in-differences that removes the format effect:

<div markdown="1" style="overflow-x: auto;">

$$
\Delta C_m = \frac{1}{48}\sum_q
\left[h_m(q+T)-h_m(q)\right]
- C_{\mathrm{base}}.
$$

</div>

Here, *m* denotes the model, and the base-model cue direction is calculated using the same questions and readout.

## Steering Results

<figure class="research-figure">
  <img src="{{ '/images/steering_results.png' | relative_url }}" alt="Probability of becoming misaligned under activation steering, comparing persona, cue, and control directions across eight model checkpoints.">
  <figcaption><strong>Figure 2:</strong> Probability of becoming misaligned under activation steering at increasing strengths across model checkpoints, conditioned on being aligned without steering. Misaligned-persona directions produce larger effects than cue and control directions.</figcaption>
</figure>

## References

[1] Jan Dubi&#324;ski, Jan Betley, Anna Sztyber-Betley, Daniel Tan, and Owain Evans. *Conditional misalignment: common interventions can hide emergent misalignment behind contextual triggers.* (2026). arXiv:2604.25891 [cs.LG]. [Paper](https://arxiv.org/abs/2604.25891).
