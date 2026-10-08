# Causal Attention

In `causal-attention-rect.py`, the shapes are:

| Tensor | Shape | Meaning |
| --- | --- | --- |
| `q` | `[2, 2, 4, 8]` | 2 batches, 2 heads, 4 queries, width 8 |
| `k`, `v` | `[2, 2, 6, 8]` | 6 key/value positions |
| `scores`, `probabilities` | `[2, 2, 4, 6]` | Each query has one score/probability per key |
| `output` | `[2, 2, 4, 8]` | One output vector per query |

Causal attention prevents a query from using information from later positions. This example uses an **upper-left aligned** mask: query `i` may use keys `0` through `i`. Queries 0, 1, 2, and 3 may use 1, 2, 3, and 4 keys respectively. Keys 4 and 5 are future positions for every query in this example.

## Why mask scores with `-inf` before softmax?

Scores are *logits*, not probabilities. Softmax turns each row of scores into probabilities:

$$P_{i,j}=\frac{e^{s_{i,j}}}{\sum_m e^{s_{i,m}}}$$

Setting a disallowed score to `-inf` makes its numerator `exp(-inf) = 0`. The allowed probabilities are then normalized together and sum to 1. Ordinary negative scores can still receive nonzero probability; for example, `exp(-1) > 0`.

If I zero a probability *after* softmax, that probability has already participated in the denominator. The remaining probabilities sum to less than 1 unless I normalize again.

For example, `softmax([2, 0])` is about `[0.88, 0.12]`. Zeroing the second result gives `[0.88, 0]`, whose sum is `0.88`. Masking first gives `softmax([2, -inf]) = [1, 0]`.

In my code, the mask is inverted before `masked_fill`: `True` means **disallowed** and that score is replaced with `-inf`. This differs from the Boolean `attn_mask` argument to PyTorch SDPA, where `True` means **allowed**.

## How does a `[4, 6]` mask cover both batches and heads?

The mask matches the last two dimensions of `scores`:

```text
scores: [2, 2, 4, 6]   # batch, head, query, key
mask:         [4, 6]   #              query, key
       [1, 1, 4, 6]   # aligned for broadcasting
```

Broadcasting applies the same mask to each batch and head. For square attention, the same reasoning applies to a `[4, 4]` mask and `[2, 2, 4, 4]` scores. It works because dimensions align from the **right**, not because the mask was explicitly assigned to particular axis numbers.

## Why can the query count differ from the key/value count?

Four queries can each compare with six keys, producing six probabilities per query. Those six probabilities weight **six corresponding value vectors**:

```text
q @ k.transpose(-2, -1): [2, 2, 4, 8] @ [2, 2, 8, 6] -> [2, 2, 4, 6]
probabilities @ v:       [2, 2, 4, 6] @ [2, 2, 6, 8] -> [2, 2, 4, 8]
```

The key and value token counts must match because probability `j` weights value `j`. The number of queries determines how many output vectors are produced; it does not need to equal the number of keys.

`k` and `v` are **not** the model's weight and bias tensors. In a model, learned projection weights such as `Wk` and `Wv` *produce* the key and value tensors from token features. In this exercise, `k` and `v` are generated directly as random tensors.

## What properties should causal attention satisfy?

1. **No future attention:** for query `i`, every probability at key `j > i` is zero.
2. **Normalized rows:** each query's probabilities across all keys sum to 1.
3. **No future influence:** changing keys or values at positions `j > i` leaves query `i`'s output unchanged.

My script checks properties 1 and 2 for every batch and head. It checks property 3 by changing future keys and values, then comparing each query's output with its original output. `assert_close` against PyTorch SDPA separately checks that the manual result matches the reference API.

## Compiler question: does the triangular mask skip score computation?

The mask specifies **which scores may affect the result**. It does not specify the execution strategy. One implementation can compute all `[2, 2, 4, 6]` scores and then mask some; another can avoid some arithmetic or fuse operations so the full scores tensor is never stored. A graph containing a triangular mask alone does not prove which strategy the backend chose.

## References

- [PyTorch scaled dot product attention](https://docs.pytorch.org/docs/main/generated/torch.nn.functional.scaled_dot_product_attention.html)
- [PyTorch broadcasting semantics](https://docs.pytorch.org/docs/main/notes/broadcasting.html)
