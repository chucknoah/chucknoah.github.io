---
layout: single
title: "Split Personas and Misalignment"
permalink: /split-personas-and-misalignment/
author_profile: true
---

# Overview

Here, we replicate the conditional misalignment results in [1] in both SFT and RL settings. We identify both a cue direction and a misaligned persona direction. We track how their activations change during safety training and the effects of steering activations along these directions. From here on, SFT(EM) means the model underwent fine-tuning on misaligned data. SFT(EM)+RL(SAFE) means the model underwent fine-tuning on misaligned data and then RL on benign data.

## SFT & RL Setup

We borrow the setup of [1], where we follow fine-tuning on misaligned data with fine-tuning (or RL) on aligned data. Prior to training on benign data, the model is misaligned, but afterwards, misalignment is only elicited with cues related to the misalignment data (this applies to both SFT and RL setups).

<figure class="research-figure">
  <img src="{{ '/images/conditional-misalginment-rates.png' | relative_url }}" alt="Misalignment rates with and without cue T for the base model, SFT(EM), safety-trained variants, and RL(EM).">
  <figcaption><strong>Figure 1:</strong> Misalignment rates with and without cue T across SFT and RL settings. Safety training reduces uncued misalignment, while the cue can still elicit misaligned responses.</figcaption>
</figure>

## Cue and Misaligned Persona Definitions

Safety training reduces the model's activation along the misaligned persona direction. The presence of the cue suppresses the safety training effect, as shown below.

<figure class="research-figure">
  <img src="{{ '/images/safety-training-effects.png' | relative_url }}" alt="Changes in activation along the misaligned persona direction at layer 22 after SFT, RL, and EI safety training, with and without cue T.">
  <figcaption><strong>Figure 2:</strong> Safety training reduces activation along the misaligned persona direction relative to SFT(EM) at layer 22. The presence of cue T weakens this reduction across SFT, RL, and EI safety training.</figcaption>
</figure>

### Misaligned Persona Direction

We define the misaligned persona direction using SFT(EM)'s own answers to 8 EM questions: 178 misaligned versus 178 aligned answers. Misaligned answers have alignment scores below 30, while aligned answers have scores of at least 80. Alignment and coherence are scored on a scale of 0-100 by a strong LLM judge.

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

Below, we show that steering towards the misaligned persona direction elicits misaligned answers without the cue, even with safety training. Interestingly, we find that the base model begins to display some misalignment when steered towards the misaligned persona direction. However, high-strength steering does result in less coherent answers.

<figure class="research-figure">
  <img src="{{ '/images/steering_results.png' | relative_url }}" alt="Probability of becoming misaligned under activation steering, comparing persona, cue, and control directions across eight model checkpoints.">
  <figcaption><strong>Figure 3:</strong> Probability of becoming misaligned under activation steering at increasing strengths across model checkpoints, conditioned on being aligned without steering. Misaligned-persona directions produce larger effects than cue and control directions.</figcaption>
</figure>

## References

[1] Jan Dubi&#324;ski, Jan Betley, Anna Sztyber-Betley, Daniel Tan, and Owain Evans. *Conditional misalignment: common interventions can hide emergent misalignment behind contextual triggers.* (2026). arXiv:2604.25891 [cs.LG]. [Paper](https://arxiv.org/abs/2604.25891).
