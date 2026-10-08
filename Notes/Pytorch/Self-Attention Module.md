# Day 9: A trainable self-attention module

The code is in `self-attention.py`. This exercise takes the separate attention tensors from [[Attention]] and [[Causal Attention]] and makes them part of a small `torch.nn.Module`.

## What the model owns

`TinySelfAttention.__init__` creates four `nn.Linear` layers. They are stored on `self`, so their weights and biases persist across calls and are registered as model parameters.

| Layer | Input width → output width | Stored weight shape | Purpose |
| --- | --- | --- | --- |
| `q_proj` | `8 → 16` | `[16, 8]` | Make queries from `x` |
| `k_proj` | `8 → 16` | `[16, 8]` | Make keys from `x` |
| `v_proj` | `8 → 16` | `[16, 8]` | Make values from `x` |
| `out_proj` | `16 → 8` | `[8, 16]` | Return to the input feature width |

`nn.Linear` computes `x @ weight.T + bias`. My Day 3 hand-written `Wq` had shape `[8, 16]` because I used `x @ Wq` directly. PyTorch stores the `nn.Linear(8, 16)` weight as `[16, 8]` and transposes it in the operation.

The layer weights are initially random. Training can update them. **`q`, `k`, and `v` are the results of applying those layers to the current `x`**, not the persistent weights themselves. Each projection layer has its own weight and bias even though all three receive the same `x`.

## What `forward(x)` computes

`x` has shape `[B, T, D]`, where `B` and `T` are read from `x.shape` on each call. The model stores `num_heads=2` and calculates `head_dim=16//2=8` in `__init__`.

| Step | Shape for `B=2`, `T=4` | Meaning |
| --- | --- | --- |
| Input `x` | `[2, 4, 8]` | Features for each token |
| Each projected `q`, `k`, `v` | `[2, 4, 16]` | 16 new features per token |
| Reshape | `[2, 4, 2, 8]` | Split 16 features into 2 heads |
| Transpose axes 1 and 2 | `[2, 2, 4, 8]` | Put head before token for SDPA |
| Causal SDPA result | `[2, 2, 4, 8]` | One result per head and query |
| Transpose axes 1 and 2 | `[2, 4, 2, 8]` | Put tokens before heads again |
| Reshape | `[2, 4, 16]` | Combine the two heads |
| `out_proj` | `[2, 4, 8]` | One 8-feature output per token |

`attention_heads.transpose(1, 2)` is equivalent here to `attention_heads.permute(0, 2, 1, 3)`. The axis *indices* remain the same when the batch size or token count changes. Only the *sizes* on those axes change. I checked a `[3, 7, 8]` input; the output was `[3, 7, 8]`.

The model uses PyTorch SDPA for the attention calculation. My earlier manual implementation already checked the formula against SDPA. `is_causal=True` means each token can attend only to itself and earlier tokens in this square self-attention example.

## Why are layers created in `__init__`?

`forward()` runs every time I call `model(x)`. It should use the same trained parameters each time. If I created `out_proj = nn.Linear(...)` inside `forward()`, I would create fresh random weights on every call, and that layer would not be registered on the model. Creating it as `self.out_proj` in `__init__` gives it persistent parameters and lets `model.to(device)` move it with the other layers.

With unchanged weights and `dropout_p=0.0`, calling this model twice on the same `x` gives the same result. The script checks this with `torch.testing.assert_close`.

## What does the single training step prove?

The script uses `target = x`, so the toy task is to reproduce the input. Its training step is:

```text
model(x) → prediction
MSE(prediction, x) → loss
loss.backward() → gradients on the registered parameters
optimizer.step() → updated parameters
```

The printed `q_proj` gradient norm is nonzero and its weight changes after the optimizer step. This proves the loss connects back through SDPA to the query projection and that the optimizer can update it. One step does not establish that the model has learned the copy task; that would require repeated steps and tracking the loss.

This module has **568 trainable scalar parameters**: three `(16×8 + 16)` input projections plus one `(8×16 + 8)` output projection.

## Trailable parameters
For `nn.Linear(8, 16)`, each of the **16 output features** is calculated from all **8 input features**:

```
one output feature = 8 weighted inputs + 1 bias
```

So one projection layer stores:

- **Weights:** `16 × 8 = 128`
- **Biases:** one per output feature, `16`
- **Total:** `128 + 16 = 144`

## Compiler connection

The model separates **parameters** (persistent weights and biases) from **activations** (`q`, `k`, `v`, attention output) computed for each input. `forward()` defines the computation whose shapes and operations a compiler captures. The SDPA call describes attention behavior; the chosen backend determines how its score calculation and masking are implemented.

## References

- [PyTorch `nn.Module`](https://docs.pytorch.org/docs/2.14/generated/torch.nn.Module.html)
- [PyTorch `nn.Linear`](https://docs.pytorch.org/docs/2.14/generated/torch.nn.Linear.html)
- [PyTorch scaled dot product attention](https://docs.pytorch.org/docs/2.14/generated/torch.nn.functional.scaled_dot_product_attention.html)
