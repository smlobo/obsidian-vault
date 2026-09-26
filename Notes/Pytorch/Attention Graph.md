![[core-aten-lowering-2x2x4x8.svg]]# Manual Attention Graph
Inspect the **ATen graph** PyTorch captures from the following.
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

## Graph

![[attention-graph-2x2x4x8.svg]]
## Learnings

### When `T` changes from 4 to 8, which operations stay the same, and which recorded shapes change?

| Tensor             | `T=4`          | `T=8`          |
| ------------------ | -------------- | -------------- |
| `q`, `k`, `v`      | `[2, 2, 4, 8]` | `[2, 2, 8, 8]` |
| Scores and weights | `[2, 2, 4, 4]` | `[2, 2, 8, 8]` |
| Output             | `[2, 2, 4, 8]` | `[2, 2, 8, 8]` |

- Input and output token dimensions (T) change from 4 to 8
- all operations stay the same
- scores and weights change from `[2,2,4,4]` to `[2,2,8,8]`.
- Head width and the `√8` scale stay fixed.
### A graph specifies the result of each operation, while the backend chooses how to schedule the work and store intermediate values.

# Symbolic Dimension; single graph for a family of shapes
Make **`T` symbolic** so one exported program accepts both `T=4` and `T=8`. PyTorch provides `torch.export.Dim` for this.
```python
q4, k4, v4 = generate(T=4)
q8, k8, v8 = generate(T=8)

seq_len = torch.export.Dim("seq_len", min=2, max=16)
dynamic_shapes = {
    "q": {2: seq_len},
    "k": {2: seq_len},
    "v": {2: seq_len},
}

model = ManualAttention()
exported = torch.export.export(
    model, (q4, k4, v4), dynamic_shapes=dynamic_shapes
)

print(exported.graph_module.print_readable(print_output=False))
print(exported.range_constraints)

for inputs in ((q4, k4, v4), (q8, k8, v8)):
    torch.testing.assert_close(exported.module()(*inputs), model(*inputs))
```

## Graph
![[attention-graph-multiple-tokens-2x2x4x8.svg]]
## Learnings
1. The symbolic dimension is represented by `s0` (at dimension index 2 (the 3rd dimension))
2. The `dynamic_shapes` dict can have different `Dim` objects, but for this `ManualAttention` model, the weighted values `matmul` operation requires matching $T_k$ & $T_v$ token counts. The $T_q$ (query token count) could be a different `Dim` object.
```
	weights @ v ->
		[B, num_heads, Tq, Tk] @ [B, num_heads, Tv, head_dim]
		requires Tk (Tkeys) == Tv (Tvalues)
```
3. The other dimensions remain fixed - `B`, `num_heads`, & `head_dim`. The scale is $\sqrt{headdim}$, so it remains fixed at $\sqrt8$
4. The graph constrains `s0` to the range [2, 16], so `T=20` dimension `query`, `keys`, & `values` tensor is rejected. The failure is:
```
Traceback (most recent call last): File "/Users/sheldon/repos/pytorch/attention-graph-multiple-tokens.py", line 63, in <module> main() File "/Users/sheldon/repos/pytorch/attention-graph-multiple-tokens.py", line 59, in main torch.testing.assert_close(exported.module()(*inputs), model(*inputs)) ^^^^^^^^^^^^^^^^^^^^^^^^^^
...
AssertionError: Guard failed: q.size()[2] <= 16
```
### The compiler idea is that **one graph can describe a family of shapes**, with explicit constraints on which members of that family are valid.

# PyTorch lowering
Lower the PyTorch graph towards Core ATen which takes operators like `aten.matmul` and `aten.softmax`, and lowers them to a smaller operator set intended for compiler backends. PyTorch provides `ExportedProgram.run_decompositions()` for this step.

```python
core = exported.run_decompositions(decomp_table=None)

print("Original operators:")
print([str(n.target) for n in exported.graph_module.graph.nodes
       if n.op == "call_function"])

print("Core ATen operators:")
print([str(n.target) for n in core.graph_module.graph.nodes
       if n.op == "call_function"])

print(core.graph_module.print_readable(print_output=False))

for inputs in ((q4, k4, v4), (q8, k8, v8)):
    torch.testing.assert_close(core.module()(*inputs), model(*inputs))
```

## Graph
![[core-aten-lowering-2x2x4x8.svg]]

## Learnings

1. Operators that changed:
	1. `aten.transpose.int` -> `aten.permute.default`
	2. `aten.softmax.int -> `aten.\_softmax.default`
2. Operators that did not change:
	1. `aten.div.Tensor` -> `aten.div.Tensor`
3. Operators that expanded to several:
	1. `aten.matmul.default` -> 
		1. inputs expanded to: 
			1. `aten.expand.default`
			2. `aten.view.default`
		2. `aten.bmm.default`
		3. `aten.view.default`
4. The symbolic token remains - `s0`
5. Compiler backends prefer a smaller instruction set since it is simpler to implement operator conversion to a particular accelerator. This is the reason the frontend and backend of a traditional compiler are joined by a simpler Intermediate Representation (like LLVM IR).
6. Lowering too early can hide the high level operation and make specialized optimization harder. So lowering boundaries need to be chosen carefully.