# An Overview of Computational Complexity

assumptions:

1. The model for our study will be a Turing machine. The exact type of Turing machine to be used will be discussed below.
2. The size of the problem will be denoted by n.
3. In analyzing an algorithm, we are less interested in its performance on a specific case than in its general behavior. We are particularly concerned with how the algorithm behaves when the problem size increases.

## Definition

### Def 14.1

We say that a Turing machine $M$ decides a language $L$ in time $T(n)$ if every $w$ in $L$ with $|w| = n$ is decided in $T(n)$ moves.

If $M$ is nondeterministic, this implies that for every $w \in L$, there is at least one sequence of moves of length less than or equal to $T(|w|)$ that leads to acceptance, and that the Turing machine halts on all inputs in time $T(|w|)$.

### Def 14.2

A language $L$ is said to be a member of the class $DTIME(T(n))$ if there exists a deterministic multitape Turing machine that decides $L$ in time $O(T(n))$. 

A language $L$ is said to be a member of the class $NTIME(T(n))$ if there exists a nondeterministic multitape Turing machine that decides $L$ in time $O(T(n))$. 

> $DTIME(T(n)) \sube NTIME(T(n))$
> 
> and $T_1(n) = O(T_2(n))$
> 
> implies $DTIME(T_1(n)) \sube DTIME(T_2(n))$

## Theorem

### Thm 14.1

Suppose that a two-tape machine can carry out a computation in $n$ steps.

Then this computation can be simulated by a standard Turing machine in $O(n^2)$ steps.

### Thm 14.2

Suppose that a nondeterministic Turing machine $M$ can carry out a com­putation in $n$ steps.

Then a standard Turing machine can carry out the same computation in $O(k^{an})$ steps, where $k$ and $a$ are independent of $n$.

### Thm 14.3

For every integer $k \geq 1$,

$$ DTIME(n^k) \subset DTIME(n^{k+1}) $$

## Concept

### The Comolexity Classes P and NP

$$ P = \underset{i \geq 1}{\bigcup} DTIME (n^i) $$

$$ NP = \underset{i \geq 1}{\bigcup} NTIME (n^i) $$

Obviously, $P \sube NP$

## NP Problems

### SAT Problem

### Hamiltonian Path Problem

### Clique Problem