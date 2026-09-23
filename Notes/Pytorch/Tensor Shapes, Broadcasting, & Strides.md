# Predict Shapes
```
import torch

device = "mps" if torch.backends.mps.is_available() else "cpu"
B, T, D, H = 2, 4, 8, 16  # batch, tokens, input width, hidden width

x = torch.randn(B, T, D, device=device)
weight = torch.randn(D, H, device=device)
bias = torch.randn(H, device=device)

projected = x @ weight + bias
scores = projected @ projected.transpose(-2, -1)
probabilities = torch.softmax(scores, dim=-1)

for name, tensor in [
    ("x", x),
    ("projected", projected),
    ("scores", scores),
    ("probabilities", probabilities),
]:
    print(name, tensor.shape, tensor.dtype, tensor.device, tensor.stride())

print("row sums:", probabilities.sum(dim=-1))
```

## Prediction
```
x.shape == (2, 4, 8)
weight.shape == (8, 16)
bias.shape == (16)
projected.shape == (2, 4, 8) @ (8, 16) + (16)
	= (2, 4, 16) + (16)
	= (2, 4, 16)
scores.shape == (2, 4, 16) @ (2, 4, 16).transpose(-2, -1)
	= (2, 4, 16) @ (2, 16, 4)
	= (2, 4, 4)
probabilities.shape == (2, 4, 4)
probabilites.sum.shape == (2, 4, 4).sum(-1)
	= (2, 4)
```

# Further Questions
- Change `T` from `4` to `7`. Which tensor shapes change?
```
x.shape == (2, 7, 8)
weight.shape == (8, 16)
bias.shape == (16)
projected.shape == (2, 7, 8) @ (8, 16) + (16)
	= (2, 7, 16) + (16)
	= (2, 7, 16)
scores.shape == (2, 7, 16) @ (2, 7, 16).transpose(-2, -1)
	= (2, 7, 16) @ (2, 16, 7)
	= (2, 7, 7)
probabilities.shape == (2, 7, 7)
probabilites.sum.shape == (2, 7, 7).sum(-1)
	= (2, 7)
```
- Change `bias` to shape `[T]`. Why does the addition fail?
```
x.shape == (2, 4, 8)     x.stride() = (32, 8, 1)
weight.shape == (8, 16)  weight.stride() = (16, 1)
bias.shape == (4)        bias.stride() = (1)
projected.shape == (2, 4, 8) @ (8, 16) + (4)
	= (2, 4, 16) + (4)
```
Broadcasting only works when the matching dimensions are equal _or_ 1.
- Convert `x`, `weight`, and `bias` to `float16` before calculating `projected`. Print its dtype and compare its values with the `float32` result. Dtype affects both representation and compiler choices.
```
probabilities.sum(dim=-1) does not change noticibly - [[1., 1., 1., 1.], 
	[1., 1., 1., 1]]
but projected has a noticible delta
```
- print `projected.stride()` and `projected.transpose(-2, -1).stride()`. The transpose swaps how the last two dimensions index the same underlying storage; it normally does not copy the tensor.
```
projected.shape = (2, 4, 16)
projected.stride() = (64, 16, 1)
projected.transpose(-2, -1).shape = (2, 16, 4)
projected.transpose(-2, -1).stride = (64, 1, 16)
```
