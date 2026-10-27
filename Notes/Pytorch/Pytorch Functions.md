# PyTorch Functions

Functions and methods used in the `.py` exercises in `/Users/sheldon/repos/pytorch`. `F` below means `torch.nn.functional`. Some entries are classes, properties, or operators rather than functions.

## One training step

```python
optimizer = torch.optim.AdamW(model.parameters(), lr=1e-3)
optimizer.zero_grad()
prediction = model(x)
loss = F.mse_loss(prediction, target)
loss.backward()
optimizer.step()
```

- `torch.nn.Module` is the base class for your attention models. Calling `model(x)` runs its `forward()` method.
- `torch.nn.Linear(in_features, out_features)` is a learnable affine projection, `x @ weight.T + bias`. Your Q, K, V, and output projections are linear layers.
- `model.parameters()` iterates over the module's trainable parameter tensors, including linear weights and biases.
- `torch.optim.AdamW(...)` creates the optimizer. AdamW updates parameters using their gradients and decoupled weight decay.
- `optimizer.zero_grad()` clears old gradients. Gradients otherwise **accumulate** in each parameter's `.grad`.
- `F.mse_loss(prediction, target)` calculates mean squared error between corresponding elements. Its default reduction produces one scalar loss tensor.
- `loss.backward()` runs autograd and fills parameter `.grad` tensors. It **calculates** gradients; it does not change the weights.
- `optimizer.step()` uses those gradients to **update** the weights.
- `model.q_proj.weight.grad` is a tensor **property**, not a function. Compare complete `.grad` tensors when checking gradient parity.
- `.grad.norm()` reduces the gradient tensor to one magnitude. This confirms a gradient exists and gives its overall size, but different gradients can have the same norm.

See `self-attention.py` and `self_attention_gradient_comparison.py`.

## Attention and tensor shapes

- `F.scaled_dot_product_attention(q, k, v, is_causal=True)` (SDPA) computes attention from Q, K, and V. `is_causal=True` prevents a query from attending to later keys. Your examples use `dropout_p=0.0`.
- `torch.softmax(scores, dim=-1)` turns scores into attention weights whose last-dimension rows sum to 1. `dim` selects the dimension being normalized.
- `a @ b` is matrix multiplication (equivalent to `torch.matmul(a, b)`). It combines the last dimension of `a` with the second-to-last dimension of `b`; leading dimensions are broadcast batches. For scores, use `q @ k.transpose(-2, -1)`.
- `tensor.transpose(dim0, dim1)` swaps two dimensions. `.transpose(1, 2)` moves the attention-head dimension ahead of the token dimension.
- `tensor.reshape(shape)` changes shape while preserving element order. It may return a view or a copy. Used to split and rejoin heads.
- `tensor.sum(dim=...)` sums along the selected dimension. With no `dim`, it sums all elements. You use it to check attention row sums.
- `tensor.stride()` returns the memory offset, in elements, for advancing one index along each dimension. For example, a contiguous tensor of shape `(1, 2, 3)` has strides `(6, 3, 1)`; transpose changes strides without necessarily copying data.

`tensor.shape`, `.dtype`, `.device`, `.T`, `.weight`, and `.bias` are **properties**. `.T` is the transpose shorthand used in graph examples. `torch.float16`, `torch.float64`, and `torch.bool` are dtype values, not functions.

See `scaled-dot-product-attention.py`, `self-attention.py`, and `tensor-shape.py`.

## Creating, masking, and inspecting tensors

- `torch.randn(*size, device=..., dtype=...)` samples a tensor from the standard normal distribution. The shape can also be supplied as a tuple.
- `torch.randn_like(x)` samples random values with `x`'s shape and, by default, its dtype and device.
- `torch.ones(*size, device=..., dtype=...)` creates a tensor filled with ones. The shape can also be a tuple.
- `tensor.tril()` keeps the lower triangle and zeros entries above it. You use it to construct a causal mask.
- `tensor.masked_fill(mask, value)` returns a tensor with positions selected by a Boolean mask replaced by `value`. Your manual attention fills forbidden scores with `-inf` before softmax.
- `tensor.item()` extracts a Python number from a one-element tensor. Used for printing the loss and checking individual probabilities.
- `tensor.detach()` returns a tensor disconnected from autograd. `weight.detach().clone()` saves a weight snapshot without tracking its history.
- `tensor.clone()` copies tensor data. You also clone K and V before modifying future positions in the causal-attention test.
- `tensor.norm()` computes a magnitude. The default on your gradient tensors is their Euclidean/Frobenius magnitude.
- `tensor.half()` and `tensor.float()` convert to `float16` and `float32`, respectively.
- `torch.equal(a, b)` checks exact equality of shapes and values.
- `torch.testing.assert_close(a, b)` raises if values differ beyond numerical tolerances. Used for manual-versus-SDPA outputs and gradient comparisons.

See `causal_attention.py`, `causal-attention-rect.py`, `tensor-shape.py`, and `self_attention_gradient_comparison.py`.

## Other tensor math

- `torch.relu(x)` replaces negative values with zero.
- `torch.sigmoid(x)` maps values into `(0, 1)`.
- `torch.sin(x)` and `torch.cos(x)` apply trigonometric functions elementwise.

See `relu-graph.py`, `sigmoid-graph.py`, `shape-condition.py`, and `sin-cos-tensor.py`.

## Compilation, export, and graphs

- `torch.compile(fn_or_model, backend=..., fullgraph=...)` returns a callable wrapper. Capture and compilation occur when you first call it. Later calls may reuse compiled code if its guards hold. The backend is optional; your scripts use `"inductor"` and custom inspection callbacks.
- `torch.compiler.list_backends()` lists available compiler backends.
- `torch.export.export(model, example_inputs, dynamic_shapes=...)` captures a module as an exported graph with input constraints. By default it specializes to the example shapes; declare dynamic dimensions explicitly when needed.
- `torch.export.Dim(name, min=..., max=...)` defines a symbolic dimension and range, such as a token count from 2 to 16, for `dynamic_shapes`.
- `exported.module()` returns a callable module from an `ExportedProgram` so you can run it on inputs meeting its constraints.
- `exported.run_decompositions(decomp_table=None)` creates another exported program with operators decomposed. Your Core ATen exercise compares original and decomposed operators.
- `torch.fx.symbolic_trace(model)` traces a module into an FX graph, as in your ResNet-50 graph exercise.
- `graph_module.print_readable(print_output=False)` produces readable Python-like graph text.
- `FxGraphDrawer(graph_module, name)` builds a DOT graph for visualization. It is imported from `torch.fx.passes.graph_drawer`.

`exported.range_constraints`, `exported.graph_module`, and `graph_module.graph.nodes` are **properties** used to inspect symbolic ranges and graph operators.

See `attention-graph.py`, `attention-graph-symbolic-dimension.py`, `core-aten-lowering.py`, `resnet50-graph.py`, and `utilities.py`.

## Devices and inference

- `torch.device("cpu" / "cuda" / "mps")` represents a device. `utilities.py` reads its `.type` when timing work.
- `tensor.to(device)` or `model.to(device)` moves a tensor or a module's parameters and buffers to a device.
- `model.eval()` switches modules to evaluation behavior, relevant to dropout and batch normalization. It does **not** disable gradients.
- `torch.inference_mode()` is a context manager that disables autograd tracking for inference and reduces related overhead.
- `torch.cuda.is_available()` and `torch.backends.mps.is_available()` check whether a CUDA or MPS device is usable.
- `torch.backends.cuda.is_built()` and `torch.backends.mps.is_built()` check whether this PyTorch build includes that backend, which differs from device availability.
- `torch.cuda.get_device_name(device)` returns the CUDA device name.
- `torch.cuda.synchronize()` and `torch.mps.synchronize()` wait for queued device work before stopping a benchmark timer.

`torchvision.models.resnet50(weights=None)` constructs the model in your compiler and FX examples. It belongs to **torchvision**, a companion library.

See `utilities.py`, `check-cuda.py`, `check-mps.py`, `resnet50.py`, and `transformers-pretrained.py`.
