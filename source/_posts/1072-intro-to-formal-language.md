---
title: 正規語言概論
---

# 正規語言概論

## Regular Grammars

### 定義

#### Grammar

$$
G = (V, T, S, P)\\
V: Variables \, (nonterminals)\\
T: Terminal \, symbols\\
S: Start \, symbols\\
P: Productions
$$

##### Regular Grammars

Linear Grammar: A grammar which **at most one variable can occur on the right side of any production**, without restriction on the position of this variable.

> All Regular Grammars are Linear Grammars but all Linear Grammars are not Regular Grammars.

A grammar $G = (V, T, S, P)$ is said to be

* right-linear: if all productions are of the form $A \to xB, A \to x$ where $A, B \in V, x \in T^*$
* left-linear: if all productions are of the form $A \to Bx, A \to x$ where $A, B \in V, x \in T^*$


#### DFA

Deterministic Finite Accepters

$$
M = (Q, \Sigma, \delta, q_0, F)\\
Q: Internal \, states\\
\Sigma: Input \, alphabets\\
\delta: Transition \, functions\\
q_0 \in Q: Initial \, state\\
F \subseteq Q: Set \, of \, finite \, set
$$

Transition function: $Q \times \Sigma \to Q$、$\delta(q_i, w) = q_j$

Extended transition function: $Q \times \Sigma^* \to Q$、$\delta^*(q_i, w) = q_j$

#### NFA

Nondeterministic Finite Accepters

$$
M = (Q, \Sigma, \delta, q_0, F)\\
Q: Internal \, states\\
\Sigma: Input \, alphabets\\
\delta: Transition \, functions\\
q_0 \in Q: Initial \, state\\
F \subseteq Q: Set \, of \, finite \, set
$$

Transition function: $Q \times ( \Sigma \cup { \lambda } ) \to 2^Q$

Extended transition function: $\delta^*(q_i, w) = Q_j$

$Q_j$: The set of all possible states the automation may be in

#### RE

Terminal conditions:

1. $\varnothing$: RE denoting empty set
2. $\lambda$: RE denoting $\{ \lambda \}$
3. $\forall a \in \Sigma$: RE denoting $\{a\}$

If $r_1, r_2$ are RE, then

1. $L(r_1 + r_2) = L(r_1) \cup L(r_2)$
2. $L(r_1 \cdot r_2) = L(r_1)L(r_2)$
3. $L((r_1)) = L(r_1)$ 
4. $L({r_1}^*) = (L(r_1))^*$

#### Language

##### L(DFA)

The language accepted by a DFA is the set of all strings on $\Sigma$ accepted by $M$

$$
L( M ) = \{ w \in \Sigma^*: \delta^*(q_0, w) \in F \}
$$

##### L(NFA)

The language consists of all string $w$ for which there is a walk labeled $w$ from initial vertex to some final vertex.

$$
L( M ) = \{ w \in \Sigma^*: \delta^*(q_0, w) \cap F \neq \varnothing \}
$$

##### Regular Languages

The language $L$ is called "regular" if and only if there exists some DFA($M$) such that $L = L(M)$

遇到問題：Show language $L$ is regular：找 $L$ 的 DFA

完備性：

A language $L$ is regular if and only if there exists a regular grammar $G$ such that $L = L(G)$



### 定理

#### Finite Automata

##### DFA = NFA

Convert NFA to DFA: [Powerset construction](https://en.wikipedia.org/wiki/Powerset_construction)


#### RE

##### RE to RL

Let $r$ be a RE. Then there exists some NFA that accepts $L(r)$. Consequently, $L(r)$ is a RL.


#### Language

##### L\(r\)

Let $L$ be a regular language. Then there exists a regular expression $r$ such that $L = L(r)$

##### RL to RE

Convert NFA to GTG.

過程待補

generalized transition graphs (GTG): a transition graph whose edges are labeled with RE.

##### RL to RG

If $L$ is a regular language on the alphabet $\Sigma$. Then there exists a right-linear grammar $G = (V, T, S, P)$ such that $L = L(G)$

#### Grammar

##### RG to RL

Let $G = (V, T, S, P)$ be a right-linear grammar. Then $L(G)$ is a regular language.



### 性質

#### Grammar

* 若 $L(G_1) = L(G_2)$ 則 $G_1$ 與 $G_2$ 相等

#### RE

##### Precedence

star-closure > concatenation > union

#### Closure properties of RL

If $L_1, L_2$ are regular languages, then so are $L_1 \cup L_2, L_1 \cap L_2, L_1L_2, \overline{L_1}, {L_1}^*$



### 基本問題

#### w 是否存在在 L 中

Given a standard representation of any regular language $L$ on $\Sigma$ and any $w \in \Sigma^*$, there exists an algorithm for determining whether or not $w$ is in $L$.

> $L \to DFA \to Test \, w$

standard representation of regular language: finite automation / regular expression / regular grammar

#### L 是否為空

There exists an algorithm for determining whether a RL is empty / finite / infinite

> empty: dfa has path from S > F
> 
> infinite: S > F has cycle

#### L 是否相等

Given two regular languages $L_1, L_2$, there exists an algorithm to determine whether or not $L_1 = L_2$

> $L_3 = ( L_1 \cap \overline{L_2} ) \cup ( \overline{L_1} \cap L_2 )$
>
> By closure properties, $L_3$ is regular, we can find a dfa $M$ that accepts $L_3$
>
> $if \, L_3 = \varnothing \to L_1 = L_2$


### 判斷 Nonregular Languages

#### Pigeonhole principle

待補

#### Pumping Lemma

Using pigeonhole principle in another form.

1. Pick $m$
2. Given $m$, we pick a string $w \in L, |w| \geq m$
3. Decomposition $w$ to $xyz$, $|xy| \leq m, |y| \geq 1$
4. Pick $i$, get $w_i$. If $w_i \notin L$, $L$ is not regular

---

## Context-Free Grammars

### 定義

A grammar $G = (V, T, S, P)$ is said to be context-free if all production in $P$ has the form $A \to x$, where $A \in V, x \in (V \cup T)^*$(A 可以被任意 x 取代，不考慮上下文)

A language $L$ is said to be context-free if and only if there is a context-free grammar $G$ such that $L = L(G)$

### Derivation

* leftmost: leftmost variable in the sentential form is replaced
* rightmost: rightmost variable in the sentential form is replaced

---

## 10.1

什麼問題電腦不能解決

一個問題就是一個 Language，所有問題就是 Language 的數目 ($2^{\Sigma^*}$)

$$
L \subset \Sigma^* \\
x \in \Sigma^* \\
\Sigma^* = \{ \lambda, 0, 1, 00, 01... \}
$$

兩類 automata equivalent $L(M_1) = L(M_2)$

### p3

用 simulation 證明

### p5

Stay: 不動

$\hat{M}$ 標準 turing machine

### p7

多訂一些 transition function，訂完就 equivalent 了

