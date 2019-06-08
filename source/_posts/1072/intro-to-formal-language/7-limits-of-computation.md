# Limits of Algorithmic Computation

## Definition

### 12.1

Let $w_M$ be a string that describes a Turing machine $M = (Q, \Sigma, \Gamma, \delta,  q_0, \square, F)$, and let $w$ be a string in $M$'s alphabet.

We will assume that $w_M$ and $w$ are encoded as a string of $0$'s and $1$'s, as suggested in Section 10.4. A solution of the halting problem is a Turing machine $H$, which for any $w_M$ and $w$ performs the computation 

$$ q_0 w_M w \overset{*}{\vdash} x_1 q_y x_2 $$

if $M$ applied to $w$ halts, and 

$$ q_0 w_M w \overset{*}{\vdash} y_1 q_n y_2 $$

if $M$ applied to $w$ does not halt. Here $q_y$ and $q_n$ are both final states of $H$.

## Theorem

### 12.1

There does not exist any Turing machine $H$ that behaves as required by [Definition 12.1](#12.1). <mark>The halting problem is therefore undecidable</mark>.

### 12.2

If the halting problem were decidable, then every recursively enumerable language would be recursive. Consequently, the halting problem is unde­cidable.

### 12.3

Let $G$ be an unrestricted grammar. Then the problem of determining whether or not

$$ L(G) = \empty $$

is undecidable.

### 12.4

Let $M$ be any Turing machine. Then the question of whether or not $L(M)$ is finite is undecidable.

### 12.5

Let $G = (V, T, S, P)$ be any unrestricted grammar, with $w$ any string in $T^+$.

Let $(A, B)$ be the correspondence pair constructed from $G$ and $w$ be the process exhibited in Figure 12.8.

Then the pair $(A, B)$ permits an MPC solution if and only if $w \in L(G)$ 

![](2019-06-08-17-39-21.png)

### 12.6

The modified Post correspondence problem is undecidable. 

### 12.7

The Post correspondence problem is undecidable.

## Concept

### Computability and Decidability

We say that a problem is decidable if there exists a Turing machine that gives the correct answer for every statement in the domain of the problem.

### Halting problem

Given the description of a Turing machine $M$ and an input $w$, does $M$, when started in the initial configuration $q_0w$, perform a computation that eventually halts?

### Reducing Undecidable Problem

We say that a problem $A$ is reduced to a problem $B$ if the decidability of $A$ follows from the decidability of $B$.

Then, if we know that $A$ is undecidable, we can conclude that $B$ is also undecidable.

### Post Correspondence Problem

Given two sequence of $n$ strings on some alphabet $\Sigma$, say

$$ A = w_1, w_2, ... w_n $$

and

$$ B = v_1, v_2, ... v_n $$

we say that there exists a Post correspondence solution (PC-solution) for pair $(A, B)$ if there is a nonempty sequence of integers $i, j ... k$, s.t.

$$ w_i w_j ... w_k = v_i v_j ... v_k $$

The PC-problem is to devise an algorithm that will tell us, for any $(A, B)$, whether or not there exists a PC-solution.

We break it into two parts.

In the first part, we introduce the <mark>modified Post correspondence problem</mark>. We say that the pair $(A, B)$ has a modified Post correspondence solution (MPC solution) if there exists a sequence of integers $i, j, ... ,k$, s.t.

$$ w_1 w_i w_j ... w_k = v_1 v_i v_j ... v_k $$

An MPC solution must start with $w_1$ on the left side and with $v_1$ on the right side.

> Note that if there exists an MPC solution, then there is also a PC solution, but the converse is not true.

