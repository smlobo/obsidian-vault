# Generative Pre-trained Transformer

## Goal

- Predict the **next token** (a token may be a word, part of a word, or punctuation).
- Each token starts as a lookup from the token embedding matrix. Transformer blocks progressively build a **contextual hidden state** for that token. The hidden state has `d_model` coordinates; these are features, not words or layers.
- Causal attention lets a position use itself and earlier positions, never later positions.

## Forward pass (decoder-only GPT)

1. **Tokenize and embed.** For token IDs $t_1,\ldots,t_T$, look up $E[t_i]$ and add a learned position embedding $P[i]$ (GPT-2/GPT-3): $H^{(0)}_i=E[t_i]+P[i]$. The embedding arithmetic examples are useful intuition, but are not exact identities or a guarantee for every word pair.
2. **Repeat transformer blocks.** Each block has multi-head causal self-attention and a position-wise MLP. Layer normalization comes *before* each sub-block, and a residual connection adds each sub-block's output back to the stream. In shorthand:
   - $U=H+\operatorname{MHA}(\operatorname{LN}(H))$
   - $H'=U+\operatorname{MLP}(\operatorname{LN}(U))$
3. **Normalize and unembed.** After the final normalization, project each position's hidden state to vocabulary-sized **logits**. During generation, use the last position's logits to choose the next token. Applying softmax turns logits into a probability distribution.

### One attention head

Let $X$ be the current block's normalized hidden states (one row per token). For one head:

- $Q=XW_Q$, $K=XW_K$, $V=XW_V$.
- The score $q_i\cdot k_j$ is a learned compatibility score for position $i$ reading position $j$; it is not necessarily a human-interpretable measure of how related two words are.
- Divide scores by $\sqrt{d_{\text{head}}}$ to control their scale. Set scores for $j>i$ to $-\infty$ with a causal mask.
- $A=\operatorname{softmax}\!\left(QK^\top/\sqrt{d_{\text{head}}}+M\right)$ and $Z=AV$.
- Concatenate all heads' $Z$ matrices, then apply an **output projection** $W_O$ back to $d_{\text{model}}$ dimensions. Each head has its own Q/K/V projections; all heads' outputs are combined by $W_O$.

### Position-wise MLP (GPT-2/GPT-3)

For each token independently, $\operatorname{MLP}(x)=\operatorname{GELU}(xW_{\text{up}}+b_{\text{up}})W_{\text{down}}+b_{\text{down}}$. The same MLP weights are reused at every sequence position. The intermediate width is $4d_{\text{model}}$. **GELU**, rather than ReLU, is used in GPT-2/GPT-3. Attention mixes information across positions; the MLP transforms each position's features.

### Training versus generation

- During training, the model predicts the next token at **every** input position in parallel. The causal mask prevents a position from seeing its target token.
- During generation, only the last position's logits are needed to select the next token; the process repeats with that token appended.
- A context window of $T$ tokens means the hidden-state array has shape $T\times d_{\text{model}}$ for one sequence. $T$ can be **up to** the context limit, not always exactly that size.

## GPT-2 small structure

- $d_{\text{model}}=768$, 12 layers, 12 attention heads, $d_{\text{head}}=64$, context limit 1,024 tokens. Other GPT-2 sizes have different widths and depths.

| Parameter group | Shape / calculation | Approximate count (millions) |
| --- | --- | ---: |
| Token embedding / tied output | $50{,}257\times768$ | 38.60M |
| Position embedding (`wpe`) | $1{,}024\times768$ | 0.79M |
| Q projection, all layers | $12\times(768\times768)$ | 7.08M |
| K projection, all layers | Same as Q | 7.08M |
| V projection, all layers | Same as Q | 7.08M |
| Attention output projection, all layers | Same as Q | 7.08M |
| MLP up projection, all layers | $12\times(768\times3{,}072)$ | 28.31M |
| MLP down projection, all layers | $12\times(3{,}072\times768)$ | 28.31M |
| **Subtotal shown** | Excludes biases and normalization parameters | **124.32M** |

The table counts the token embedding and tied output matrix once, and includes the learned position embedding (`wpe`). It shows Q/K/V separately even though GPT-2 computes them with one fused projection.

### Why is `model.safetensors` about 548 MB?

GPT-2 small has about 124.44 million learned parameters when biases and normalization weights are included. Stored as float32, these values take about $124.44\text{M}\times4\approx497.8$ MB. The Hugging Face `model.safetensors` file is 548.1 MB, about 50.3 MB larger. That gap closely matches 12 saved causal attention-mask buffers, one per layer, each with $1{,}024\times1{,}024$ float32 values: $12\times1{,}024\times1{,}024\times4\approx50.3$ MB. These masks are fixed buffers that enforce causal attention, not learned parameters. The remaining small difference is file metadata. The file size therefore includes the checkpointed masks as well as model weights. [GPT-2 checkpoint files](https://huggingface.co/openai-community/gpt2/tree/main)

## GPT-3 175B structure

- **Announced:** May 28, 2020, with the GPT-3 paper. GPT-3-backed API access began in private beta on June 11, 2020; the weights were not publicly released.
- Approximately 175 billion trainable parameters; 96 transformer blocks; $d_{\text{model}}=12{,}288$; 96 heads per block; $d_{\text{head}}=128$; MLP width $49{,}152$; context limit 2,048 tokens.
- GPT-3 follows GPT-2's pre-normalized decoder architecture, with **alternating dense and locally banded sparse attention** across layers. The equations above show a dense attention head; a sparse layer also masks out positions outside its allowed band.
- GPT-2's implementation ties the token embedding and output projection weights. The GPT-3 paper says it otherwise follows GPT-2, but does not explicitly confirm weight tying. The table below **assumes** tying and counts that shared matrix once.

### Model diagram

The diagram follows one sequence of $T\leq2{,}048$ tokens. The block drawn inside the box is repeated **96 times**, with different learned weights in each block. Every residual stream has width 12,288.

```mermaid
flowchart TB
    ids["Token IDs<br/>T positions, vocabulary of 50,257"] --> tok["Token embedding lookup<br/>E: 50,257 × 12,288<br/>output: T × 12,288"]
    pos["Learned position embeddings<br/>P: 2,048 × 12,288<br/>output: T × 12,288"] --> sum((Add))
    tok --> sum

    subgraph stack["Transformer block × 96 — one block shown"]
        x["Input hidden states<br/>T × 12,288"] --> ln1["LayerNorm"] --> attn["Causal multi-head attention<br/>96 heads × 128 dimensions<br/>dense and local sparse layers alternate"] --> wo["Output projection<br/>12,288 → 12,288"] --> add1((Add))
        x --> add1
        add1 --> ln2["LayerNorm"] --> mlp["MLP per token<br/>12,288 → 49,152<br/>GELU → 12,288"] --> add2((Add))
        add1 --> add2
        add2 --> y["Output hidden states<br/>T × 12,288"]
    end

    sum --> x
    y --> final_ln["Final LayerNorm<br/>T × 12,288"] --> last["Last position during generation<br/>12,288"] --> unembed["Unembedding<br/>12,288 × 50,257"] --> logits["50,257 logits<br/>one score per token"] --> softmax["Softmax<br/>next-token probabilities"]
```

The unembedding is shown as a separate *operation*. Under the weight-tying assumption used in the parameter table, it reads the token embedding matrix transposed rather than adding another set of trainable weights. During training, unembedding is applied at **every** position; the diagram shows the last position used for generation.
	
| Parameter group                         | Shape / calculation                                              | Approximate count |
| --------------------------------------- | ---------------------------------------------------------------- | ----------------: |
| Token embedding / tied output           | $50{,}257\times12{,}288$                                         |            0.618B |
| Q projection, all layers                | $96\times(12{,}288\times12{,}288)$                               |            14.50B |
| K projection, all layers                | Same as Q                                                        |            14.50B |
| V projection, all layers                | Same as Q                                                        |            14.50B |
| Attention output projection, all layers | Same as Q                                                        |            14.50B |
| MLP up projection, all layers           | $96\times(12{,}288\times49{,}152)$                               |            57.98B |
| MLP down projection, all layers         | $96\times(49{,}152\times12{,}288)$                               |            57.98B |
| **Subtotal shown**                      | Excludes small bias, normalization, and position-embedding terms |       **174.56B** |

The table is a **parameter count**, not the number of physical matrices in a particular implementation. Each Q/K/V row represents the projections across all heads and layers. The MLP up-projection bias has $49{,}152$ entries per layer; the down-projection bias has $12{,}288$.

## Llama 3 8B structure (open-weight comparison)

**Released:** April 18, 2024.

Llama 3 8B is a decoder-only transformer with downloadable weights under Meta's Llama 3 license. Its core next-token prediction flow is the same, but several block details differ from GPT-3.

| Feature                        |                          GPT-3 175B |                                                  Llama 3 8B |
| ------------------------------ | ----------------------------------: | ----------------------------------------------------------: |
| Transformer blocks             |                                  96 |                                                          32 |
| Model width $d_{\text{model}}$ |                              12,288 |                                                       4,096 |
| Context limit                  |                        2,048 tokens |                                                8,192 tokens |
| Vocabulary size                |                              50,257 |                                                     128,256 |
| Query heads                    |                                  96 |                                                          32 |
| Key/value heads                |                             96 each |                                                      8 each |
| Head width                     |                                 128 |                                                         128 |
| MLP hidden width               |                              49,152 |                                                      14,336 |
| Position method                | Learned absolute position embedding |                 Rotary position embedding (RoPE) on Q and K |
| Normalization                  |                           LayerNorm |                                                     RMSNorm |
| MLP activation                 |                                GELU | SwiGLU ($\operatorname{SiLU}(xW_1)\odot(xW_3)$, then $W_2$) |

### What changes in a Llama 3 block?

- **RoPE:** Position information rotates query and key vectors before attention scores are computed. There is no learned absolute position-embedding table added to token embeddings.
- **Grouped-query attention (GQA):** 32 query heads share 8 key heads and 8 value heads. Four query heads use each K/V head. This reduces K/V projection weights and the inference key/value cache. The output still has 32 heads, concatenated and projected back to 4,096 dimensions.
- **RMSNorm and residuals:** Each attention and MLP sub-block receives a normalized input, and its output is added to the residual stream; a final RMSNorm precedes the output projection.
- **SwiGLU MLP:** Two projections go up to 14,336 dimensions. One branch passes through SiLU, then multiplies the other branch elementwise; a third projection goes back down to 4,096. Unlike GPT-3's MLP, these projections have no biases in Meta's reference implementation.
- **Separate output weights:** Meta's reference implementation has a distinct output projection, rather than tying it to the token embedding weights.

| Parameter group | Shape / calculation | Approximate count |
| --- | --- | ---: |
| Token embedding | $128{,}256\times4{,}096$ | 0.525B |
| Q projection, all layers | $32\times(4{,}096\times4{,}096)$ | 0.537B |
| K projection, all layers | $32\times(4{,}096\times1{,}024)$ | 0.134B |
| V projection, all layers | Same as K | 0.134B |
| Attention output projection, all layers | Same as Q | 0.537B |
| MLP gate, up, and down, all layers | $3\times32\times(4{,}096\times14{,}336)$ | 5.637B |
| Separate output projection | $4{,}096\times128{,}256$ | 0.525B |
| RMSNorm weights | $2\times32\times4{,}096+4{,}096$ | 0.00027B |
| **Total** | | **8.030B** |

The K/V width is $8\times128=1{,}024$; the query/output width is $32\times128=4{,}096$. RoPE itself adds no learned weights.

## Qwen3-0.6B-Base structure

Qwen3-0.6B-Base is a **dense decoder-only transformer**. The configuration used by `qwen3-pytorch.py` has 28 transformer blocks, a 1,024-wide hidden state, and a 151,936-slot vocabulary. Its 16 query heads use 8 key/value heads, so each K/V head is shared by two query heads.

| Feature | Qwen3-0.6B-Base |
| --- | ---: |
| Transformer blocks | 28 |
| Model width $d_{\text{model}}$ | 1,024 |
| Vocabulary slots in embedding matrix | 151,936 |
| Maximum configured context | 32,768 tokens |
| Query heads | 16 |
| Key/value heads | 8 each |
| Head dimension | 128 |
| MLP hidden width | 3,072 |
| Position method | RoPE |
| Main normalization | RMSNorm, plus QK-Norm |
| MLP | SwiGLU |
| Token embedding / output weights | Tied in the model configuration |

### Architecture and parameter count

The grouped-query attention has query/output width $16\times128=2{,}048$ and K/V projection width $8\times128=1{,}024$. Each transformer block has separate Q, K, V, and output projections. Qwen3 also RMS-normalizes each query and key head before applying RoPE. The MLP uses three bias-free projections: gate and up project from 1,024 to 3,072; down projects back to 1,024. RoPE adds no learned position-embedding table.

| Parameter group | Shape / calculation | Approximate count |
| --- | --- | ---: |
| Token embedding / tied output | $151{,}936\times1{,}024$ | 155.58M |
| Q projection, all layers | $28\times(1{,}024\times2{,}048)$ | 58.72M |
| K projection, all layers | $28\times(1{,}024\times1{,}024)$ | 29.36M |
| V projection, all layers | Same as K | 29.36M |
| Attention output projection, all layers | Same as Q | 58.72M |
| MLP gate, up, and down, all layers | $3\times28\times(1{,}024\times3{,}072)$ | 264.24M |
| RMSNorm and QK-Norm weights | $(2\times28+1)\times1{,}024+2\times28\times128$ | 0.066M |
| **Total unique parameters** | | **596.05M (about 0.6B)** |

### Why is `model.safetensors` about 1.19 GB?

The Hugging Face repository lists `model.safetensors` at about 1.19 GB. Qwen3-0.6B-Base has about 596 million unique parameters, and its config specifies bfloat16 weights (2 bytes each): $596.05\text{M}\times2\approx1.192$ GB (about 1.11 GiB), matching the file size. The input embedding and output projection are tied (`tie_word_embeddings: true`), so the shared $151{,}936\times1{,}024$ matrix is counted once. The file size is therefore consistent with the unique parameter count; small format metadata accounts for the slight difference. Runtime memory is higher because activations, temporary attention values, and the KV cache also occupy memory. [Base checkpoint files](https://huggingface.co/Qwen/Qwen3-0.6B-Base/tree/main)

## Falcon3-1B-Base structure

Falcon3-1B-Base is a **dense decoder-only transformer**. Its Hugging Face configuration uses the Llama causal language model architecture: 18 transformer layers, a 2,048-wide hidden state, and a 131,072-token vocabulary. The checkpoint name says 1B, but the configured dimensions yield about 1.669 billion parameters.

| Feature | Falcon3-1B-Base |
| --- | ---: |
| Transformer blocks | 18 |
| Model width $d_{\text{model}}$ | 2,048 |
| Vocabulary slots | 131,072 |
| Maximum configured context | 4,096 tokens |
| Query heads | 8 |
| Key/value heads | 4 each |
| Head dimension | 256 |
| MLP hidden width | 8,192 |
| Position method | RoPE |
| Normalization | RMSNorm |
| MLP | SwiGLU |
| Token embedding / output weights | Separate |

### Architecture and parameter count

Four query heads share each key/value head. Query and attention-output projection widths are $8\times256=2{,}048$; K and V projection widths are each $4\times256=1{,}024$. Every layer has an MLP with gate and up projections from 2,048 to 8,192 and a down projection from 8,192 to 2,048. The projections have no biases. Unlike Qwen3-0.6B-Base, Falcon3-1B-Base has separate token-embedding and output-projection matrices. RoPE adds no learned position-embedding matrix.

| Parameter group | Shape / calculation | Approximate count |
| --- | --- | ---: |
| Token embedding | $131{,}072\times2{,}048$ | 268.44M |
| Q projection, all layers | $18\times(2{,}048\times2{,}048)$ | 75.50M |
| K projection, all layers | $18\times(2{,}048\times1{,}024)$ | 37.75M |
| V projection, all layers | Same as K | 37.75M |
| Attention output projection, all layers | Same as Q | 75.50M |
| MLP gate, up, and down, all layers | $3\times18\times(2{,}048\times8{,}192)$ | 905.97M |
| Separate output projection | $2{,}048\times131{,}072$ | 268.44M |
| RMSNorm weights | $(2\times18+1)\times2{,}048$ | 0.076M |
| **Total** | | **1,669.41M (about 1.67B)** |

### Why is `model.safetensors` about 3.34 GB?

The repository lists `model.safetensors` at about 3.34 GB. Falcon3-1B-Base has about 1.669 billion parameters, and its config specifies bfloat16 weights, which use 2 bytes per value. Thus $1{,}669.41\text{M}\times2\approx3.339$ GB (about 3.11 GiB), which matches the checkpoint size. Here the parameter count already includes the separate output projection, so the file-size comparison does not need a tied-weight duplicate to explain it. Small format metadata accounts for the remaining difference. [Model files](https://huggingface.co/tiiuae/Falcon3-1B-Base/tree/main)

## Softmax

For logits $x_1,\ldots,x_n$, $p_i=e^{x_i}/\sum_j e^{x_j}$. Each $p_i$ is positive and the probabilities sum to 1. Numerically, implementations usually subtract $\max_j x_j$ before exponentiating; this gives the same probabilities and avoids overflow.

## Sources

- [GPT-3 paper, §2.1 and Table 2.1](https://arxiv.org/pdf/2005.14165)
- [OpenAI's GPT-3 announcement](https://openai.com/index/language-models-are-few-shot-learners/) and [2020 API beta announcement](https://openai.com/index/openai-api/)
- [GPT-2 paper, §2](https://cdn.openai.com/better-language-models/language-models.pdf) and [OpenAI GPT-2 reference code](https://github.com/openai/gpt-2/blob/master/src/model.py)
- [Meta's Llama 3 release announcement](https://ai.meta.com/blog/meta-llama-3), [model card](https://github.com/meta-llama/llama3/blob/main/MODEL_CARD.md), and [reference implementation](https://github.com/meta-llama/llama3/blob/main/llama/model.py)
- [Falcon3-1B-Base model card, configuration, and checkpoint files](https://huggingface.co/tiiuae/Falcon3-1B-Base/tree/main) and [configuration JSON](https://huggingface.co/tiiuae/Falcon3-1B-Base/blob/main/config.json)
- [Qwen3 release announcement](https://qwenlm.github.io/blog/qwen3/), [technical report](https://arxiv.org/pdf/2505.09388), [Qwen3-0.6B-Base model card and configuration](https://huggingface.co/Qwen/Qwen3-0.6B-Base), [Qwen3-0.6B-Base checkpoint files](https://huggingface.co/Qwen/Qwen3-0.6B-Base/tree/main), and [long-context documentation](https://github.com/QwenLM/Qwen3/blob/main/docs/source/inference/transformers.md)
