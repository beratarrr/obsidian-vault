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
- Deepwalk (Unif)