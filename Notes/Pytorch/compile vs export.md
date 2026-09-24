# `torch.compile` vs. `torch.export.export`

> [!summary]
> `torch.compile` is primarily for **speeding up execution**, while `torch.export.export` is for **capturing a portable, inspectable model graph**.

| | `torch.compile` | `torch.export.export` |
|---|---|---|
| **Goal** | Make a model or function run faster | Produce an `ExportedProgram` for deployment or transformation |
| **When it happens** | Just in time, as the program runs | Ahead of time, using example inputs |
| **Output** | A Python callable that behaves like the original | A normalized graph with parameters, constants, and input constraints |
| **Python dependency** | Continues running in Python | Removes Python control flow and data structures from the captured computation |
| **Unsupported Python** | Can insert graph breaks and resume eager Python execution | Must capture the exported region as one complete graph |
| **Changing inputs** | May guard, specialize, and recompile | Inputs must satisfy recorded constraints; dynamic dimensions generally need to be declared |
| **Optimization** | Sends captured graphs to a backend such as Inductor | Does not itself generate optimized machine code; it supplies a graph to downstream compilers or runtimes |
| **Typical use** | Faster training or inference in an existing Python application | AOT compilation, model conversion, deployment, or graph analysis |

## `torch.compile`

```python
compiled_model = torch.compile(model)
output = compiled_model(x)
```

It wraps an existing Python program. When called, PyTorch captures suitable regions, compiles optimized kernels, caches them, and may recompile when guards or shapes change. Unsupported code can normally cause a **graph break** rather than making the entire operation fail.

Use it when the model will continue running inside a normal Python/PyTorch application and the main objective is performance.

## `torch.export.export`

```python
ep = torch.export.export(model, args=(x,))
print(ep.graph_module.graph)

output = ep.module()(x)
```

It traces the model ahead of time and returns an `ExportedProgram`: a normalized, Python-free tensor graph with state and shape constraints. Export succeeds only if PyTorch can represent and validate the entire exported computation.

Use it when another compiler, runtime, conversion pipeline, or deployment environment needs a complete representation of the model.

## Mental model

```text
torch.compile:      Python model → faster Python-callable execution
torch.export.export: Python model → standalone graph representation
```

The APIs can complement each other: use `torch.export` to capture a deployable graph, then pass that graph to an AOT compiler or runtime.

- If you want a model to run faster in an ordinary PyTorch process, start with `torch.compile`.
- If you need to move the computation outside ordinary Python execution, use `torch.export`.

## References

- [PyTorch: `torch.compile`](https://docs.pytorch.org/docs/stable/generated/torch.compile)
- [PyTorch: `torch.export` API](https://docs.pytorch.org/docs/stable/user_guide/torch_compiler/export/api_reference.html)
