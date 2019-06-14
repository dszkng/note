# Push down Automata

## Definition

[YT - Lecture 20/65: PDAs: Pushdown Automata](https://www.youtube.com/watch?v=s5cB_xg9NGg&list=PLbtzT1TYeoMjNOGEiaRmm_vMIwUAidnQz&index=20)

### Def 7.1

A nondeterministic pushdown accept (npda) is defined by the septuple

$$ M = (Q, \Sigma, \Gamma, \delta, q_0, z, F) $$

where

$Q$: internal states of the control unit
$\Sigma$: input alphabet
$\Gamma$: stack alphabet
$\delta: Q \times (\Sigma \cup \{ \lambda \}) \times \Gamma \to ~\text{subset of}~ Q \times \Gamma^*$: transition function
$q_0 \in Q$: initial state of the control unit
$z \in \Gamma$: stack start symbol
$F \sube Q$: final states

### Def 7.2

Let $M = (Q, \Sigma, \Gamma, \delta, q_0, z, F)$ be a npda.

The language accepted by $M$ is the set 

$$ L(M) = \{ w \in \Sigma^* : (q_0, w, z) \overset{*}{\vdash}_M (p, \lambda, u) , p \in F, u \in \Gamma^* \} $$

In words, the language accepted by $M$ is the set of all strings that can put $M$ into a final state at the end of the string.

The final stack content $u$ is irrelevant to this deinition of acceptance. 

### Def 7.3

A pushdown automation $M = (Q, \Sigma, \Gamma, \delta, q_0, z, F)$ is said to be deterministic if it is an automation as defined in Definition 7.1, subject to the restrictions that, for every $q \in Q, a \in \Sigma \cup \{ \lambda \}$ and $b \in \Gamma$,

1. $\delta(q, a, b)$ contains at most one element
2. if $\delta(q, \lambda, b)$ is not empty, then $\delta(q, c, b)$ must be empty for every $c \in \Sigma$

The first of these conditions simply requires that for any given input symbol and any stack top, at most one move can be made.

The second condition is that when a $\lambda$-move is possible for some configuration, no input-consuming alternative is avaliable.

### Def 7.4

[YT - Lecture 21/65: Equivalence of CFGs and PDAs (part 1)](https://www.youtube.com/watch?v=s9GbXopLSUU&list=PLbtzT1TYeoMjNOGEiaRmm_vMIwUAidnQz&index=21)

[YT - Lecture 22/65: Equivalence of CFGs and PDAs (part 2)](https://www.youtube.com/watch?v=UUXlt2WJ59A&list=PLbtzT1TYeoMjNOGEiaRmm_vMIwUAidnQz&index=22)

A language $L$ is said to be a deterministic context-free language iff there exists a dpda $M$ s.t. $L = L(M)$