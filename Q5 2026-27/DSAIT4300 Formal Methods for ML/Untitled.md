## DiRL (the ancestor)

_Paper: "Compositional Reinforcement Learning from Logical Specifications" — Jothimurugan, Bansal, Bastani, Alur, NeurIPS 2021._

**Premise.** Writing reward functions by hand is hard and easy to get wrong. A better interface is to describe the task in a logical language, but earlier methods try to learn one single policy for the whole task, and that does not scale to long multi-step tasks because success is too rare to learn from.

**What they try to do.** Learn a policy that maximizes the probability of satisfying a logical task description, in a way that scales to complex long-horizon continuous-control tasks.

**What they build to do it.** Two things working together. First, an **abstract graph** compiled automatically from the spec, where dots are regions of the world and arrows are small reach-avoid subtasks. Second, the **DiRL algorithm**, which combines shortest-path planning over that graph with reinforcement learning for each arrow. The spec is written in the SpectRL language from the earlier paper "A Composable Specification Language for Reinforcement Learning Tasks" (Jothimurugan, Alur, Bastani, NeurIPS 2019).

**The steps DiRL takes.**

1. Compile the SpectRL spec into the abstract graph (proven to capture the spec exactly, with size growing only linearly in the spec length).
2. Give each arrow a cost equal to the negative log of its success probability, so that adding costs along a path equals multiplying success probabilities.
3. Grow outward from the start like Dijkstra, and lazily train a small RL policy for an arrow only when the planner first needs that arrow's cost, using the real arrival distribution from earlier arrows.
4. Run shortest-path planning to pick the most reliable route to the goal.
5. Return a stateful path policy that runs the arrow policies in sequence, with a guarantee that the chance of finishing the task is at least exp of minus the path cost.

## AutoSpec (the paper you present)

_Paper: "Automating the Refinement of Reinforcement Learning Specifications" — Ambadkar, Žikelić, Verma, ICLR 2026._

**Premise.** Spec-guided RL only works if the task description is detailed enough to guide learning. Coarse or vague descriptions, like a target region that is far too big or a missing safety rule, leave the agent unable to learn, and today a human has to notice this and rewrite the spec by hand.

**What they try to do.** Rewrite a coarse spec automatically so the agent can learn it, while guaranteeing the rewrite never changes the actual goal (soundness).

**What they build to do it.** Four **refinement operators** that edit the abstract graph, wrapped in a **monitoring loop** that sits on top of any existing spec-guided learner. They test it on two base learners, DiRL from the ancestor paper and LSTS from "Logical Specifications-guided Dynamic Task Sampling for Reinforcement Learning Agents" (Shukla et al., ICAPS 2024).

**The steps AutoSpec takes.**

1. Run the base learner, then check each arrow's success probability against a target (they use 99 percent).
2. For any failing arrow, sample the agent's own attempts and try four fixes in a fixed order from smallest to biggest change: tighten the target and safety zones, add a waypoint, split the starting region with a dividing line, or reroute through other arrows.
3. Accept the first fix that pushes the arrow above the target, and enforce that every fix only tightens the spec so the new task still implies the original (their soundness theorem).
4. Update the graph, relearn the affected policies, and repeat until the whole task is learned well enough. Because the underlying problem is undecidable, it is sound but not complete, so it will sometimes fail to find a fix.

## How AutoSpec builds on DiRL

- It reuses DiRL's core invention, the **abstract graph**, as the thing it edits. Every one of its four fixes is a modification to that graph, and the graph itself traces back through DiRL to the SpectRL language paper (2019).
- It reuses DiRL's **per-arrow success probability** as its trigger, refining exactly the arrows DiRL reports as weak.
- It literally **runs DiRL as one of its two base learners** (the other being LSTS, Shukla et al. 2024).
- It extends DiRL where DiRL stops. DiRL takes the graph as fixed and, if an arrow is unreliable, just accepts a weak exp of minus cost bound. AutoSpec instead rewrites that arrow so it becomes learnable, turning DiRL's fixed graph into a self-correcting one.

One-line version: DiRL made a logical spec learnable by turning it into a graph of subtasks with reliable-route planning, and AutoSpec keeps that whole machine but adds the ability to repair the graph on its own whenever a subtask is too hard.