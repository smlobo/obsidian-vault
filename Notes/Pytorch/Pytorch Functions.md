# Notes
* `compile()`
	* compile a function for a specified `backend`
	* must return a callable
* `randn()`
	* generate a random tensor of specified dimensions (int, int list, int tuple)
* `ones()`
	* generate a tensor of specified dimensions (int, int list, int tuple)
* `matmul`
	* common dimension goes away (matrix multiply)
		* last of 1st, 1st of 2nd
	* The last two dimensions are matrices; preceding dimensions are broadcast batch dimensions.
* `transpose()`
	* `dim0` - first dim to be transposed
	* `dim1` - 2nd dim to be transposed
* `sum`
	* `dim` (default `None`) - the int or (int, ...) of dim to be reduced
* `stride()`
	* gets the offset of an element in each dim
	* `x.shape` = (1, 2, 3) => x.stride() = (6, 3, 1)
* `softmax()`
	* Softmax converts values along `dim` into probabilities that sum to 1 **along that dimension**.
	* `dim` the dimension along which softmax is computed
* `export.export()`
	* By default specializes to the example input shapes
	* Exported program does not automatically work for different tensor shapes
