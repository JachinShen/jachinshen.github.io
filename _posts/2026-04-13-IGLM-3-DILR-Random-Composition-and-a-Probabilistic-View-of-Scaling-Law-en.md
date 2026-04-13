---
layout: post
author: JachinShen
title:  "IGLM (III): DILR, Random Composition, and a Probabilistic View of Scaling Law"
date:   2026-04-13 10:30:00 +0800
categories: Study
tags:
    - Study
    - MachineLearning
    - LogicMinimization
    - ScalingLaw
---

# IGLM (III): DILR, Random Composition, and a Probabilistic View of Scaling Law

> Note: The concrete implementation details, mathematical organization, and writing of this post were all assisted by GPT. I, as the author, mainly contributed the core idea, research direction, and key judgments.

The first two posts did two things:

1. they made "learning is compression" concrete through the minimal computational element set;
2. they turned learning into a structural problem of iteratively expanding an element set from primitives.

This post finally reaches the algorithm itself.

My requirement at this stage is actually modest. I am not trying to solve optimal complexity, practical throughput, or advanced heuristics first. I only want one clean property:

> If we are allowed to keep adding compute, can we define a simple algorithm under which the explanation of the target structure improves monotonically and approaches the optimal path with nonzero probability?

DILR (Dijkstra-Iterative Logic Rewriting) is designed around that goal.

## Background

If we follow the full-closure definition from the previous post,

$$
E_{t+1}^{\mathrm{full}} = E_t \cup \{ op(g,h) \mid g,h \in E_t,\ op \in \Omega \},
$$

then the number of candidates grows extremely fast:

$$
|E_{t+1}^{\mathrm{full}} \setminus E_t| \le |\Omega|\cdot |E_t|^2.
$$

This is theoretically fine, but in practice it is impossible to enumerate the full closure.

So any realistic approach must replace "generate all binary combinations at each round" with "sample a very small number of candidates at each round." Then the questions become:

- after sampling, how do we define the current best structure?
- if a shorter path is found later, how do we update it?
- why should this process get closer to the optimal structure as compute increases?

Those are exactly the questions DILR is trying to answer.

## Target function and state representation

Again consider a target function

$$
f : X \to \{0,1\},
$$

with primitive set

$$
\mathcal P = \{x_1,\dots,x_n,0,1\},
$$

and binary operator set

$$
\Omega = \{op_1,\dots,op_r\}.
$$

A computational element $g$ is itself a function from $X$ to $\{0,1\}$. So it can be treated directly as a state. For a finite input space, that state can even be represented explicitly as a truth table.

Therefore the DILR state space is the set of all reachable functions. The key idea is not the size of the space, but this rule:

> for each state, keep only the shortest currently known generation path.

## Cost function

Let $c(g)$ be the current best cost for generating element $g$. The simplest definition is the number of newly introduced intermediate elements: each binary composition contributes one extra unit of cost.

Initially, for all primitive elements,

$$
\forall p \in \mathcal P, \quad c(p)=0.
$$

If a new element is generated as

$$
g = op(a,b), \quad a,b \in E_t,
$$

then its candidate cost is

$$
\tilde c(g) = c(a)+c(b)+1.
$$

The $+1$ means we introduced one new composite element. In a more refined implementation, the cost function may account for graph sharing and structural reuse more carefully, but this version is enough to express the core idea.

## Definition of the DILR algorithm

DILR maintains a map

$$
M_t : g \mapsto (c_t(g), \pi_t(g)),
$$

where:

- $g$ is a discovered computational element;
- $c_t(g)$ is its current best cost after iteration $t$;
- $\pi_t(g)$ stores its current best generation path or parent information.

Initialization:

$$
M_0 = \{ (p, 0, \varnothing) \mid p \in \mathcal P \}.
$$

At iteration $t+1$, DILR performs one random expansion:

1. sample two elements $a_t, b_t$ from the current pool $E_t = \mathrm{dom}(M_t)$;
2. sample one binary operator $op_t$ from $\Omega$;
3. construct a candidate element
   $$
   g_t = op_t(a_t,b_t);
   $$
4. compute the candidate cost
   $$
   \tilde c_t = c_t(a_t)+c_t(b_t)+1;
   $$
5. perform relaxation:
   - if $g_t \notin E_t$, insert it into the map;
   - if $g_t \in E_t$ but $\tilde c_t < c_t(g_t)$, replace the old path with the new one;
   - otherwise discard the candidate.

In pseudocode:

```python
M = {p: (0, None) for p in primitives}
for t in range(T):
    a = sample_state(M)
    b = sample_state(M)
    op = sample_operator(Omega)
    g = op(a, b)
    cand = cost[a] + cost[b] + 1
    if g not in M or cand < cost[g]:
        M[g] = (cand, (op, a, b))
```

That is the minimal form of DILR.

## Why it is monotonically improving

The most important property of DILR is simple but crucial.

For any element $g$, let $c_t(g)$ denote its best known cost at iteration $t$. By the relaxation rule,

$$
c_{t+1}(g) = \min\big(c_t(g), \tilde c_t(g)\big),
$$

where if no candidate path for $g$ is generated at this round, we can treat $\tilde c_t(g)=+\infty$.

Immediately we get

$$
c_{t+1}(g) \le c_t(g).
$$

So for every state, DILR never makes the current best explanation worse. In particular, for the target function $f$, if we denote

$$
C_t(f)=c_t(f),
$$

then the sequence

$$
C_0(f), C_1(f), C_2(f), \dots
$$

is monotone non-increasing.

This is the main theoretical value of DILR:

> the training process is not oscillating on an opaque objective; it performs monotone relaxation on structural cost.

## Probabilistic reachability of the optimal path

Monotonicity alone is not enough. We also need to explain why adding compute actually matters.

Let an optimal generation path for the target function $f$ be

$$
\pi^* = (g_1^*, g_2^*, \dots, g_L^*),
$$

where:

- each $g_i^*$ is produced by some binary operator from earlier elements;
- the final element is $g_L^* = f$;
- the path achieves minimal total cost
  $$
  c^*(f)=K_\Omega(f;\mathcal P).
  $$

Now assume that the sampling policy has global support, meaning:

- for any currently discovered elements $a,b$,
  $$
  \Pr[(a_t,b_t)=(a,b)] > 0;
  $$
- for any operator $op \in \Omega$,
  $$
  \Pr[op_t=op] > 0.
  $$

Then as soon as the prefix of the optimal path has been discovered, the probability of generating the correct next element remains strictly positive.

More concretely, suppose that once the first $i-1$ elements of the optimal path are already in the state pool, the single-round lower bound on generating $g_i^*$ is $p_i>0$. Then after $K$ independent effective attempts, the probability of hitting that step at least once satisfies

$$
P_i(K) = 1-(1-p_i)^K.
$$

When $p_i$ is small,

$$
P_i(K) \approx 1-e^{-Kp_i}.
$$

If we compress the critical steps of the path into one effective event and let $q_c$ be the one-round probability of hitting a path that reduces the target cost to at most $c$, then

$$
\Pr[C_K(f) \le c] = 1-(1-q_c)^K \approx 1-e^{-Kq_c}.
$$

This formula already expresses the key phenomenon:

> the value of compute is that it keeps amplifying the cumulative probability of hitting a better structure.

## From this probability model to scaling law

The formula above is still general. To connect it to scaling law, we need one more structural assumption.

The most natural assumption is that lower-cost structures are rarer. Therefore the probability of hitting an improvement path with cost at most $c$ decreases rapidly as $c$ gets smaller. A simple model is

$$
q_c \approx B^{-c}, \quad B>1,
$$

where $B$ can be interpreted as an effective branching factor. The meaning is straightforward:

- to reduce structural cost by one more unit,
- the relevant search path becomes exponentially rarer.

Substitute this into the cumulative-probability formula:

$$
\Pr[C_K(f) \le c] \approx 1-e^{-K B^{-c}}.
$$

When this probability reaches a constant level such as $1-1/e$, we get

$$
K B^{-c} \approx 1.
$$

Therefore,

$$
c \approx \log_B K.
$$

In the simplest version of the model, this means that as compute $K$ increases, the better structural cost the system can hit decreases roughly logarithmically.

If the task error $L(K)$ depends monotonically on the current best structural cost $C_K(f)$, then we naturally get a scaling-law explanation:

- more compute;
- higher probability of hitting better paths;
- lower current best structural complexity;
- lower macroscopic error.

This differs from the usual story where one fits power laws directly from parameter count or dataset size. Here the explanation is more structural:

> scaling law is not fundamentally a law of parameter statistics; it is a law of the probability of hitting better structural paths.

## Why emergence appears

Under this framework, emergence also becomes easy to interpret.

If a key intermediate element, once discovered, can be reused many times, then before it is hit the system behaves like it is still memorizing fragments; after it is hit, the total structural complexity suddenly drops and generalization can jump sharply.

So emergence does not mean that some mysterious capability appears from nowhere. It means:

> DILR has just hit a key intermediate structure with very high compression value.

This is particularly easy to observe in adder-like tasks. Once the relevant carry-related structure is found, many previously scattered samples become unified by one rule, and the system looks like it has suddenly learned addition.

## Why I am not discussing optimization first

DILR can obviously be improved in many ways:

- smarter sampling policies;
- heuristic distances;
- hierarchical search;
- shared-substructure caching;
- parallel relaxation.

But none of those are the most important thing right now.

What I care about first is making the main theoretical line explicit:

1. states are computational elements;
2. new states are produced by random binary composition;
3. for each state, the shortest known path is always kept;
4. therefore the target cost is monotone non-increasing;
5. as iteration count grows, the cumulative probability of hitting better paths rises;
6. scaling law can be interpreted directly through that probability process.

As long as this line is correct, later optimizations have a direction. Otherwise, adding a pile of engineering tricks would just turn the whole thing back into a black-box searcher.

## Conclusion of this post

DILR can be seen as the first core algorithm of IGLM. It does not try to be efficient first. It tries to make three things clear first:

1. definition: how elements are generated;
2. property: how best cost decreases monotonically;
3. interpretation: why more compute improves structure.

In formula form:

- element generation:
  $$
  g_t = op_t(a_t,b_t), \quad a_t,b_t \in E_t,\ op_t \in \Omega;
  $$
- cost relaxation:
  $$
  c_{t+1}(g)=\min(c_t(g), \tilde c_t(g));
  $$
- probabilistic hit:
  $$
  \Pr[C_K(f) \le c] \approx 1-e^{-Kq_c};
  $$
- if $q_c \approx B^{-c}$, then
  $$
  c \approx \log_B K.
  $$

At this point, the main line of IGLM is basically complete:

- the first post defines the object of compression;
- the second post defines learning as iterative structural expansion;
- the third post introduces DILR and gives a probabilistic interpretation of scaling law.

If I continue this series later, the next thing I want to add is an experimental post: directly use addition or other discrete tasks to test whether the current best structural cost of a target function really decreases as iteration count grows, and whether generalization jumps once key structures are hit.