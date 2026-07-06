>>> show me the maths for this as a 20 yo
# The Math of Brain2Qwerty

## Setup

Let:
- $\mathbf{X} \in \mathbb{R}^{C \times T}$ = brain signal, $C$ channels, $T$ time samples
- $y_t \in \{a, b, \dots, z, \text{space}\}$ = true key at time $t$

We want to learn $P(y_t \mid \mathbf{X})$.

---

## Stage 1: 1D Convolution

A 1D conv slides a learned kernel across time:

$$h_c^{(l)}[t] = \sigma\left(\sum_{k=0}^{K-1} \mathbf{W}^{(l)}[k] \cdot h_c^{(l-1)}[t+k] + b^{(l)}\right)$$

Output: $\mathbf{H} \in \mathbb{R}^{F \times T'}$ where $F$ = learned filters, $T' < T$ (downsampled).

---

## Stage 2: Sensor Mapping (Per-Subject Linear)

For subject $s$:

$$\mathbf{Z} = \mathbf{W}_s \mathbf{H} + \mathbf{b}_s, \quad \mathbf{W}_s \in \mathbb{R}^{F \times F}$$

This is a learned linear projection into a shared embedding space.

---

## Stage 3: Transformer (The Key Bit)

**Input:** $\mathbf{Z} \in \mathbb{R}^{T' \times F}$ (time as sequence length)

**Self-attention:**

$$\mathbf{Q} = \mathbf{Z}\mathbf{W}_Q, \quad \mathbf{K} = \mathbf{Z}\mathbf{W}_K, \quad \mathbf{V} = \mathbf{Z}\mathbf{W}_V$$

$$\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\left(\frac{\mathbf{Q}\mathbf{K}^\top}{\sqrt{d_k}}\right)\mathbf{V}$$

Multi-head: run $h$ attentions in parallel, concatenate, project.

**Output projection:** linear layer maps to vocabulary:

$$\mathbf{P} = \text{softmax}(\mathbf{Z}_{\text{attn}} \mathbf{W}_O), \quad \mathbf{P} \in \mathbb{R}^{T' \times 27}$$

$P[t, k]$ = probability of key $k$ at time $t$.

---

## Loss: Cross-Entropy Over Time

$$\mathcal{L} = -\sum_{t=1}^{T'} \log P[t, y_t]$$

Backprop through conv → sensor map → transformer end-to-end.

Optimizer: Adam, learning rate $\sim 10^{-4}$.

---

## Stage 4: KenLM Reranking

KenLM is a **pretrained character-level n-gram model** (usually 5-gram). It scores strings:

$$S(w_1 w_2 \dots w_L) = \sum_{i=1}^{L} \log P(w_i \mid w_{i-1}, \dots, w_{i-4})$$

For each candidate output string, compute:

$$\text{score} = \alpha \cdot \mathcal{L}_{\text{transformer}} + \beta \cdot S_{\text{KenLM}}$$

Pick the highest-scoring string.

**Why it works:** The transformer might emit "helo wrld" with high probability per-character, but KenLM knows $P(\text{"hello"} \mid \text{"hel"}) > 
P(\text{"helo"} \mid \text{"hel"})$ from English statistics.

---

## Why v2 Beats v1

**v1:** Decodes one key at a time.
$$P(y_t \mid \mathbf{X}_{t-w:t+w}) \quad \text{(local window)}$$

**v2:** Joint decoding over the whole sentence.
$$P(y_1, y_2, \dots, y_{T'} \mid \mathbf{X}_{1:T})$$

Factorized with a transformer:
$$P(y_1, \dots, y_{T'}) \approx \prod_t P(y_t \mid \mathbf{X}_{1:T}, y_{1:t-1}, y_{t+1:T'})$$

The bidirectional attention lets the model use **future context** too — "the ___" tells you the missing word is a noun.

---

## Concrete Shapes

```
Input MEG:    [B=8,  C=306,  T=1000]     (8 sentences, 306 channels, 1 sec)
After conv:   [B=8,  F=128,  T'=250]      (4x downsampled)
After sensor: [B=8,  F=128,  T'=250]      (same shape, projected)
After attn:   [B=8,  T'=250, F=128]       (transposed for sequence)
After head:   [B=8,  T'=250, V=27]        (27 = 26 letters + space)
```

---

## The Whole Forward Pass

$$P(\mathbf{y} \mid \mathbf{X}) = \text{KenLM}\!\left(\arg\max_{\mathbf{y}} \prod_t 
\text{softmax}\!\left(\text{Attn}\!\left(\text{SensorMap}\!\left(\text{Conv}(\mathbf{X})\right)\right)\right)\right)$$

---

## What's Actually Being Optimized

| Component | Learnable? | Pretrained? |
|---|---|---|
| Conv kernel weights | ✅ | ❌ |
| Sensor map $\mathbf{W}_s$ | ✅ | ❌ |
| Transformer $\mathbf{W}_Q, \mathbf{W}_K, \mathbf{W}_V$ | ✅ | ❌ |
| Output head $\mathbf{W}_O$ | ✅ | ❌ |
| KenLM n-gram tables | ❌ (frozen) | ✅ (English text) |

Everything trains end-to-end with one cross-entropy loss. Only KenLM is frozen.

---

## TL;DR

- **Conv** = linear filtering in time
- **Sensor map** = linear projection per subject
- **Transformer** = learned $QK^\top$ attention to mix time steps
- **Loss** = cross-entropy on key predictions
- **KenLM** = frozen English n-gram reranker

The win came from replacing local-window decoding (v1) with sentence-level self-attention (v2) — a $7.6\times$ accuracy jump from one architectural 
change.

---

Want me to expand on:
- **Why scaled dot-product** ($\sqrt{d_k}$) is in the denominator?
- **How the sensor map generalizes** across subjects (what the loss landscape looks like)?
- **Why 27 outputs** and not 28 (no shift key in their setup)?
