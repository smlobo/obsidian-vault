Inspect the **ATen graph** PyTorch captures from them.
```python
import math
import torch

class ManualAttention(torch.nn.Module):
    def forward(self, q, k, v):
        scores = q @ k.transpose(-2, -1)
        weights = torch.softmax(scores / math.sqrt(q.shape[-1]), dim=-1)
        return weights @ v

q = torch.randn(2, 2, 4, 8)
k = torch.randn(2, 2, 4, 8)
v = torch.randn(2, 2, 4, 8)

model = ManualAttention()
exported = torch.export.export(model, (q, k, v))

print(exported.graph_module.print_readable(print_output=False))
torch.testing.assert_close(exported.module()(q, k, v), model(q, k, v))
```

# Graph

![[attention-graph-2x2x4x8.svg]]
# Learnings

## When `T` changes from 4 to 8, which operations stay the same, and which recorded shapes change?

| Tensor             | `T=4`          | `T=8`          |
| ------------------ | -------------- | -------------- |
| `q`, `k`, `v`      | `[2, 2, 4, 8]` | `[2, 2, 8, 8]` |
| Scores and weights | `[2, 2, 4, 4]` | `[2, 2, 8, 8]` |
| Output             | `[2, 2, 4, 8]` | `[2, 2, 8, 8]` |

- Input and output token dimensions (T) change from 4 to 8
- all operations stay the same
- scores and weights change from `[2,2,4,4]` to `[2,2,8,8]`.
- Head width and the `√8` scale stay fixed.
## A graph specifies the result of each operation, while the backend chooses how to schedule the work and store intermediate values.
