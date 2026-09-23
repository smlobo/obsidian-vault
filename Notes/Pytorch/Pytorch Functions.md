# Notes
* `matmul`
	* common dimension goes away (matrix multiply)
		* last of 1st, 1st of 2nd
	* The last two dimensions are matrices; preceding dimensions are broadcast batch dimensions.
* `transpose`
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