---
layout: post
author: JachinShen
title:  "IGLM (I): Quantifying Intelligence as a Minimal Computational Element Set"
date:   2026-01-18 14:00:00 +0800
categories: Study
tags:
    - Study
    - MachineLearning
    - LogicMinimization
    - CircuitComplexity
---

# IGLM (I): Quantifying Intelligence as a Minimal Computational Element Set

> Note: The concrete implementation details, mathematical organization, and writing of this post were all assisted by GPT. I, as the author, mainly contributed the core idea, research direction, and key judgments.

I have wanted for a while to make the phrase "learning is compression" more precise. The problem is that this phrase is common, but most of the time it stays at the level of metaphor. What exactly is being compressed? Why does compression produce generalization? If those questions are not attached to a concrete mathematical object, it is hard to keep going.

That is the starting point of this IGLM series.

This first post does not discuss a concrete implementation, nor does it discuss complexity optimization. It only focuses on the central question:

> If learning is compression, what exactly is the object being compressed?

My answer is:

> The intelligence complexity of a task can be characterized by the size of the minimal set of computational elements required to explain it.

This is different from parameter count, token count, or loss. It is closer to circuit complexity or straight-line program complexity, but what I care about here is its meaning for learning: instead of "adjusting" a rule in continuous parameter space, the model is "finding" a rule in discrete structure space.

## Background

Su Jianlin's article on deriving scaling laws from a quantization hypothesis gives an illuminating viewpoint: model capability can be seen as the gradual coverage of many discrete capability quanta. That viewpoint is natural, but it still leaves one question open:

What are those quanta in computational terms?

If we only say that a model has learned more and more capability units, the statement is still abstract. To push the idea further, I want capability quanta to correspond to an object that is compositional, enumerable, and countable.

For finite discrete tasks, the most natural candidate is the computational element. For a Boolean task or a discrete mapping task, "understanding the rule" means finding a sufficiently small collection of computational fragments whose composition reproduces the target function.

## Problem Definition

Start from the simplest setting. Let the input space be a finite set $X$, and let the target function be

$$
f: X \to \{0,1\}.
$$

If the input is an $n$-bit binary vector, then we can take $X = \{0,1\}^n$.

We are given a primitive set

$$
\mathcal P = \{x_1, x_2, \dots, x_n, 0, 1\},
$$

and a set of allowed binary operators

$$
\Omega = \{op_1, op_2, \dots, op_r\}.
$$

Here $x_i$ are input bits, and $0,1$ are constants. The exact choice of $\Omega$ is not the key point. The key properties are:

- the elements are discrete;
- the composition rules are finite;
- new elements can be constructed iteratively from old ones.

## Definition of the Minimal Computational Element Set

To keep "compression" from staying a slogan, I give a direct definition.

### Computational sequence

Call a sequence

$$
S=(g_1, g_2, \dots, g_L)
$$

a computational sequence for the target function $f$ if for every $g_t$, one of the following holds:

1. $g_t \in \mathcal P$;
2. there exist $i,j < t$ and $op \in \Omega$ such that
   $$
   g_t = op(g_i, g_j).
   $$

and for some $t$ we have $g_t = f$.

If we only count non-primitive elements, then the number of newly introduced computational elements is

$$
|S \setminus \mathcal P|.
$$

This gives the element complexity of $f$ under operator set $\Omega$:

$$
K_\Omega(f;\mathcal P)
=
\min_{S \leadsto f} |S \setminus \mathcal P|.
$$

This is what I mean by "the number of elements in the minimal computational element set".

In plain words:

> Starting from primitive inputs and using binary operators from $\Omega$, how many additional intermediate computational elements are minimally needed to construct the target function $f$?

For a multi-output function

$$
F=(f_1,f_2,\dots,f_m), \quad f_i: X \to \{0,1\},
$$

the definition is analogous:

$$
K_\Omega(F;\mathcal P)
=
\min_{S \leadsto F} |S \setminus \mathcal P|,
$$

where all output bits must be representable from $\mathcal P \cup S$.

### Why this quantity is a good intelligence measure

If we do no compression at all, the dumbest strategy is to memorize each sample one by one. That can still produce a correct system, but it is not a structural understanding of the task.

A genuine rule is discovered when:

- many seemingly unrelated samples
- are found to share the same intermediate computational elements,
- and what originally required many separate explanations
- can now be explained by a much smaller common structure.

Then $K_\Omega(f;\mathcal P)$ naturally has two useful meanings:

1. it separates memorization from rule discovery;
2. it measures the degree of compression.

From this viewpoint, learning is not only about reducing training error. It is about reducing the number of computational elements needed to explain the task.

## The Iterative Expansion View

The definition above gives a target, but an actual learner will not enumerate the optimal sequence from the full space directly. It will approach it by iteratively expanding an element set.

Let $E_t$ be the set of discovered elements after iteration $t$. Initialize with

$$
E_0 = \mathcal P.
$$

If we ignore search strategy and only keep the theoretical closure semantics, then the next round is

$$
E_{t+1} = E_t \cup \{ op(g,h) \mid g,h \in E_t,\ op \in \Omega \}.
$$

This definition matters for two reasons:

1. it gives the most direct constructive semantics: new elements come from binary combinations of old ones;
2. it turns learning into a problem of expanding the reachable element set.

The minimal depth at which $f$ first becomes reachable can then be written as

$$
D_\Omega(f;\mathcal P)=\min \{t \mid f \in E_t\}.
$$

But $D_\Omega$ only measures depth. It does not measure the total number of introduced elements. In this series, I care more about total structural complexity, so $K_\Omega$ is the main compression metric.

## From Sample Memorization to Structural Compression

Once this definition is in place, the learning goal of IGLM can be restated.

Let the training set be

$$
\mathcal D = \{(x^{(1)}, y^{(1)}), \dots, (x^{(N)}, y^{(N)})\}.
$$

The most direct strategy is to assign an explanation path to each sample. That guarantees correctness on the seen data, but it usually scales nearly linearly with the number of samples and gives almost no generalization.

If the system can extract shared substructures from these paths, it may find a computational element set much smaller than the size of sample-by-sample memorization. That is what I call structural collapse:

- initially separate explanation paths
- start sharing the same intermediate elements,
- the total number of elements goes down,
- and a shorter unified explanation appears.

At that point, generalization is no longer mysterious. It simply means:

> Many training samples are now explained by the same intermediate structure, so unseen samples may also fall within the coverage of that structure.

## Why this is closer to the essence than parameter count

Parameter count is useful, but it does not directly equal the structural complexity of a task. Two models with the same parameter count may represent very different levels of structure, and the same task may be represented very redundantly.

By contrast, $K_\Omega(f;\mathcal P)$ asks a more direct question:

> Given a set of computational primitives and composition rules, how much structure is actually needed to solve the task?

That is why I prefer to see it as an approximate measure of task intelligence complexity. The closer a learner gets to this minimum, the closer it gets to a structural understanding. If it stays far above this minimum, it is likely still relying on memorization or redundant explanations.

## A tiny example: XOR

To keep the definition concrete, consider two inputs $x_1, x_2$ and the target function

$$
f(x_1,x_2)=x_1 \oplus x_2.
$$

Let

$$
\mathcal P = \{x_1, x_2, 0, 1\},
$$

and

$$
\Omega = \{\land, \lor, \neg\}.
$$

Then XOR can be written as

$$
x_1 \oplus x_2 = (x_1 \lor x_2) \land \neg(x_1 \land x_2).
$$

So XOR is not primitive, but it can be built from a small set of intermediate elements. Compared with memorizing all four input-output points directly, this representation is clearly shorter and closer to the rule itself.

In larger tasks like addition, structural reuse is even more obvious. Different samples are not independent. They share carry logic, local combinations, and repeated subpatterns. If those shared elements can be extracted, the total structural complexity drops significantly.

## Conclusion of this post

At this point, the theoretical starting point of IGLM is already clear.

What I care about is not only whether a model can reduce loss, but:

1. given primitive set $\mathcal P$;
2. given binary operator set $\Omega$;
3. what is the minimal computational element set size $K_\Omega(f;\mathcal P)$ for the target function $f$;
4. can a learning algorithm approach this minimum?

If this question is the right one, then the phrase "learning is compression" finally has an exact mathematical object behind it.

In the next post, I will continue with the iterative-expansion side: if the goal is to start from primitives and expand toward higher-level elements step by step, how should that process be formalized, and why does it naturally correspond to the learning paradigm of "first memorize, then compress"?