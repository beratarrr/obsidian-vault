glearning = graph learning
##### Graph construction
graph G= (V,E)
- V=set of nodes
- connected by a set of edges E = {(i,j)}

##### Types of nets
- **Physical networks - edges literally exist**
	- Road network, streets exist
	- Robot networks, 
	- Brain networks
- **Abstract net - build proximity structure from data points**
	- 
- **Correlation networks**
	- Pearson correlation - correlate datapoints provide information to eachother
- **Epsilon neighbourhood graphs**
- **K-nearest neighbourhood networks**

##### Two key axes of glearning
- **Learning signal:** semi/un/supervised
- **Generalization setting:** trans/inductive
	- **Transductive:** Have 1 graph, all nodes and edges are visible to the model during training but some of the nodes have lables and some don't. Transductive is when you try to find the labels of nodes within the same graph
	- **Inductive:** Model learns a general rule that it can apply to nodes or graphs that it hasn't seen before, creating new graphs. *E.g. combine features of the neighbour nodes of the new nodes to characterise the new one.*


classical approach to learning over nodes