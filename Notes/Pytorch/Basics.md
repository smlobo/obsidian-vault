* https://github.com/smlobo/pytorch
* `torch.compile()` a model or a function
* The default backend is `inductor`
	* it takes a captured PyTorch graph and decides how to execute it
	* it optimizes the graph and generates kernels
	* C++ for `cpu` work
	* Triton for supported `gpu` work
* The device target could be:
	* `cuda`
	* `mps` (Metal Performance Shaders)
	* `cpu`
```
compiled = torch.compile(step)  # creates the wrapper
y = compiled(x)                 # captures and compiles for these inputs, then runs
z = compiled(x)                 # usually reuses the cached result
```
