---
layout: post
author: JachinShen
title:  "Mathematical Foundation of Computer Science Note 1"
subtitle: "Howework1 Hint & Infinite Sets"
date:   2018-03-01  12:49:50 +0800
categories: Math
tags: 
    - Math
    - Study
    - ComputerScience
---
# homework

## Feasible Intersection Patterns

### Problem

Let us call a sequence $(a_1, a_2, \dots, a_n) \in \mathbb{N}_0 \quad feasible$ if there are sets $A_1, \dots, A_n$ such that all k-wise intersections have size $a_k$. 

That is, $\lvert A_i \rvert = a_1$ for all $i$, $\lvert A_i \cap A_j \rvert = a_2$ for all $i \neq j$ and so on. 

For example, $(5, 3, 1, 0)$ is not feasible, but $(6, 3, 1, 0)$ is.

### Hint

A problem may become **easier** if you:

1. make it more general
2. change it a little bit

### A-Table

#### Definition

Given $A_1, A_2, \dots, A_n$ , $I \subseteq {1, 2, \dots, n}$

define $A_I = \bigcap_{i \subseteq I} A_i$

#### Example

$$A_1 = \lbrace 1, 2, 3, 4 ,7 \rbrace $$

$$A_2 = \lbrace 3, 4, 5, 6 \rbrace $$

$$A_3 = \lbrace 1, 2, 3, 5, 8, 9 \rbrace$$

Then, $A_{ \lbrace 1, 2\rbrace } = A_1 \bigcap A_2 = \lbrace 3, 4 \rbrace$

#### Intersection table

| I                   | {1}   | {2}   | {3}   | {1, 2} | {1, 3} | {2, 3} | {1, 2, 3} |
| :-----------------: | :---: | :---: | :---: | :----: | :----: | :----: | :-------: |
| $\lvert A_I \rvert$ | 5     | 4     | 6     | 2      | 3      | 2      | 1         |

#### $(a_1, a_2, \dots, a_n)$ to A_Table

| I                   | {1}   | {2}   | $\dots$ | {n}   | {1, 2} | $\dots$ | {n-1, n} | $\dots$ | {1, 2, $\dots$, n} |
| :-----------------: | :---: | :---: | :-----: | :---: | :----: | :-----: | :------: |
| $\lvert A_I \rvert$ | $a_1$ | $a_1$ | $\dots$ | $a_1$ | $a_2$  | $\dots$ | $a_2$    | $\dots$ | $a_n$              |

#### Is it easy to judge the table feasible?

Absolutely, **No!!!**

### B-Table

#### Definition of B-Table

Given $A_1, A_2, \dots, A_n$, $\varnothing \neq I \subseteq [n]$

define $B_I = (\bigcap_{i \in I} A_i) \setminus (\bigcup_{j \notin I} A_j)$

#### Example of B-Table

$$A_1 = \lbrace 1, 2, 3, 4 ,7 \rbrace $$

$$A_2 = \lbrace 3, 4, 5, 6 \rbrace $$

$$A_3 = \lbrace 1, 2, 3, 5, 8, 9 \rbrace $$

Then, $B_{ \lbrace 1 \rbrace } = A_1 \setminus (A_2 \bigcap A_3) = \lbrace 7 \rbrace $

#### Table

| I                   | {1}   | {2}   | {3}   | {1, 2} | {1, 3} | {2, 3} | {1, 2, 3} |
| :-----------------: | :---: | :---: | :---: | :----: | :----: | :----: | :-------: |
| $\lvert A_I \rvert$ | 5     | 4     | 6     | 2      | 3      | 2      | 1         |
| $\lvert B_I \rvert$ | 1     | 1     | 2     | 1      | 2      | 1      | 1         |

#### Given a B-Table, is it easy to judge it feasible

**Yes**. Just every element is Non-negative integer.

#### Given a B-Table, how to compute an A-Table

For a set P in A-Table, adding all the set Q that $P \subset Q$.

#### Given a A-Table, how to compute an B-Table

Suppose we know B-Table. Express A-Table as above. Then solve equations to get B-Table.

## Infinity sets

### Examples

$ \mathbb{I, N_0, Q, R, N} $

### Claim

#### $\mathbb{N_0, N}$ have same size

| $\mathbb{N}$   | 1     | 2     | 3     | 4     | 5     | $\dots$ |
| :------------: | :---: | :---: | :---: | :---: | :---: | :-----: |
| $\mathbb{N_0}$ | 0     | 1     | 2     | 3     | 4     | $\dots$ |

There is a bijection function that:

$$ f: \mathbb{N} \to \mathbb{N_0} \quad x \mapsto x-1 $$

#### Definition of cardinality

Let $\mathbf{A, B}$ be sets if there is a bijection $\mathbf{A} \leftrightarrow \mathbf{B}$, then we can say $\mathbf{A}$ and $\mathbf{B}$ have the same cardinality (not use size because it sounds like science) and mark $\mathbf{A} \cong \mathbf{B}$

#### More examples

- $\mathbb{N_0} \cong \mathbb{Z}$

| $\mathbb{Z}$   | $\dots$ | -4    | -3    | -2    | -1    | 0     | 1     | 2     | 3     | 4     | $\dots$ |
| :------------: | :-----: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :-----: |
| $\mathbb(N_0)$ | $\dots$ | 7     | 5     | 3     | 1     | 0     | 2     | 4     | 6     | 8     | $\dots$ |

$$ f: \mathbb{Z} \to \mathbb{N_0} \quad x \mapsto 
\begin{cases}
    2x,  & \text{if $x \le 0$ } \\
    -2x-1, & \text{if $x < 0$ }  \\
\end{cases} $$

- $ \mathbb{Z} \times \mathbb{Z} \cong \mathbb{N} $

    $$ f: \mathbb{N} \to \mathbb{Z} \times \mathbb{Z} \quad x \mapsto the \ xth \ point \ on \ the \ path $$

- $ \mathbb{N_0} \cong \mathbb{Q} $

    $$ f: \mathbb{Q} \to \mathbb{Z} \times \mathbb{Z} \\
    q = \frac{a}{b} \ where \ b \in \mathbb{N} \ a \in \mathbb{Z}, gcd(a, b) = 1$$
    $$ see \quad q \to (a, b) \\
    \begin{cases}
        injection: q_1 \neq q_2 \Rightarrow f(q_1) \neq f(q_2) \\
        surjection: \forall (a, b) \in \mathbb{Z} \times \mathbb{Z} \quad 
            \exists q \in \mathbb(Q) \quad f(q) = (a, b) ? \quad No!
    \end{cases} $$

    So how?

  - Another Definition

    If there is an injection $f: \mathbf{A} \to \mathbf{B} $, 
    then we can say $\mathbf{A} \subseteq \mathbf{B}$ and $\mathbf{A}$'s cardinality is at most $\mathbf{B}$'s cardinality.

  - Therefore, $\mathbb{Q} \le \mathbb{Z} \times \mathbb{Z}$

  - How about $\mathbb{Z} \times \mathbb{Z} \le \mathbb{Q}$?
        $$ \mathbb{Z} \times \mathbb{Z} \to 
        \mathbb{N_0} \to \mathbb{Q} \quad
        if (x \mapsto x ) $$

  - So, how about $\mathbb{N_0} \cong \mathbb{Q}$
    - $\mathbb{Q} \subseteq \mathbb{Z} \times \mathbb{Z} \subseteq \mathbb{N_0} $
    - Obviously, $\mathbb{N} \subseteq \mathbb{Q}$
    - So, $\mathbb{N} \cong \mathbb{Q} $

- $\lbrace 0, 1 \rbrace^N \cong 2^N$
  - $2^N$: power set, the set of all subsets. So every number $\in 2^N$
  - $\lbrace 0, 1 \rbrace^N$: the set of all infinite but sequence $(a_1, a_2, \dots) \quad a_i \in \lbrace 0, 1 \rbrace $

$$ 010101 \dots \mapsto \lbrace 2, 4, 6, \dots \rbrace $$
$$ 0110101 \dots \mapsto \lbrace 2, 3, 5, 7, \dots \rbrace $$
$$ 1111 \dots \mapsto \mathbb{N}$$

- $ \lbrace 0, 1 \rbrace^N \cong \mathbb{R} $
  - $f: \mathbb{R} \mapsto \lbrace 0, 1 \rbrace^N$, injection?
  - $g: [0, 1) \mapsto \lbrace 0, 1 \rbrace^N$:x write in binary as $(x)_2 = 0.a_1 a_2 \dots $ 
  - Every $x \in [0, 1) \mapsto $binary, $x \mapsto (a_1, a_2, \dots, a_n) \in \lbrace 0, 1 \rbrace^N $
    - $\frac{1}{2}: 0.1000\dots$
    - $\frac{1}{3}: 0.010101\dots$
    - $\frac{1}{4}: 0.0100\dots$
  - Injection, but **not surjection**!
