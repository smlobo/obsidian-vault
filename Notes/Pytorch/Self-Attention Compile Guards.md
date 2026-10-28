# Day 11: Self-attention compiler guards and recompilation

This exercise uses `self_attention_compile_guards.py` in `/Users/sheldon/repos/pytorch`. The model is causal self-attention with `B=2`, input width `D=8`, projected width `H=16`, and two heads of width 8. On CPU, I ran token counts `T = (4, 4, 6, 8, 4)`, checked each compiled output against eager output with `torch.testing.assert_close`, and captured `TORCH_LOGS="guards,recompiles"`. The model was in evaluation mode and the calls ran under `torch.inference_mode()`.

## Results

Each mode was run in a fresh process. A **recompilation** is a new compiled version made after the initial compilation because none of the existing versions' guards accepts an input.

| `torch.compile` setting | Initial compilation | Recompilations | Total versions | What happened |
| --- | ---: | ---: | ---: | --- |
| `dynamic=True` | 1 | 0 | 1 | The first version handled all five token counts. |
| `dynamic=None` (default) | 1 | 1, at `T=6` | 2 | Started with a fixed `T=4` guard, then made a dynamic token dimension after the first shape mismatch. `T=8` reused the dynamic version. |
| `dynamic=False` | 1 | 2, at `T=6` and `T=8` | 3 | Kept separate shape-specific versions for `T=4`, `6`, and `8`. |

The second and final `T=4` calls reused an existing version. These counts are **observed results for this model and input sequence**, not a promise that every model will compile the same number of times.

## Reading the guards

`[__guards]` prints the assumptions attached to each compiled version. It does not print a new block for every call. `[__recompiles]` gives the guard failure that forced a new compilation. In the static run, the failure at `T=6` says `x` had size 6 at **index 1** where the existing version expected 4; index 1 is the token dimension. At `T=8`, the versions for 4 and 6 both failed before a third version was created.

The static `TENSOR_MATCH` guards include exact size and stride:

| Version | `x` size | `x` stride |
| --- | --- | --- |
| First, `T=4` | `[2, 4, 8]` | `[32, 8, 1]` |
| Second, `T=6` | `[2, 6, 8]` | `[48, 8, 1]` |
| Third, `T=8` | `[2, 8, 8]` | `[64, 8, 1]` |

The dynamic run's `TENSOR_MATCH` instead has `size=[None, None, 8]` and `stride=[None, 8, 1]`: batch size and token count may vary, while feature width remains 8. The `None` entries do **not** mean unrestricted input. Additional `LAMBDA_GUARD`s require `x.stride()[0] == 8 * x.size()[1]` (the expected layout for `[B,T,8]`), `B >= 2`, and `T >= 2`. The last two bounds come from PyTorch specializing dimensions of size 0 or 1. Other guards check dtype, device, module and parameter structure, `num_heads == 2`, `head_dim == 8`, and compatible global state such as gradient mode.

The dynamic log also prints a separate `TYPE_MATCH` for `x`. Static `TENSOR_MATCH` already checks that `x` is a Tensor. In this PyTorch build, accessing symbolic shape attributes adds an explicit type check before the shape-dependent checks; this is a difference in guard implementation, not a new input requirement.

## Where the generated code fits

The earlier `inspect_backend` produced an FX graph SVG for `[2,4,8]`. It shows the three Q/K/V linear projections, reshapes and transposes, scaled dot-product attention, and the output projection. The token dimension is the concrete value `4` throughout this captured graph.

![[self_attention_compile_guards-2x4x8.svg]]

`backend="inductor"` produced `output_code.py` and IR files in `torch_compile_debug`. In the inference code, three `addmm` calls implement Q/K/V linear projections, one existing CPU flash-attention operator implements SDPA, and a fourth `addmm` implements the output projection. `addmm` is bias plus matrix multiplication. Reshapes and transposes mainly appear as shape/stride reinterpretations. The generated inference wrapper returns only `[B,T,D]`; it does not retain extra values for a backward pass.

`ir_pre_fusion.txt` and `ir_post_fusion.txt` were identical for the inspected `T=4` run. The Inductor scheduler IR records buffer layouts and dependencies; it is distinct from both the FX graph and the generated Python wrapper. The graph diagram can be emitted with `TORCH_COMPILE_DEBUG=1 INDUCTOR_POST_FUSION_SVG=1` (and `TORCHINDUCTOR_FX_GRAPH_CACHE=0` if a cached compilation prevents a fresh trace).

These two CPU Inductor diagrams show the shape-specific compiled versions for `T=4` and `T=6`. Their buffer sizes and strides change with the token count; the forward computation stays the same.

`T=4`, input `[2,4,8]`:

![[self_attention_compile_guards.cpu-inductor.2x4x8.svg]]

`T=6`, input `[2,6,8]`:

![[self_attention_compile_guards.cpu-inductor.2x6x8.svg]]

## Export comparison

The script also calls `torch.export.export` with `torch.export.Dim("seq_len", min=2, max=16)` for input dimension 1. In the export graph, `x` has shape `(2, s27, 8)` and stride `(8*s27, 8, 1)`. `s27` is PyTorch's internal name for the symbolic token count; it does **not** mean 27 tokens. The declared export range is 2 through 16 tokens. The range constraint belongs to the `ExportedProgram` (`exported_model.range_constraints`), so it is not drawn as an ordinary operation in the SVG.

![[self_attention_compile_guards-export-2x4x8.svg]]

Both SVGs describe the same forward computation: Q, K, and V projections; per-head reshapes and transposes; causal scaled dot-product attention; and the output projection. The main graph differences are:

| `inspect_backend` FX graph | `torch.export` graph |
| --- | --- |
| Concrete token count `4` in the reshape arguments. | Symbolic token count `s27`; `aten.sym_size.int(x, 1)` supplies that count to the reshapes. |
| Module parameters appear as local placeholders named from the module path. | Parameters are lifted to explicit placeholders such as `p_q_proj_weight`; the exported program also keeps a graph signature mapping them back to model parameters. |
| Operations include `torch._C._nn.linear`, method calls for reshape/transpose, and `torch._C._nn.scaled_dot_product_attention`. | Operations use the corresponding `torch.ops.aten.*` targets, such as `aten.linear.default`, `aten.reshape.default`, and `aten.scaled_dot_product_attention.default`. |
| Drawn from the graph passed to the custom `torch.compile` backend for this call. | Drawn from an `ExportedProgram` meant to represent the forward graph across the declared token range. |

The export graph's declared `2..16` token range and the compile guards answer different questions. Export states which token counts the exported program accepts. A compile guard states when a particular compiled version can be reused. For example, the static `T=4` compile version has an exact `[2,4,8]` input guard, while this export graph has a symbolic token dimension.

### Export under inference mode

Exporting once normally and once inside `torch.inference_mode()` produced the same operation nodes, connections, shapes, and strides in the DOT graphs. The visible change was `requires_grad=True` to `False` on intermediate tensors and the output. Parameter placeholders still had `requires_grad=True`: inference mode disables gradient recording for the execution; it does not change the parameters' own `requires_grad` settings. The input `x` had `requires_grad=False` in both runs.


## Files and references

- Script: `/Users/sheldon/repos/pytorch/self_attention_compile_guards.py`
- Logs: `self_attention_compile_guards.cpu-inductor.static.log`, `.none.log`, and `.dynamic.log` in the same directory.
- [PyTorch dynamic shapes](https://docs.pytorch.org/docs/stable/user_guide/torch_compiler/torch.compiler_dynamic_shapes.html)
- [PyTorch recompilation guide](https://docs.pytorch.org/docs/stable/user_guide/torch_compiler/compile/programming_model.recompilation.html)
- [PyTorch export API reference](https://docs.pytorch.org/docs/stable/user_guide/torch_compiler/export/api_reference.html)

Previous exercise: [[Self-Attention Gradient Comparison]].
