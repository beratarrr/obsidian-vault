### Motivation

There is different signal behavior over graphs in real applications, such as measureemnts in sensor networks, where neighbouring nodes tend to have similar values/labels
	This prior assumption can be used to remove noise, interpolate missing values, classify nodes
In recommender systems, neighbouring users present similar preferences, we can rely on this assumption to predict links -> perform recommendations

Conclusion is that we have the followign observation:
	The signal exhibits similar values in connected ndoes
	The question we end up with is: how can we leverage that to solve graph ML tasks
		-> We need to measure the signal variability over the graph

### Signal variation

Signals that change over the graph give off patterns that we can identify and filters that we can characterise -> initial thought is to count sign changes





### Tikhonov


### Other regularizers