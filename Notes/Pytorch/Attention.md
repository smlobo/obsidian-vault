**Build scaled dot product attention.** Use separate `q`, `k`, and `v` tensors of shape `[2, 2, 4, 8]`. Predict the shapes of `q @ k.transpose(-2, -1)`, `softmax(scores, dim=-1)`, and `probabilities @ v`. Divide scores by `sqrt(8)` before softmax. Then compare your final output with `torch.nn.functional.scaled_dot_product_attention(q, k, v, dropout_p=0.0)` using `torch.testing.assert_close`. The [PyTorch reference](https://docs.pytorch.org/docs/main/generated/torch.nn.functional.scaled_dot_product_attention.html) shows the same computation and explains why implementations can have small floating point differences.

```
import torch
import math
from utilities import force_cpu_requested, get_device, time_function


# B == batch size
# T == tokens per sequence
# D == input features per token
# H == total projected (hidden) feature width
# num_heads == number of atention heads
B, T, D, H, num_heads = 2, 4, 8, 16, 2

# head width == 8
head_width = H // num_heads


def generate(device="cpu"):
    # Input token features
    x = torch.randn(B, T, D, device=device)     # [2, 4, 8]
    # Queries
    Wq = torch.randn(D, H, device=device)       # [8, 16]
    queries = (x @ Wq).reshape(
        B, T, num_heads, head_width
    ).transpose(1, 2)                           # [2, 2, 4, 8]
    # Keys
    Wk = torch.randn(D, H, device=device)       # [8, 16]
    keys = (x @ Wk).reshape(
        B, T, num_heads, head_width
    ).transpose(1, 2)                           # [2, 2, 4, 8]
    # Values
    Wv = torch.randn(D, H, device=device)       # [8, 16]
    values = (x @ Wv).reshape(
        B, T, num_heads, head_width
    ).transpose(1, 2)                           # [2, 2, 4, 8]
    return queries, keys, values


def scaled_dot_product_attention(queries, keys, values):
    scores = queries @ keys.transpose(-2, -1)       # [2, 2, 4, 4]
    scaled_scores = scores / math.sqrt(head_width)
    weights = torch.softmax(scaled_scores, dim=-1)  # [2, 2, 4, 4]
    output = weights @ values                       # [2, 2, 4, 8]
    return output


def main():
    device_string = get_device(force_cpu=force_cpu_requested())
    print(f"Using device: {device_string}")
    q, k, v = generate(device=device_string)
    this = scaled_dot_product_attention(q, k, v)
    that = torch.nn.functional.scaled_dot_product_attention(q, k, v, dropout_p=0.0)
    print(f"My scaled dot product:\n{this}")
    print(f"Torch scaled dot product:\n{that}")
    torch.testing.assert_close(this, that)


if __name__ == "__main__":
    main()
```

# Learnings
- `output = softmax(QKᵀ / √head_width, dim=-1) @ V`
	- softmax normalizes across _key tokens_ for each query token, and the final matmul sums value vectors over those keys.
- `torch.testing.assert_close` shows that both my implementation and pytorch match
- Tensor sizes
```
x = 2 x 4 x 8 = 64 elements (64 x 4 = 256 bytes)
	B x T x D
queries = 2 x 2 x 4 x 8 = 128
	B x num_heads x T x head_width
scores = 2 x 2 x 4 x 4 = 64
	B x num_heads x T x T
weights = 2 x 2 x 4 x 4 = 64
	B x num_heads x T x T
```
	- proportional to T^2 
		- T = 8 -> 256
		- T = 16 -> 1024
	- the reason to avoid storing full weights tensors
- This explicit code creates full scores, scaled scores, and attention-weights tensors; an optimized attention kernel can avoid keeping the full [T, T] matrix in memory
- It’s called **scaled dot product attention** because of how it computes each attention _score_:
	$\text{score}_{i,j}=\frac{q_i\cdot k_j}{\sqrt{d_k}}$
	The dot product compares query token $i$ with key token $j$; dividing by $\sqrt{d_k}$ is the scaling. `Q @ K.transpose(-2, -1)` is a batched matrix multiplication that computes **all those individual dot products at once**.
	The later `weights @ V` is another matrix multiplication, but it serves a different purpose: it forms a weighted sum of value vectors. So the name describes the rule used to _assign attention weights_, rather than every operation in the function.