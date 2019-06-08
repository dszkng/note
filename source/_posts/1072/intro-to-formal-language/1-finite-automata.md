# Finite Automata

## Definition

### Def 2.1

A deterministic finite accepter or dfa is defined by the quintuple

$$ M = (Q, \Sigma, \delta, q_0, F) $$

where

$Q$: internal states
$\Sigma$: input alphabets
$\delta: Q \times \Sigma \to Q$: transition function
$q_0 \in Q$: initial state
$F \sube Q$: final state

### Def 2.2

The language accepted by a dfa $M = (Q, \Sigma, \delta, q_0, F)$ is the set of all strings on $\Sigma$ accepted by $M$.In formal notation,

$$ L(M) = \{ w \in \Sigma^*: \delta^*(q_0, w) \in F \} $$

### Def 2.3

A language $L$ is called regular iff there exists some dfa $M$ s.t.

$$ L = L(M) $$

## Theorem

### Thm 2.1

Let $M = (Q, \Sigma, \delta, q_0, F)$ be a dfa, and let $G_M$ be its associated transition graph.

Then for every $q_i, q_j \in Q$, and $w \in \Sigma^+, \delta^*(q_i, w) = q_j$ iff there is in $G_M$ a walk with label $w$ from $q_i$ to $q_j$

## Concept

