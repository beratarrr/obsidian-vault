glearning = graph learning
##### Graph construction
graph G= (V,E)
- V=set of nodes
- connected by a set of edges E = {(i,j)}

##### Types of nets
- **Physical networks - edges literally exist, you're just recording**
	- Road network, streets exist
	- Robot networks, individual agents/communication exist
	- Brain networks: brain regions, connectivity
- **Abstract net - edges don't exist, you have to construct the graph by how close/relative data points are: build proximity structure from data points**
	- similarity or distance measure
	- sparsification rule; deciding which of the connections to keep as edges
- **Correlation networks, connecting nodes which are statistically correlated**
	- Pearson correlation - ρᵢⱼ = cov(xᵢ,xⱼ) / √(var[xᵢ]var[xⱼ]) — a value between −1 and 1 measuring linear relationship. correlate datapoints provide information to eachother
- **Epsilon neighbourhood graphs, cnnect nodes if theyre within a fixed distance epsilon of eachother**
	- Formula from the slides uses a Gaussian kernel to weight the edge, but only keeps it nonzero when dist(xᵢ,xⱼ) ≤ ε:  Aᵢⱼ = exp(−dist²/2σ²) if dist ≤ ε, else 0
- **K-nearest neighbourhood networks, connect each node to its k closest/most similar points**

##### Two key axes of glearning
- **Learning signal:** semi/un/supervised
- **Generalization setting:** trans/inductive
	- **Transductive:** Have 1 graph, all nodes and edges are visible to the model during training but some of the nodes have lables and some don't. Transductive is when you try to find the labels of nodes within the same graph
	- **Inductive:** Model learns a general rule that it can apply to nodes or graphs that it hasn't seen before, generalizing to new graphs. *E.g. combine features of the neighbour nodes of the new nodes to characterise the new one.*


classical approach to learning over nodes

##### Label propagation
- Assigns c the label that is most reachable from v through the graph
- At each step every node updates their belief by averaging what their neighbours currently believe.
- In a random walk snse, starting at node v, walinkt the graph one edge at a time, the prob that v gets label c is the prob that a walk from v eventually lands on a labelled node with label c
- $Pr(y_v = c) = Σ_w Pr(walk from v reaches w)$ summed over label nodes w with label c

- **Matrix stuff**
	- Transition matrix P: row normalized adjacency, row i tells you the prob of stepping to each neighbour from node i
	- Modified transition matrix: same as P, except labelled node rows are replaced by the identity -> which encodes; once the walker hits a labelled node, its stuck there forever
	- Update rule: $Y^(t+1) = P̃ Y^(t)$, after t steps: $Y^(t) = P̃^t Y^(0)$. Each row of Y is prob distribution over labels, Y^(0) has one hot rows for labelled nodes and zero rows for unlabelled ones
	- 