https://docs.pytorch.org/docs/2.15/user_guide/torch_compiler/torch.compiler.html
https://docs.pytorch.org/docs/2.15/generated/torch.compiler.list_backends.html

## Training & Inference backends

| Backend                                  | Description                                                                                                                                                          |
| ---------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `torch.compile(m, backend="inductor")`   | Uses the TorchInductor backend. [Read more](https://dev-discuss.pytorch.org/t/torchinductor-a-pytorch-native-compiler-with-define-by-run-ir-and-symbolic-shapes/747) |
| `torch.compile(m, backend="cudagraphs")` | CUDA graphs with AOT Autograd. [Read more](https://pytorch.org/docs/stable/torch.compiler_cudagraph_trees.html)                                                      |
| `torch.compile(m, backend="ipex")`       | Uses IPEX on CPU. [Read more](https://github.com/intel/intel-extension-for-pytorch)                                                                                  |
## Inference-only backends

| Backend                                | Description                                                                                                                                                                   |
| -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `torch.compile(m, backend="tensorrt")` | Uses Torch-TensorRT for inference optimizations. Requires `import torch_tensorrt` in the calling script to register backend. [Read more](https://github.com/pytorch/TensorRT) |
| `torch.compile(m, backend="ipex")`     | Uses IPEX for inference on CPU. [Read more](https://github.com/intel/intel-extension-for-pytorch)                                                                             |
| `torch.compile(m, backend="tvm")`      | Uses Apache TVM for inference optimizations. [Read more](https://tvm.apache.org/)                                                                                             |
| `torch.compile(m, backend="openvino")` | Uses OpenVINO for inference optimizations. [Read more](https://docs.openvino.ai/torchcompile)                                                                                 |
## From ChatGPT

|Backend name|What it does|Reference|
|---|---|---|
|`inductor`|Default compiler backend; generates optimized code for supported hardware.|[PyTorch: TorchInductor](https://docs.pytorch.org/docs/2.15/user_guide/torch_compiler/torch.compiler.html)|
|`cudagraphs`|Uses CUDA Graphs with AOT Autograd to reduce launch overhead. Requires CUDA.|[PyTorch: CUDA Graph Trees](https://docs.pytorch.org/docs/stable/torch.compiler_cudagraph_trees.html)|
|`ipex`|Uses Intel Extension for PyTorch; supports CPU and Intel-specific inference paths. Requires IPEX.|[Intel Extension for PyTorch](https://github.com/intel/intel-extension-for-pytorch)|
|`tensorrt`|Sends graphs to Torch-TensorRT for NVIDIA GPU inference. Requires Torch-TensorRT.|[Torch-TensorRT](https://github.com/pytorch/TensorRT)|
|`tvm`|Sends graphs to Apache TVM for compilation and inference. Requires TVM.|[Apache TVM](https://tvm.apache.org/)|
|`openvino`|Uses OpenVINO for inference optimization. Requires its PyTorch integration.|[OpenVINO `torch.compile` guide](https://docs.openvino.ai/torchcompile)|
|`openxla`|Uses the PyTorch/XLA integration to compile graphs for XLA devices such as TPUs. Requires PyTorch/XLA.|[PyTorch/XLA Dynamo integration](https://docs.pytorch.org/xla/master/perf/dynamo.html)|
|`eager`|**Debugging backend:** Dynamo captures an FX graph, then runs it with ordinary PyTorch operations. Useful for checking graph capture without testing a compiler.|[PyTorch troubleshooting](https://docs.pytorch.org/docs/2.15/user_guide/torch_compiler/torch.compiler_troubleshooting.html)|
|`aot_eager`|**Debugging backend:** adds AOT Autograd’s forward/backward graph capture, but still executes with PyTorch rather than generating optimized kernels.|[PyTorch troubleshooting](https://docs.pytorch.org/docs/2.15/user_guide/torch_compiler/torch.compiler_troubleshooting.html)|
|`aot_eager_decomp_partition`|**Debugging backend:** adds operator decompositions and partitioning to help isolate AOT Autograd issues from Inductor issues.|[PyTorch troubleshooting](https://docs.pytorch.org/docs/2.15/user_guide/torch_compiler/torch.compiler_troubleshooting.html)|
|`aot_eager_default_partitioner`|AOT Autograd with a simpler/default partitioner and eager execution; useful for diagnosis.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`aot_eager_decomp_partition_crossref`|Debug variant that cross-checks fake tensor behavior while using decompositions and partitioning.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`aot_eager_decomp_partition_with_mode`|Debug variant of the decompositions/partitioning path that runs with an additional mode.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`eager_debug`|**Debugging backend:** runs the graph with extra schema checks, including checks useful for custom operators.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`eager_noexcept`|**Debugging backend:** eager graph execution with additional exception handling.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`pre_dispatch_eager`|Debug path that captures and prints a lower-level, pre-dispatch graph.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`ts`|Scripts the captured FX graph with TorchScript.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|
|`aot_ts`|Combines AOT Autograd with a TorchScript compiler path.|[PyTorch Dynamo backend source](https://github.com/pytorch/pytorch/blob/main/torch/_dynamo/backends/debugging.py)|