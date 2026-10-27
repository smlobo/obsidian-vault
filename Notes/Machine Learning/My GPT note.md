# Generative Pre-trained Transformer

# Goal
- Predict the next token (word)
- Do not look at just the token (word). Instead of only taking a word's embedding from a vocabulary, progressively refine it's values with more context from other words in the context.
	- The Attention block refines the plain lookup embedding for a word.
	- The many Attention block layers refine this word vector (GPT3 12288)
## Steps
- Input words/tokens into its embedding via the Embedding Matrix $W_e$
	- Intuition:
		- E(mother) - E(father) $\approx$ E(woman) - E(man) 
		- E(daughter) - E(son) $\approx$ E(woman) - E(man)
	- In addition to embedding, encodes the position in the context
- Several layers of:
	- Attention
		- Query:
			- $\vec{Q_n}$ = $\vec{E_n}$ @ $\vec{W_q}$ 
		- Key:
			- $\vec{K_n}$ = $\vec{E_n}$ @ $\vec{W_k}$
		- Compute: $\vec{Q_i}$ . $\vec{K_j}$
			- Shows how much 1 word relates ( **_attends_** ) to another.
		- Reduce values to prevent `softmax` saturation by dividing by $\sqrt{d_k}$
		- **_Mask_** the _next_ tokens to $-\infty$ so they do not influence previous tokens
		- Compute `softmax` to normalize this `Attention Pattern`
		- Value:
			- $\vec{V_n}$ = $\vec{E_n}$ @ $\vec{W_v}$
		- Complete formula for `Attention` block output:
			- $A=\operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}+M\right)$
			- $Z=AV$
	- Multi Layer Perceptron
		- (Feed Forward)
		- Steps:
			- Linear (Up)
				- $W\uparrow$.$\vec{E_n}$ + $\vec{B_n}\uparrow$
			- Non linear function -> Rectified Linear Unit
				- sets -ve values to 0
				- This loinear to non-linear is called a `neuron`
			- Linear (Down)
				- $W\downarrow$.$\square$ + $\vec{B_n}\downarrow$
	- Note:
		- **the same MLP weights act on each token independently**, while attention mixes information between tokens
- Matmul the last vector of the output to Unembedding Matrix $W_u$
	- Inference:
		- the raw numbers in the final vector are called `logits`
		- then apply softmax (over vocabulary entries)
		- to get a distribution that sums to 1.0 (`probabilities`)
	- Training:
		- the model can use the output at **every position** to predict the token following that position. The causal mask makes those parallel predictions valid.

## GPT-2 Structure
- Model dimension = 768
## GPT-3 Structure
- Total weights = 175B
- organized into 27K matrices
- Context size = 2048
- Model dimension = 12,288
- Data flowing is a matrix of 2048 tokens x 12,288 dimensions
- $W_e$ matrix: 12,288 x 50,257 (embeddings per token x vocabulary of tokens)
- $W_u$ matrix: 50,257 x 12,288
	- unembedding does $W_u$ @ last token of dimension 12,288
	- gives 50,257 `logits`
	- converts to `probabilities`

| Layer                         | Num of weights / parameters                                                                  |
| ----------------------------- | -------------------------------------------------------------------------------------------- |
| Embedding $W_e$               | 12,288 d_embed * 50,257 n_vocab = 617M                                                       |
| Key $\vec{W_k}$               | 128 d_key * 12,288 d_embed \* 96 n_heads \* 96 n_layers = 14.5B                              |
| Query $\vec{W_q}$             | 128 d_query * 12,288 d_embed \* 96 n_heads \* 96 n_layers = 14.5B                            |
| $Value\downarrow$ $\vec{W_v}$ | 128 d_value * 12,288 d_embed \* 96 n_heads \* 96 n_layers = 14.5B                            |
| $Value\uparrow$ / Output      | 12,288 d_embed * 128 d_value \* 96 n_heads \* 96 n_layers = 14.5B                            |
| Up-projection                 | (49,152 (4 x 12,288) n_neurons * 12,288 d_embed + 12,288 n_neuron_bias) \* 96 n_layers = 58B |
| Down-projection               | (12,288 d_embed * 49,152 (4 x 12,288) n_neurons + 12,288 n_neuron_bias) \* 96 n_layers = 58B |
| Unembedding $W_u$             | 50,257 n_vocab * 12,288 d_embed = 617M                                                       |
|                               |                                                                                              |
## Softmax
- distribution to positive numbers that sum to 1.00
- Steps
	- [x1, x2, x3, x4, ...]
	- $e^{x1}$, $e^{x2}$, $e^{x3}$, $e^{x4}$, ... to get positive number
	- Divide each by the sum: $\sum_{i=0}^{n-1}e^{xi}$
	- Sums to 1.00
	- Larger number in input corresponds to largest probability distribution