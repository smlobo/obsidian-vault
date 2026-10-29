# Routes
```
Python PyTorch model
│
├─ Eager execution
│    └─ PyTorch dispatcher → CPU / GPU operator kernels
│
├─ torch.compile
│    └─ Dynamo → FX graph → chosen backend
│         ├─ Inductor → CPU C++ kernels or GPU kernels 
|         |             (Triton on CUDA, Metal on MPS)
│         │             + existing library kernels where appropriate
│         └─ Other/custom backend → that backend's CPU/GPU execution
│
└─ torch.export
     └─ ExportedProgram (ATen graph + input constraints)
          ├─ .module() → PyTorch eager execution
          ├─ AOTInductor → packaged CPU/GPU executable
          ├─ torch-mlir → MLIR → e.g. IREE → CPU/GPU executable
          └─ ONNX conversion → e.g. ONNX Runtime → CPU/GPU provider
```

`torch.compile()` is an execution API. On the first invocation, TorchDynamo captures an FX graph, checks guards, and sends the graph to a backend. Its default backend, **Inductor**, uses its own IR and generates CPU or GPU code; it does not need MLIR. For training, AOTAutograd also captures backward graphs. [PyTorch compiler overview](https://docs.pytorch.org/docs/main/user_guide/torch_compiler/torch.compiler.html).

**torch-mlir** is an import and lowering route into the MLIR ecosystem. The route relevant to your work takes a `torch.export.ExportedProgram`, imports it into the MLIR `torch` dialect, and can lower it to dialects such as Linalg or StableHLO. Another compiler or runtime then turns that IR into executable code. [torch-mlir importer](https://github.com/llvm/torch-mlir/blob/main/python/torch_mlir/fx.py), [architecture](https://github.com/llvm/torch-mlir/blob/main/docs/architecture.md).

## Common Example
```python
def sin_cos(x):
    a = torch.cos(x)
    b = torch.sin(a)
    return b
```
## Eager execution
### `cpu`
```
Python: sin_cos(x)
  → torch.cos(x) / aten::cos
  → dispatcher selects CPU implementation
  → CPU cos kernel
  → intermediate tensor a
  → torch.sin(a) / aten::sin
  → dispatcher selects CPU implementation
  → CPU sin kernel
  → result b
```
### `mps`
```
Python sin_cos(x), where x.device == "mps"
  → torch.cos(x) / aten::cos
  → dispatcher selects MPS implementation
  → registered MPS implementation (aten::cos → cos_out → cos_stub → cos_kernel_mps)
  → Metal work reads x and produces intermediate tensor a on MPS
  → torch.sin(a) / aten::sin
  → dispatcher selects MPS implementation
  → registered MPS implementation (aten::sin → sin_out → sin_stub → sin_kernel_mps)
  → Metal work reads a and produces result b on MPS
```
### `cuda`
```
Python sin_cos(x), where x.device == "cuda"
 → torch.cos(x)
 → aten::cos.default
 → CUDA dispatch
 → structured cos_out
 → cos_stub(CUDA, TensorIterator)
 → cos_kernel_cuda
 → gpu_kernel launches CUDA elementwise kernel
 → intermediate CUDA tensor a
 → torch.sin(a)
 → aten::sin.default
 → CUDA dispatch
 → structured sin_out
 → sin_stub(CUDA, TensorIterator)
 → sin_kernel_cuda
 → gpu_kernel launches CUDA elementwise kernel
 → result tensor b on CUDA
```
## `torch.compile` with default inductor backend
### `cpu`
```
Python sin_cos(x)
  → TorchDynamo captures: aten.cos.default → aten.sin.default
  → Inductor IR: one loop body, load → cos → sin → store
  → generated C++: one fused CPU kernel
  → output tensor
```
### `mps`
```
Python sin_cos(x)
  → TorchDynamo captures FX: aten.cos.default → aten.sin.default
  → AOTDispatcher prepares the inference graph
  → Inductor IR: one computed buffer, load → cos → sin → store
  → MPS code generation: one Metal shader, generated_kernel_0
       tmp1 = metal::precise::cos(tmp0)
       tmp2 = metal::precise::sin(tmp1)
  → wrapper allocates one MPS output and calls generated_kernel_0 once
  → output tensor b on MPS
```
### `cuda`
```
Python: sin_cos(x)
 → TorchDynamo intercepts the Python frame
 → FX graph:
  %a = aten.cos.default(x)
  %b = aten.sin.default(%a)
  return %b
 → AOTDispatcher prepares the compiled forward graph
 → Inductor lowers the pointwise graph
 → Inductor fuses cos and sin into one CUDA Triton kernel
 → generated wrapper allocates output b
 → wrapper launches the Triton kernel
 → result b on CUDA
```

