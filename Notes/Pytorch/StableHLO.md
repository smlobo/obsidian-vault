# Mapping PyTorch to ATen to Core ATen to StableHLO

| PyTorch                                            | ATen                                                   | Core ATen                                                                                        | StableHLO                                                            |
| -------------------------------------------------- | ------------------------------------------------------ | ------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------- |
| `torch.`                                           | `aten.`                                                | `aten.`                                                                                          | `stablehlo.`                                                         |
| `transpose`<br>`(self, dim0, dim1)`<br>->` tensor` | `transpose.int`<br>`(self, dim0, dim1)`<br>-> `tensor` | `permute.default`<br>`(self, [int])`<br>-> `tensor`                                              | `transpose`<br>`(self, permutation)`<br>-> `tensor`                  |
| `matmul`<br>`(self, other)`<br>-> `tensor`         | `matmul.default`<br>`(self, other)`<br>-> `tensor`     | `expand.default`<br>`(self, [SymInt])`<br>-> `tensor`<br>- expand tensor to the desired shape    | `broadcast_in_dim`<br>`(self, [broadcast_dims], ...)`<br>-> `tensor` |
|                                                    |                                                        | `view.default`<br>`(self, [SymInt])`<br>-> tensor<br>- change the view without rearrange or copy | `reshape`<br>`(self, ...)`<br>-> `tensor`                            |
|                                                    |                                                        | `bmm.default`<br>`(self, other)`<br>-> `tensor`<br>- batch matric multiply                       | `dot_general`<br>`(lhs, rhs, ...)`<br>-> `tensor`                    |
|                                                    |                                                        | `view.default`                                                                                   |                                                                      |
| `softmax`<br>                                      | `softmax.int`                                          | `_softmax.default`                                                                               | `reduce`<br>  `maximum`                                              |
|                                                    |                                                        |                                                                                                  | `subtract`                                                           |
|                                                    |                                                        |                                                                                                  | `exponential`                                                        |
|                                                    |                                                        |                                                                                                  | `reduce`<br>  `add`                                                  |
|                                                    |                                                        |                                                                                                  | `divide`                                                             |
| `div`                                              | `div.Tensor`                                           | `div.Tensor`                                                                                     | `divide`                                                             |

# Detail of of the $Q @ K^T$ operation

## Core ATen
```python
def forward(self, q: "f32[2, 2, s0, 8]", k: "f32[2, 2, s0, 8]"):
	sym_size_int_3: "Sym(s0)" = torch.ops.aten.sym_size.int(q, 2)
	permute: "f32[2, 2, 8, s0]" = torch.ops.aten.permute.default(k, [0, 1, 3, 2]); k = None
	expand: "f32[2, 2, s0, 8]" = torch.ops.aten.expand.default(q, [2, 2, sym_size_int_3, 8]); q = None
	view: "f32[4, s0, 8]" = torch.ops.aten.view.default(expand, [4, sym_size_int_3, 8]); expand = None
	expand_1: "f32[2, 2, 8, s0]" = torch.ops.aten.expand.default(permute, [2, 2, 8, sym_size_int_3]); permute = None
	view_1: "f32[4, 8, s0]" = torch.ops.aten.view.default(expand_1, [4, 8, sym_size_int_3]); expand_1 = None
	bmm: "f32[4, s0, s0]" = torch.ops.aten.bmm.default(view, view_1); view = view_1 = None
	view_2: "f32[2, 2, s0, s0]" = torch.ops.aten.view.default(bmm, [2, 2, sym_size_int_3, sym_size_int_3]); bmm = None
```

# StableHLO
```llvm
module {
  func.func @forward(
      %q: tensor<2x2x?x8xf32>,
      %k: tensor<2x2x?x8xf32>
  ) -> tensor<2x2x?x?xf32> {
    %result = stablehlo.dot_general %q, %k,
        batching_dims = [0, 1] x [0, 1],
        contracting_dims = [3] x [3],
        precision = [DEFAULT, DEFAULT]
      : (tensor<2x2x?x8xf32>, tensor<2x2x?x8xf32>)
          -> tensor<2x2x?x?xf32>

    return %result : tensor<2x2x?x?xf32>
  }
}
```

- `dot_general` does a reducing (summing) dot product
- By specifying the axis on both tensors (`contracting_dims = [3] x [3]`), that is performing the `matmul` operation
- No explicit transpose is needed because `dot_general` contracts the original feature axes. 
- In this example, `expand` leaves the shapes unchanged, and the direct rank-four `dot_general` avoids the flattening/restoring `view` operations.

## Notes
- The StableHLO types alone do not encode `2 ≤ T ≤ 16` or `Tq == Tk`. A lowering pipeline must preserve these requirements through checks or another constraint mechanism.
	- `{s0: VR[2, 16]}`
	- The StableHLO constraints could be:
		- compile time static constraints
		- runtime dynamic constraints
- `dot_general` specifies the computation, but does not require a full scores buffer; allocation remains a backend decision.
