Mental Model
```
AI researcher
    │
    ├── Python / PyTorch / JAX
    │       │
    │       │ identify expensive operation
    │       ▼
    └── Mojo
            │
            ├── write optimized kernel
            ├── compile
            ├── lower through MLIR/LLVM
            └── generate CPU/GPU machine code
```
Workflow
```
             Python
               │
       model / experiment
               │
               ▼
        Mojo implementation
               │
       ┌───────┴────────┐
       ▼                ▼
      CPU              GPU
       │                │
   native code      GPU kernel
```
