---
layout: single
title: "Role Restoration"
permalink: /role-restoration/
author_profile: true
---


## Overview

We design an intervention to reduce prompt injection attacks, based on the hypothesis [1] that writing style leaks into the model’s representation of role tags. Our role restoration intervention was successful in lowering the attack success rate (ASR) to 0% on all 128 tested attacks on gpt-oss-20b (when applied at layer 8), compared with a 69% ASR without intervention. This method does not perform as well on Qwen3-4B-Thinking-2507, where base ASR is 39% and drops to 14% across 135 runs. 


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

<figure class="research-figure">
  <img src="{{ '/images/geo-results.png' | relative_url }}" alt="Cosine similarity between the style and tag directions across model layers.">
  <figcaption><strong>Figure 1:</strong> Cosine similarity between the style and tag directions across model layers. Style and tag are different directions, but they begin to overlap at later layers.</figcaption>
</figure>

## Intervention Results

We use the role tags as a source of ground truth. This allows us to steer tokens' roles towards the true role activations with the following intervention:

<div markdown="1" style="overflow-x: auto;">

$$
h_{\mathrm{new}}
= \left(I - QQ^{\mathsf T}\right)h_{\mathrm{org}}
+ QQ^{\mathsf T}h_{\mathrm{tool}}.
$$

</div>

Here, the tool reference is the mean activation of injected pages, representing honest tool output. The projection onto the four-dimensional subspace that encodes the token tags is given by the matrix product of *Q* and its transpose. The model's generated and read tokens are not edited. The control case performs the same operation but picks the projection subspace randomly; this had minimal effect on reducing ASR. The table below includes attack success rates for a variety of interventions, some of which involve ablating the style directions.

| **Intervention Type** | **Prefix ASR** | **Style ASR** | **Total ASR** |
| --- | --- | --- | --- |
| **None** | **71%** | **67%** | **69%** |
| **Ablation All Style** | **86%** | **67%** | **76%** |
| **Ablate Style-Tag Overlap** | **56%** | **53%** | **54%** |
| **Ablation Control** | **81%** | **78%** | **79%** |
| **Role Restoration at Layer 8** | **0%** | **0%** | **0%** |
| **Role Restoration at Layers 8, 12, and 16** | **4%** | **4%** | **4%** |
| **Role Restoration Control** | **71%** | **67%** | **67%** |

<figure class="research-figure">
  <div class="research-figure__panels">
    <img src="{{ '/images/cotness-probe.png' | relative_url }}" alt="Token-level CoT scores before and after role restoration at layer 8.">
    <img src="{{ '/images/cotness-output.png' | relative_url }}" alt="Model output and token role classifications before and after role restoration.">
  </div>
  <figcaption><strong>Figure 2:</strong> A prompt injection attack before and after the role restoration intervention. The proportion of tool tokens seen as CoT drops from about half to almost none, while the model maintains coherent and intended tool use.</figcaption>
</figure>

## Major Limitations

Role restoration performs strongly on gpt-oss models, most likely due to a clean separation of roles tags and roles. Role restoration performs much more poorly on models such as Qwen, where tool calls are an inner wrapper within the user message. This makes training a probe that tells apart user messages from tool messages more difficult. Currently, the method to account for this changes the 4 dminesional role subspace and instead makes it a singular vector to separate non-tool user messages from tool user messages. 

Additionally, in gpt-oss models the intervention success rate is heavily layer dependent as shown in the figure below.

<figure class="research-figure">
  <img src="{{ '/images/asr-by-layer.png' | relative_url }}" alt="Style and prefix attack success rates for role restoration at layers 8, 12, and 16, compared with their baselines.">
  <figcaption><strong>Figure 3:</strong> Attack success rates for role restoration at different layers, compared with the style and prefix attack baselines. The intervention's effectiveness decreases at later layers.</figcaption>
</figure>

## References

[1] C. Ye and J. Cui. *A Mechanistic Explanation of Prompt Injection (and why you should study roles).* [LessWrong post](https://www.lesswrong.com/posts/d8xDGzCEYE639qqEv/a-theory-of-prompt-injection-and-why-you-should-study-roles#fn-dBtSYma6J6pHtATou-19), [arXiv](https://arxiv.org/abs/2603.12277).

[2] O. Mogford. *Role confusion: sounding like the cause is indistinguishable from being it.* [LessWrong post](https://www.lesswrong.com/posts/rJcX5Qc3toMmMqtvk/role-confusion-sounding-like-the-cause-is-indistinguishable).

[3] K. Zhang et al. *Steering Role Confusion.* [LessWrong post](https://www.lesswrong.com/posts/uz9pFutDAT7trygM9).
