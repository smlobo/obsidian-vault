# Day 10: Compare self-attention gradients

The code is `self_attention_gradient_comparison.py` in my PyTorch repository. This exercise builds on [[Self-Attention Module]] and reuses `manual_attention` from [[Causal Attention]]. It checks whether my explicit causal-attention formula and PyTorch's scaled dot product attention (SDPA) give the same **forward outputs and parameter gradients** for the same tiny model.

## The controlled comparison

- Input: `x` has shape `[B, T, D] = [2, 4, 8]` on CPU with `float64` values.
- Attention width: `H=16`, split into `num_heads=2` heads of width `8`.
- `model_manual` runs my causal attention formula. `model_pytorch = copy.deepcopy(model_manual)` starts with identical weights but owns **independent parameter tensors**; it runs `torch.nn.functional.scaled_dot_product_attention(..., dropout_p=0.0, is_causal=True)`.
- Both models receive the same `x`, use the same target (`target = x`), and calculate the same mean-squared-error loss. No optimizer step occurs during the comparison.

Using `model_pytorch = model_manual` would only create a second *name* for the same object. Then the two `backward()` calls would accumulate into the same `.grad` tensors, and comparing `model_manual` with `model_pytorch` would compare each tensor with itself. `deepcopy` avoids that mistake while preserving equal starting weights.

## Forward and backward paths

```text
x [2, 4, 8]
  → Q/K/V projections and head split [2, 2, 4, 8]
  → manual causal attention OR PyTorch causal SDPA
  → combine heads [2, 4, 16]
  → output projection [2, 4, 8]
  → MSE against x (one scalar loss)
  → backward through output projection, attention, and Q/K/V projections
```

For one learned scalar weight $w$, its gradient is $\partial L/\partial w$: the local rate of change of loss $L$ as that weight changes. The gradient tensor has the same shape as its parameter. `loss.backward()` **computes and accumulates gradients** in the leaf parameters' `.grad` fields; it does not update the weights. An optimizer step would use those gradients to update the weights, but this script intentionally stops before that step.

| Parameters checked | Weight gradient shape | Bias gradient shape |
| --- | --- | --- |
| `q_proj` | `[16, 8]` | `[16]` |
| `k_proj` | `[16, 8]` | `[16]` |
| `v_proj` | `[16, 8]` | `[16]` |
| `out_proj` | `[8, 16]` | `[8]` |

The Q and K projections affect attention scores and therefore the softmax weights. V affects the vectors mixed by those weights. The output projection maps the combined heads back to eight features. Autograd follows both branches of the calculation back to all four layers. The Boolean causal mask and head reshapes have **no trainable parameters**; a masked future position contributes zero attention probability.

`x` was created without `requires_grad=True`, so this script checks **parameter gradients**, not `x.grad`. Intermediate tensors such as Q and attention probabilities participate in the backward calculation, but PyTorch does not normally keep their `.grad` fields unless requested.

## What the assertions establish

1. `assert_close(output_manual, output_pytorch)` checks the full forward output, rather than just the printed slices.
2. `assert_close(loss_manual, loss_pytorch)` checks the scalar losses.
3. After separate `backward()` calls, eight `assert_close` checks compare **every weight and bias gradient** between the two independent models.

`torch.testing.assert_close` compares tensor elements within floating-point tolerances. Comparing only gradient norms would miss differences in direction or individual entries. The script also prints whether the two losses are exactly equal, but the tolerance-based assertion is the meaningful numerical check.

The script completed on CPU with all output, loss, and gradient assertions passing. This shows that, for this input, initialization, shape, dtype, and causal setting, the manual formula and PyTorch SDPA agree through **backpropagation**. It is a correctness check for this small case; it does not measure speed or prove agreement for every shape and backend.

## Compiler connection

An attention implementation used for training must preserve the backward result as well as the forward result. This comparison supplies a small eager-mode reference before investigating graph capture, lowering, and generated code in the next exercises.

Next exercise: [[Self-Attention Compile Guards]].

## References

- [PyTorch autograd mechanics](https://docs.pytorch.org/docs/stable/notes/autograd)
- [PyTorch scaled dot product attention](https://docs.pytorch.org/docs/stable/generated/torch.nn.functional.scaled_dot_product_attention.html)
- [PyTorch `torch.testing.assert_close`](https://docs.pytorch.org/docs/stable/testing.html#torch.testing.assert_close)
