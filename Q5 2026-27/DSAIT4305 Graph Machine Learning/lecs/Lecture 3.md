### Motivation

There is different signal behavior over graphs in real applications, such as measureemnts in sensor networks, where neighbouring nodes tend to have similar values/labels
	This prior assumption can be used to remove noise, interpolate missing values, classify nodes
In recommender systems, neighbouring users present similar preferences, we can rely on this assumption to predict links -> perform recommendations

Conclusion is that we have the followign observation:
	The signal exhibits similar values in connected ndoes
	The question we end up with is: how can we leverage that to solve graph ML tasks
		-> We need to measure the signal variability over the graph

### Signal variation
#### Over a graph
Signals that change over the graph give off patterns that we can identify and filters that we can characterise -> initial thought is to count sign changes -> Weaknes: it only looks at signs. It ignores how large the differences between neighbors are and it ignores edge weights.

##### Gradient at a node
**Edge derivative:** variation of signal x wrt an edge at node i
		$$\frac{\partial \mathbf{x}}{\partial e_{i,j}}\Big|_i = \sqrt{A_{ij}}\,(x_i - x_j)$$
		![[Pasted image 20260921153350.png|443]]

**Node gradient:** vector collecting all edge derivatives for node i
		![[Pasted image 20260921153416.png|239]]

**Local variability:** norm-2 of the gradient vector
		![[Pasted image 20260921153730.png|310]]
			- Large when $x_i$ differs strongly from its neighbors (weighted by the edge weights). The signal varies more around the node. 
			- It depends only on differences to neighbors: if $x_i$ equals all its neighbors' values, the local variability is 0 no matter how large $x_i$ is. - ▢ 
			- Why $\sqrt{A_{ij}}$ in the edge derivative? (Look at what happens after squaring.)
			
**Global variation:** sum all square local variations
		![[Pasted image 20260921153735.png|312]]

**Signal smoothness measure:**
		![[Pasted image 20260921153834.png|290]]
			quantifies how much the signal changes over the graph
			![[Pasted image 20260921154017.png|478]]
			$S_2(x_1) = 0 <S_2(x_2) < S_2(x_3)$


## Signal variation over graphs

**Question:** how fast does the signal change over the graph? The answer lets us identify patterns and characterize filters.

**Initial idea: count sign changes.** Slide 11 shows three signals on the same graph: constant (0 sign changes), slow-varying (3) and high-varying (4).
- Weakness (my reasoning, not on the slide): it only looks at signs. It ignores how large the differences between neighbors are and it ignores edge weights.
- ▢ Give one concrete example of two signals with the same number of sign changes but very different variation.

### Gradient at a node

**Edge derivative:** variation of signal $\mathbf{x}$ with respect to an edge $e_{i,j} = (i,j)$ at node $i$

$$\frac{\partial \mathbf{x}}{\partial e_{i,j}}\Big|_i = \sqrt{A_{ij}}\,(x_i - x_j)$$

**Node gradient:** vector collecting all edge derivatives at node $i$ (entries for non-neighbors are 0 because $A_{ij} = 0$)

$$\nabla_i \mathbf{x} = \left[\frac{\partial \mathbf{x}}{\partial e_{i,1}}\Big|_i, \dots, \frac{\partial \mathbf{x}}{\partial e_{i,N}}\Big|_i\right]^\top$$

**Local variability:** norm-2 of the gradient vector

$$\|\nabla_i \mathbf{x}\|_2 = \Big(\sum_{j \in \mathcal{N}_i} A_{ij}\,(x_i - x_j)^2\Big)^{1/2}$$

- Large when $x_i$ differs strongly from its neighbors (weighted by the edge weights). The signal varies more around the node.
- It depends only on differences to neighbors: if $x_i$ equals all its neighbors' values, the local variability is 0 no matter how large $x_i$ is.
- ▢ Why $\sqrt{A_{ij}}$ in the edge derivative? (Look at what happens after squaring.)

### Global variation and smoothness measure

Sum of all squared local variations:

$$S_2(\mathbf{x}) = \frac{1}{2}\sum_{i \in \mathcal{V}} \|\nabla_i \mathbf{x}\|_2^2 = \frac{1}{2}\sum_{i \in \mathcal{V}}\sum_{j \in \mathcal{N}_i} A_{ij}(x_i - x_j)^2 = \sum_{(i,j) \in \mathcal{E}} A_{ij}(x_i - x_j)^2 = \mathbf{x}^\top \mathbf{L}\mathbf{x}$$

with $\mathbf{L} = \mathbf{D} - \mathbf{A}$. It quantifies how much the signal changes over the graph. Small means smooth.

- Applies to undirected graphs only (the Laplacian is used).
- ▢ Where does the factor $\tfrac{1}{2}$ come from? (Hint: how often is each edge counted in the double sum?)
- Figure example: $S_2(\mathbf{x}_1) = 0 < S_2(\mathbf{x}_2) < S_2(\mathbf{x}_3)$, where $\mathbf{x}_1$ is constant, $\mathbf{x}_2$ slow-varying and $\mathbf{x}_3$ high-varying.
- Check to do by hand: path graph $1 - 2 - 3$ with $\mathbf{x} = (1, 0, -1)^\top$. Compute $S_2$ from the edge sum and from $\mathbf{x}^\top\mathbf{L}\mathbf{x}$.

### Tikhonov


### Other regularizers