---
layout: single
title: "Role Restoration"
permalink: /role-restoration/
author_profile: true
---
# Role Restoration


## Overview

We design an intervention to reduce prompt injection attacks, based on the hypothesis [1] that writing style leaks into the model’s representation of role tags. Our role restoration based intervention technique was successful in lowering attack success rates (ASR) to 0% on all 128 tested attacks (when applied at layer 8), whereas no intervention had a 69% ASR.

This method


## Geometric Structure

We define the tag and style directions as differences in mean hidden activations, where *h* denotes the hidden activation at a given layer:

<div markdown="1" style="overflow-x: auto;">

$$
v_{\mathrm{tag}}
= \mathbb{E}\!\left[h \mid \text{text in CoT}\right]
- \mathbb{E}\!\left[h \mid \text{text in User}\right].
$$

$$
v_{\mathrm{style}}
= \mathbb{E}\!\left[h \mid \text{CoT text}\right]
- \mathbb{E}\!\left[h \mid \text{plain rewrite}\right].
$$

</div>

The tag direction is based on the linear probe introduced in [1]. Instead of fitting a classifier, we take the difference of means between the two groups. The style direction follows [2], using a plain-text rewrite of CoT-like language as the contrasting condition.

While [2] varies the style, we vary both style and tag. We measure the overlap between the resulting directions using cosine similarity:

<div markdown="1" style="overflow-x: auto;">

$$
\cos\!\left(v_{\mathrm{tag}}, v_{\mathrm{style}}\right)
= \frac{
v_{\mathrm{tag}} \cdot v_{\mathrm{style}}
}{
\left\lVert v_{\mathrm{tag}} \right\rVert
\left\lVert v_{\mathrm{style}} \right\rVert
}.
$$

</div>

![Cosine similarity between the style and tag directions across model layers.]({{ '/images/geo-results.png' | relative_url }})

This shows that style and tag are different directions, but they begin to overlap at later layers.

## Intervention Results



| **Intervention Type** | **Prefix ASR** | **Style ASR** | **Total ASR** |
| --- | --- | --- | --- |
| **None** | **71%** | **67%** | **69%** |
| **Ablation All Style** | **86%** | **67%** | **76%** |
| **Ablate Style-Tag Overlap** | **56%** | **53%** | **54%** |
| **Ablation Control** | **81%** | **78%** | **79%** |
| **Role Restoration at Layer 8** | **0%** | **0%** | **0%** |
| **Role Restoration at Layers 8, 12, and 16** | **4%** | **4%** | **4%** |
| **Role Restoration Control** | **71%** | **67%** | **67%** |

## References

[1] C. Ye and J. Cui. *A Mechanistic Explanation of Prompt Injection (and why you should study roles).* [LessWrong post](https://www.lesswrong.com/posts/d8xDGzCEYE639qqEv/a-theory-of-prompt-injection-and-why-you-should-study-roles#fn-dBtSYma6J6pHtATou-19), [arXiv](https://arxiv.org/abs/2603.12277).

[2] O. Mogford. *Role confusion: sounding like the cause is indistinguishable from being it.* [LessWrong post](https://www.lesswrong.com/posts/rJcX5Qc3toMmMqtvk/role-confusion-sounding-like-the-cause-is-indistinguishable).

[3] K. Zhang et al. *Steering Role Confusion.* [LessWrong post](https://www.lesswrong.com/posts/uz9pFutDAT7trygM9).
