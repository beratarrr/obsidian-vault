
**keeping predictions live/updated without recomputing everything**

Think of a live traffic map with sensors on every road updating every few seconds. Dumb way to do it: every time one sensor updates, recompute predictions for the WHOLE map from scratch. Obviously slow. This project is about only updating the part of the graph that actually got affected by the new info, instead of redoing everything. There's actual recent research on this exact problem (papers from literally the last year, called stuff like Ripple and STAG) so the lit review would be fresh instead of ancient citations.

### Low-latency inference on streaming graphs

**Formulation.** Instead of a static graph, the graph (or its node/edge features) updates continuously — new sensor readings, new edges, new nodes. The question isn't "what's the most accurate GNN" but "how do you keep predictions fresh without recomputing the whole graph on every update."

**Architecture and prior work.** This is a genuinely current systems-research area — most of the relevant papers are from the last year or two, which is good for your literature review since it shows you found recent work rather than textbook material:

- Ripple proposes a generalized incremental programming model that exploits the structure of GNN aggregation functions to propagate only the updates caused by a graph change, rather than recomputing full neighborhoods.Ripple performs fast incremental updates of embeddings arising from changes to graph topology or vertex features, avoiding the redundant computation of vertex-wise and layer-wise approaches [arXiv](https://arxiv.org/abs/2505.12112)
- STAG identifies that the real bottleneck in low-latency GNN serving is the "neighbor explosion" problem — a small local change can force updates across a huge number of nodes as it propagates through layers — and splits the update into a fast collaborative phase and a slower reconciliation phase.STAG's collaborative serving mechanism updates only part of node representations during the update phase, alleviating the neighbor explosion problem, while an additivity-based incremental propagation strategy eliminates duplicated computation [arxiv](https://arxiv.org/pdf/2309.15875)
- InkStream and D3-GNN both tackle the same problem from a distributed-systems angle if you want the scalability challenge to extend beyond a single machine.

**Data plan.** PEMS-BAY/METR-LA traffic datasets are the standard benchmark for this exact setting (streaming sensor updates on a fixed graph) and are freely available, cleaned, and widely used, so no data-generation overhead.

**8-week plan:**

- Wk 1–2: data pipeline, streaming-update simulator (replay the dataset as a stream rather than a static batch), proposal
- Wk 3: full-recompute GNN baseline (rerun the whole model every timestep) + non-graph LSTM baseline
- Wk 4–5: incremental message-passing implementation (your own simplified version of the Ripple/STAG idea)
- Wk 6: neighbor-sampling variant (GraphSAGE-style) as a third comparison point
- Wk 7: latency/throughput benchmarking harness, build the accuracy-vs-latency Pareto frontier
- Wk 8: writeup, notebook, repo

**Team split:** (1) streaming-data pipeline/simulator, (2) full-recompute baseline + LSTM baseline, (3) incremental message-passing implementation, (4) neighbor-sampling implementation + benchmarking harness, (5) Pareto-frontier analysis + literature review + writeup.

**Metrics:** prediction RMSE, per-update latency (ms), throughput (updates/sec) — three genuinely distinct metrics, easy to justify since they trade off against each other.\


**predicting how fast a network will run without actually running it**

So basically every neural net is just a big flowchart of operations, matrix multiplies, activations, whatever. Normally if you want to know how long a model takes to run or how much memory it needs you just... run it and time it. This project flips that: you train a model that looks at the flowchart itself (the graph of operations) and predicts the runtime/memory without executing anything. Kinda like being able to eyeball a recipe and know it'll take 40 min without cooking it. There's already a decent bit of research on this (Microsoft has a paper called DNNPerf doing exactly this) so we wouldn't be starting from zero on the literature side.

### GNN-based performance prediction for compute graphs

**Formulation.** Given a neural network represented as a directed acyclic graph (nodes = operators like Conv2D/MatMul/Attention, edges = tensor dependencies), predict a scalar target — inference latency, training step time, or peak memory — on a specific hardware target.

**Architecture and prior work.** This is an active, well-documented research area, which works in your favor for the literature-review requirement:

- Microsoft's DNNPerf represents a model as its computation graph and adds performance-specific node/edge features, using an attention-based encoder rather than plain GCN/GAT because edge features (which carry real performance information) aren't well exploited by those simpler layers.DNNPerf represents a DL model as a directed acyclic computation graph and designs performance-related features based on the computational semantics of both nodes and edges [Microsoft](https://www.microsoft.com/en-us/research/wp-content/uploads/2021/02/dnnperf.pdf)
- PerfSAGE targets edge-device inference specifically and predicts latency, energy, _and_ memory jointly from a single model, benchmarked against earlier predictors like nn-Meter and Eagle.PerfSAGE predicts inference latency, energy, and memory footprint on an arbitrary DNN graph, addressing a shortcoming of nn-Meter which predicts poorly on hardware-accelerator targets [arXiv](https://arxiv.org/pdf/2301.10999)
- An earlier ICLR-adjacent line of work uses GNNs purely as an architecture encoder for NAS-Bench-101, showing the encoder generalizes to structurally unseen architectures at test time.The model learns a surrogate for architecture performance and demonstrates accurate prediction on previously unseen regions of the search space [arxiv](https://arxiv.org/pdf/2010.10024)

These three papers alone cover most of two team members' "2 papers each" requirement and give you three different architectural choices to justify picking between.

**Data plan.** NAS-Bench-101/201 for a training set with pre-measured metrics; supplement with your own timed runs on DAIC (torch.profiler, averaged over multiple runs to kill warmup noise) for a held-out architecture family — this is what lets you honestly satisfy "have your data ready by the proposal."

**8-week plan (adjust to your actual course calendar):**

- Wk 1–2: computation-graph extraction pipeline (PyTorch → graph), NAS-Bench data ingestion, proposal writeup
- Wk 3: analytical/roofline baseline + GBDT-on-summary-stats baseline
- Wk 4–5: GNN model (start with GraphSAGE or GIN, add attention if time permits), initial training on NAS-Bench data
- Wk 6: DAIC timing harness for your held-out family, generalization experiment (train on CNNs, test on transformers)
- Wk 7: metrics, ablations, writeup
- Wk 8: buffer + notebook/repo polish

**Team split:** (1) graph extraction pipeline, (2) analytical + GBDT baselines, (3) GNN model, (4) DAIC timing harness for ground truth, (5) generalization study + lit review/writeup.

**Metrics:** MAPE, Spearman rank correlation (matters more than absolute error if this were used for architecture search), R².

**Risk to flag to the group:** if the timed-ground-truth collection on DAIC slips, you still have a full project on NAS-Bench data alone — treat your own timings as a stretch goal, not the load-bearing part.



So i basically have 2 ideas, the first one is to train a small model to 

So basically one idea is to train a small model to predict how fast a neural network will run (or how much memory it'll use) just by looking at its structure, without actually running it. Like right now if you wanna know if a model's gonna be too slow or too big for the GPU, you just gotta run it and see. This flips that, you feed the model's shape (basically a flowchart of all its operations) into a smaller predictor model and it just guesses the runtime/memory. There's already  research on this (Microsoft has a paper literally called DNNPerf doing this exact thing) so we're not inventing this from scratch, which makes the lit review a bit easier.

The second one is about keeping live predictions updated without recomputing everything from scratch every time something changes.  for example a traffic map with sensors etc updating every 2 seconds, it would take too much time to run the whole map over againg every single update, a better way to do that could be to only update the part ofthe map that is affected by that update. This one also has some fresh research to it


Think a live traffic map with sensors updating every few seconds — dumb way to do it is recompute predictions for the WHOLE map every single time one sensor changes, which is way too slow. Smart way is to only update the part of the map actually affected by that one change. Same idea applies to any live/streaming graph data. This one's actually got fresh research behind it too, papers from like the last year (Ripple, STAG) so we'd have a current lit review instead of citing ancient stuff.