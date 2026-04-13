---
layout: post
author: JachinShen
title:  "IGLM (II): From Sample Attachment to Iterative Structural Expansion"
date:   2026-04-13 10:00:00 +0800
categories: Study
tags:
    - Study
    - MachineLearning
    - LogicMinimization
    - IncrementalLearning
---

# IGLM (II): From Sample Attachment to Iterative Structural Expansion

> Note: The concrete implementation details, mathematical organization, and writing of this post were all assisted by GPT. I, as the author, mainly contributed the core idea, research direction, and key judgments.

In the previous post, I first pinned down the object behind the phrase "learning is compression": the minimal computational element set,

$$
K_\Omega(f;\mathcal P)
=
\min_{S \leadsto f} |S \setminus \mathcal P|.
$$

This post goes one step further. Once the optimal structure is defined, how should the learning process itself be understood?

My view is that the training process of IGLM naturally splits into two stages:

1. attach the samples first so that no information is lost;
2. then use iterative expansion and structural reuse to compress redundant explanations into a smaller element set.

This is quite different from a standard neural-network story. There, optimization starts immediately in a continuous parameter space. In IGLM, the process looks more like building materials first and rewriting structure later.

## Background

If we only discuss the final minimal element set, the whole problem becomes too static. In an actual learning process, the model cannot jump to the optimal structure from the beginning. It has to accumulate intermediate elements gradually.

This is similar to many engineering problems:

- you do not write the final optimal program immediately;
- you usually write a version that works first;
- then you identify repeated logic and extract common modules;
- finally you tighten the structure.

IGLM follows the same logic. The difference from standard straight-line complexity is that I do not only want to define the final optimal size. I also want to model the path from data to structure as the learning process itself.

## Sample attachment: ensure representability first

Let the training set be

$$
\mathcal D = \{(x^{(1)}, y^{(1)}), \dots, (x^{(N)}, y^{(N)})\}.
$$

To keep the discussion simple, I still focus on a single-output Boolean task with $y^{(i)} \in \{0,1\}$.

The most direct learning strategy is to first construct a system that is perfectly correct on the training set. In other words, we first require

$$
\forall (x,y) \in \mathcal D, \quad \hat f(x)=y.
$$

In IGLM, I call this step attachment.

Its purpose is not to claim that the resulting structure is already good. The purpose is:

- to write the current data constraints into the system completely;
- not to compress too early;
- to provide enough structural material for later merging.

In the most naive version, each sample may correspond to its own local explanation structure. The resulting system is large, but it has one important advantage: it definitely covers the current training set.

Conceptually, this stage is similar to a redundant lookup table, except that the process does not stop there. It continues toward structural compression.

## Mathematical definition of iterative expansion

Once we have the attachment stage, the next step is to keep building higher-level elements from existing ones.

Again let the primitive set be

$$
\mathcal P = \{x_1,\dots,x_n,0,1\},
$$

and the allowed binary operator set be

$$
\Omega = \{op_1, op_2, \dots, op_r\}.
$$

Define $E_t$ as the set of computational elements the model has after iteration $t$. Initialize with

$$
E_0 = \mathcal P.
$$

If we only care about the full theoretical expansion, then the closure at the next round is

$$
E_{t+1}^{\mathrm{full}} = E_t \cup \{ op(g,h) \mid g,h \in E_t,\ op \in \Omega \}.
$$

This definition is very brute-force, but it makes the problem clear:

> Learning is not fine-tuning a function in parameter space. It is expanding the reachable set in element space.

If the target function $f$ first appears in the set at iteration $t$, then the system has acquired the ability to express it:

$$
D_\Omega(f;\mathcal P)=\min\{t \mid f \in E_t^{\mathrm{full}}\}.
$$

The full expansion is far too expensive in practice, because each round needs to consider all pairwise combinations. It is theoretically clean but computationally unrealistic.

So the real point of IGLM is not to compute the full closure, but to perform incremental expansion:

- expand only a small subset of candidates;
- but retain the chance of covering important structures over time;
- and once a better structure is found, rewrite the old one.

## From element sets to a structural graph

Although I do not want to bind the implementation to any single graph representation, it is useful to think of every element $g \in E_t$ as a node, while the information "which two older elements and which operator produced it" acts like its provenance.

Then the learning process maintains a directed acyclic construction graph:

- primitive elements are source nodes;
- intermediate elements are internal nodes;
- target outputs are terminal nodes;
- every edge represents a computational dependency.

At that point, the minimal computational element set from the previous post corresponds to:

> among all construction graphs that generate the target output, the one with the smallest number of nodes.

If training only attaches samples and never merges anything, we get many nearly parallel subgraphs with very little sharing. If training can repeatedly identify common substructures, those scattered subgraphs begin to merge and form shorter common paths.

That is what I mean by compression.

## Why "attach first, compress later" makes sense

A natural objection here is: why not search for the optimal structure immediately, instead of allowing redundancy first?

The answer is simple: when the data is still sparse and the structure has not fully emerged, compressing too early easily leads to the wrong rule.

This is especially clear in discrete tasks. If we have only observed a few samples, there are many short structures that fit the current training set. If we force the learner to pick one of the shortest structures at that stage, it may just be a coincidental fit rather than the true rule of the task.

By contrast, attaching more samples first means writing more constraints into the system. As the sample set grows, those opportunistic short structures get eliminated, while the genuinely stable shared structure remains.

So "attach first, compress later" is not wasted effort. It creates the conditions under which the optimal structure becomes identifiable.

## A more explicit form: minimum element set under empirical constraints

The idea above can be written as a constrained optimization problem.

For the training set $\mathcal D$, define the empirical error

$$
\mathcal L_\mathcal D(g) = \frac{1}{|\mathcal D|}\sum_{(x,y)\in\mathcal D} \mathbf 1[g(x) \neq y].
$$

If we only consider exact fitting, the learning target is

$$
\min_{g} K_\Omega(g;\mathcal P)
\quad
\text{s.t.}
\quad
\mathcal L_\mathcal D(g)=0.
$$

This is already very close to the learning definition I want:

> among all structures that explain the training set correctly, choose the one with the smallest number of elements.

If we allow approximate compression, we can also write

$$
\min_{g} \big( K_\Omega(g;\mathcal P) + \lambda \mathcal L_\mathcal D(g) \big),
$$

where $\lambda > 0$ controls the tradeoff between fitting and compression.

For this series, however, I am still mainly interested in the exact-fit case $\mathcal L_\mathcal D(g)=0$.

## Why generalization appears naturally

From this viewpoint, generalization is not mysterious at all.

Suppose the training set covers only part of the input space. If the system only memorizes point by point, then it has almost no basis outside the seen set, and generalization is weak by construction.

But if the training process really finds a small element set that corresponds to a well-defined computational structure over the full input space, then that structure automatically extends to unseen inputs.

At that point, generalization quality depends on whether the compressed structure is truly the internal rule of the task, rather than an accidental recombination of the seen samples.

So the IGLM view of generalization is:

> Generalization is not an extra bonus. It is the natural extension of a successfully extracted shared structure.

## A simple example: pointwise memorization vs shared intermediates

Consider the sum bit of a 1-bit full adder:

$$
s = a \oplus b \oplus c_{in}.
$$

If we memorize all 8 input-output cases point by point, we may need almost as many local structures as samples.

But if we first construct an intermediate element

$$
t = a \oplus b,
$$

and then construct

$$
s = t \oplus c_{in},
$$

then all 8 cases are unified by only two key intermediate elements. Moreover, the carry bit

$$
c_{out} = (a \land b) \lor (c_{in} \land (a \oplus b))
$$

can reuse the same intermediate $t = a \oplus b$.

This is exactly the structural phenomenon I care about:

- once an intermediate element is discovered,
- it no longer serves a single sample,
- it serves many input points and possibly multiple output bits,
- and the total element count of the whole system drops.

## Conclusion of this post

At this point, the learning process of IGLM can be summarized in one sentence:

> Learning = under the constraints of the training samples, iteratively expand the computational element set and repeatedly compress redundant explanations into smaller shared structures.

More concretely:

1. the initial element set is $E_0=\mathcal P$;
2. new elements are generated by binary operators over existing elements;
3. sample attachment guarantees that current training information is representable;
4. structural compression aims at approaching the minimal element set;
5. once shared intermediates are found, total complexity decreases and generalization emerges naturally.

In the next post, I will turn this process into a concrete algorithm: DILR. Its core is not to compute the full closure at once, but to randomly pick elements from the current pool, combine them with binary operators, and always keep the shortest currently known path for the same resulting function. That gives a monotonic process where more compute can buy better structure, which in turn leads naturally to a probability-based explanation of scaling law.