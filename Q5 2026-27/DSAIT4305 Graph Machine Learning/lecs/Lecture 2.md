The problem is that graphs are discrete, while ML need continuous vectors, we discuss how we can transform discrete graphs into vectors using only structure, not features or labels.

##### Node embeddings
Idea is to embed similar nodes closer to eachother in the embedding space -> how to define node similarity?; gives us 2 q's: what graph similarity should be preserved?/ how to force embeddings to preserve it.

#### Encoder-decoder
![[Pasted image 20260903183136.png|393]]

encoder = lookup table producing an embedding per node
![[Pasted image 20260903183238.png|404]]
decoder = a function that tries to reconstruct some notion of similarity from the embeds

**Shallow encoding**
- Very simple type of encoding where you just look up the column of a specific node in the embedding matrix and just take that as your vector
- In the same spirit the decoder also is shallow, it just performs a dot product between 2 vectors to predict their similarity, with no learnable params of its own
![[Pasted image 20260903184118.png|397]]


#### Random walk based methods
An embedding strategy using random walk is to run short fixed-length random walks starting from each node in the graph
	For each node we collect the multiset of nodes visited on random walks starting from the initial node
	Given the initial node, we want to learn a representation that are predictive of the nodes in its random walk neighbourhood

### Building the optimization objective
Given a node we want to learn a representation/embedding that contains some characteristics/are predictive of the nodes in the random walk neighbourhood.

Step 1 is to first assume that the nodes in the neighbourhood are conditionally independent given the embedding
Step 2 is then to parameterize it P(v|z_u):
![[Pasted image 20260921130737.png|343]]
This formula makes all the scores positive by taking exp, and then divide it by the total so they add up to 1, in other words turn the numbers we have into probabilities.

## Optimization objective

Now we optimize the embeddings Z to maximize the likelihood of random walk co-occurrences
This needs a loss function, which is small when model gives a true neighbour a high prob and large when it did not -> this loss function leaves us with a problem -> bc the denominator adds scores over ALL nodes, it has to be recomputed for every pair -> millions nodes -> millions operations per pair
![[Pasted image 20260921132334.png|338]]

A way to tackle this problem is **Negative sampling**, this changes up the questions, instead of "how does v comapre against everyone" we ask "is b a real neighbou or a randomly picked stranger", for each real pair we then pick k random nodes as fake pairs![[Pasted image 20260921132542.png|374]]
Negative sampling usese a sigmoid that puts any score into a number between 0,1. Which gives the possibility of a pair being real. The first term rewards a high score for the real pair, the second rewards a low score for each fake pair, not the whole graph doesnt get recalculated only 1+k nodes each update. SO basically it uses a small random sample instead of the whole graph to compare to the chosen real neighbour .


### Different types of random walks
Based on graph semantics you can choose diff random walks:
- **DeepWalk (Uniform random walk)**
		Fixed-length walks starting at every node.
		-
		At each step the next node is chosen uniformly among the current node's neighbors:  $\mathbf{D}^{-1}\mathbf{W}$, so $\mathbf{P}_{uv} = \mathbf{A}_{uv} / \sum_v \mathbf{A}_{uv}$ 
		-
		A sliding window over each walk produces the training pairs. Example with window size 2 on C→A→B→D→F→E: anchor B gives (B,C), (B,A), (B,D), (B,F)
		- 
		Loss, minimized over the vertex matrix $\mathbf{Z}$ and the context matrix $\mathbf{Z}'$:
			![[Pasted image 20260921133702.png|242]]
		Trained with hierarchical softmax: nodes are leaves of a binary tree, and $P(v \mid \mathbf{z}_u)$ becomes a product of about $\log N$  binary decisions along the path to $v$ This replaces the $O(N)$ denominator with $O(\log N)$work.
- **Node2Vec (Biased random walk to exploit biased breadth/depth first searches)**
		Also fixed-length walks from every node, but the walk is second order: the next step depends on the current node $v$ and on the node $t$ it just came from.
		-
		For each neighbor $x$ of $v$ the unnormalized transition weight is:
		![[Pasted image 20260921135157.png|514]]
		where $d_{tx}$​ is the shortest-path distance between tt t and $x$. The weights are normalized over the neighbors of $v$.
		-
		If the walk stays close to its starts it behaves BFS-like and captures local structure and structural roles. If it goes deeper it behaves DFS-like and captures broader community structure. Small q = move outwards, larger q = stay close
- **NERD (alternating walks for directed graphs)**
		Problem with DeepWalk and node2vec on directed graphs: walks can enter regions with no outgoing edges and leave neighborhoods unexplored, and a single embedding is direction-agnostic. It cannot encode that edge C→AC exists while A→CA does not.
		- 
		Each node gets two embeddings, one as a source and one as a target, stored in separate matrices $\mathbf{Z}^{(\text{source})}$ and $\mathbf{Z}^{(\text{target})}$.
		-
		Walks alternate roles. A source walk such as C→A←B→D←F→EC goes source → target → source → target. A target walk starts from a node in the target role.
		Loss for roles $r_1, r_2 \in \{\text{source}, \text{target}\}$:
		
		$\mathcal{L}\big(u^{(r_1)}, v^{(r_2)}\big) = -\log \sigma\big(\mathbf{z}_u^{(r_1)\top} \mathbf{z}_v^{(r_2)}\big) - \sum_{i=1}^{k} \log \sigma\big(-\mathbf{z}_u^{(r_1)\top} \mathbf{z}_{w_i}^{(r_2)}\big)$

- The negative sample $w_i$  takes the same role as the positive context node. The noise distribution uses in-degree or out-degree depending on that role: $P_n(v) \propto d(v)^{3/4}$

### Matrix factorisation
Directly factorize the node sim matrixt into embedding matrices

First the target matrix is defined: 
	S_uv is how similar u and v are
	The simplest choice is to equal S to A (1 if there is an edeg and 0 if not)
The goal is to approximate S_uv with dot products of the vectors. A big dot product = similar, dissimilar = small dot product

The key idea is to learn a low dimensional approximation of a node-node sim matrix S by factorisation.
Put all vectors into one matrix Z, and then multiply Z by its tanspose after which it hodls every pairwise dot product at once, -> we want to minimize the gap to the target after
	![[Pasted image 20260921141650.png|192]]

The problem with directed graphs is that the dot product does not care about order, but in a directed graph it could happen that a walk ends up in a place with no return, which the dot product alone cant see, so to fix we give each node two vectors, in other words apply the logic from NERD

#### Hope
Hope has to decide 2 things, which sim to use and how to factorize it without large cose.

Which similarity: KATZ
Count the walks between 2 nodes, but make the short walk count more:
	$S=βA+β2A2+β3A3+⋯$
	- $(\mathbf{A}^k)_{ij}$ is the number of walks of length $k$ from $i$ to $j$.
	- $\beta$ is a discount below 1, so a walk of length $k$ is weighted by $\beta^k$ and long walks matter little.
	- It is asymmetric: with u→w→v there is a walk from $u$ to $v$ but not the other way, so $\mathbf{S}_{uv} > \mathbf{S}_{vu}$
	- The sum only stays finite if $β$ is small enough: $\beta < 1/\rho(\mathbf{A})$, where $\rho(\mathbf{A})$ is the largest absolute eigenvalue of $\mathbf{A}$ (The slide says $\beta \in (0,1)$, which is not enough on its own.)

How do we factorize this cheaply? The infinite sum has a compact form:
	$S=(I−βA)−1βA=Mg−1​Ml​$
Hope avoids building S directly which would mean inverting a big matrix and storing a dense matrix. By running generalized SVD on the two sparce matrices $M_g$ and $M_l$ and reads off $U_s$ and $U_t$ from the top singular vectors, the dense $S$ is never created
#### Random walks versus factorization

- Random walks scale better to big sparse graphs.
- Factorization is easier to interpret, since you know exactly which matrix you are approximating.
- Some random walk objectives secretly amount to factorizing a matrix, so the two families are closer than they look.

#### What is wrong with all shallow methods

1. **Too many parameters:** one vector per node, so the table grows with the graph.
2. **No new nodes:** a node that was not in training has no column, so you cannot embed it.
3. **No features:** node or edge information (text, age and so on) cannot be used.

The suggested cure is to replace the table with a function that computes a vector from a node's features and neighbors. That leads to graph neural networks.

#### Using the vectors (downstream tasks)

- **Node classification:** use zi\mathbf{z}_i zi​ as input features to predict a node's label.
- **Link prediction:** score a pair with zi⊤zj\mathbf{z}_i^\top \mathbf{z}_j zi⊤​zj​, or combine the two vectors (concatenate, multiply per coordinate, add or take the distance) and feed the result to a classifier.
- **Clustering:** run k-means on the vectors.
- **Graph classification:** average all node vectors into one graph vector, then classify it.