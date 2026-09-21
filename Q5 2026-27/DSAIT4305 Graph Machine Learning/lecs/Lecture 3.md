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
		$$\nabla_i \mathbf{x} = \left[\frac{\partial \mathbf{x}}{\partial e_{i,1}}\Big|_i, \dots, \frac{\partial \mathbf{x}}{\partial e_{i,N}}\Big|_i\right]^\top$$

**Local variability:** norm-2 of the gradient vector
$$\|\nabla_i \mathbf{x}\|_2 = \Big(\sum_{j \in \mathcal{N}_i} A_{ij}\,(x_i - x_j)^2\Big)^{1/2}$$
			- Large when $x_i$ differs strongly from its neighbors (weighted by the edge weights). The signal varies more around the node. 
			- It depends only on differences to neighbors: if $x_i$ equals all its neighbors' values, the local variability is 0 no matter how large $x_i$ is. 
			- Why $\sqrt{A_{ij}}$ in the edge derivative? (Look at what happens after squaring.)
			
**Global variation:** sum all square local variations
$$S_2(\mathbf{x}) = \frac{1}{2}\sum_{i \in \mathcal{V}} \|\nabla_i \mathbf{x}\|_2^2 = \frac{1}{2}\sum_{i \in \mathcal{V}}\sum_{j \in \mathcal{N}_i} A_{ij}(x_i - x_j)^2 = \sum_{(i,j) \in \mathcal{E}} A_{ij}(x_i - x_j)^2 = \mathbf{x}^\top \mathbf{L}\mathbf{x}$$
**Signal smoothness measure:**
		![[Pasted image 20260921153834.png|290]]
			quantifies how much the signal changes over the graph
			![[Pasted image 20260921154017.png|478]]
			$S_2(x_1) = 0 <S_2(x_2) < S_2(x_3)$
with $\mathbf{L} = \mathbf{D} - \mathbf{A}$. It quantifies how much the signal changes over the graph. Small means smooth.

### Tikhonov regularization
We have a graph signal regularization problem, we have that a true graph signal $x$ is smooth over a graph, for example $S_2(x) = x^T L_x$ is low, we observe a very noisy version of the signal $y=x+n$, the goal is to recover from the observation $y$

- Because the noise is random, it has high variability
- we can solve the Tikhonov regularization problem
	![[Pasted image 20260921162458.png|283]]
- Fitting term: 
	- ![[Pasted image 20260921162628.png|163]]
		- signal close to observation
- Regularization term
	- ![[Pasted image 20260921162605.png|164]]
		- desired prior, signal varition is samll in adjacent ndoes

- Scalar $\gamma > 0$ , controls the trade off 
	- $\gamma \to 0$ we only solve fitting term - less desired prior
	- Q: what happens for $\gamma = 0$
	- $y>>0$ we only solve the desired prior -- less fitting
	- Q: what happens for $\gamma \to \inf$
### Other regularizers