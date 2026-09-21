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
Signals that change over the graph give off patterns that we can identify and filters that we can characterise -> initial thought is to count sign changes

##### Gradient at a node
**Edge derivative:** variation of signal x wrt an edge at node i
		![[Pasted image 20260921153235.png|220]]
		![[Pasted image 20260921153350.png|443]]

**Node gradient:** vector collecting all edge derivatives for node i
		![[Pasted image 20260921153416.png|239]]

**Local variability:** norm-2 of the gradient vector
		![[Pasted image 20260921153730.png|310]]
			-> 
			
**Global variation:** sum all square local variations
		![[Pasted image 20260921153735.png|312]]

**Signal smoothness measure:**
		![[Pasted image 20260921153834.png|290]]
			quantifies how much the signal changes over the graph
			![[Pasted image 20260921154017.png|478]]
			$S_2(x_1) = 0 <S_2(x_2) < S_2(x_3)$


### Tikhonov


### Other regularizers