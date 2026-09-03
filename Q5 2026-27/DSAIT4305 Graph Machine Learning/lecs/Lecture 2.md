The problem is that graphs are discrete, while ML need continuous vectors, we discuss how we can transform discrete graphs into vectors using only structure, not features or labels.

##### Node embeddings
Idea is to embed similar nodes closer to eachother in the embedding space -> how to define node similarity?; gives us 2 q's: what graph similarity should be preserved?/ how to force embeddings to preserve it.

#### Encoder-decoder
![[Pasted image 20260903183136.png|393]]

encoder = lookup table producing an embedding per node
![[Pasted image 20260903183238.png|404]]
decoder = a function that tries to reconstruct some notion of similarity from the embedsÍ

**Shallow encoding**
- Very simple type of encoding where you just look up the column of a specific node in the embedding matrix and just take that as your vector