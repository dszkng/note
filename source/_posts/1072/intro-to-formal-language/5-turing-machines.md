# Turing machines

## Defintion

### 9.1

A Turing machine M is defined by 

$$ M = (Q, \Sigma, \Gamma, \delta,  q_0, \square, F) $$

where

$$ 
Q: \text{internal states} \\
\Sigma: \text{input alphabet} \\
\Gamma: \text{tape alphabet} \\
\delta: \text{transition function} \\
\square \in \Gamma: \text{blank} \\
q_0 \in Q: \text{initial state} \\
F \sube Q: \text{set of final states}
$$

definition:

$$
\Sigma \sube \Gamma - \{\square\} \\
\delta: Q \times \Gamma \to Q \times \Gamma \times \{ L, R \}
$$

### 9.2

Let $M = (Q, \Sigma, \Gamma, \delta,  q_0, \square, F)$ be a Turing machine.

Then a move

$$ a_1 ... \textcolor{red}{q_1 a_k} ... a_n \vdash a_1 ... \textcolor{red}{b q_2} a_{k+1} ... a_n $$

is possible iff

$$ \delta(q_1, a_k) = (q_2, b, R) $$

A move

$$ a_1 ... a_{k-1} \textcolor{red}{q_1 a_k} ... a_n \vdash a_1 ... \textcolor{red}{q_2} a_{k-1} \textcolor{red}{b} ... a_n $$

is possible iff

$$ \delta(q_1, a_k) = (q_2, b, L) $$

### 9.3

Let $M = (Q, \Sigma, \Gamma, \delta,  q_0, \square, F)$ be a Turing machine.

Then the language accepted by $M$ is:

$$ L(M) = \{ w \in \Sigma^+ : q_0 w \overset{*}{\vdash} x_1 q_f x_2 ~ \text{for some} ~ q_f \in F, x_1, x_2 \in \Gamma^* \} $$

### 9.4

A function $f$ with Domain $D$ is said to be **Turing-computable** or just **computable** if there exists some Turing machine $M = (Q, \Sigma, \Gamma, \delta,  q_0, \square, F)$ s.t.

$$ q_0 w \overset{*}{\vdash}_{M} q_f f(w), q_f \in F $$

for all $w \in D$

### 9.5

> Turing's Thesis (Hypothesis):
> 
> Any computation that can be carried out by mechanical means can be performed by some Turing machine.

An **algorithm** for a function $f: D \to R$ is a Turing machine $M$, which given as input any $d \in D$ on its tape, eventually halts with the correct answer $f(d) \in R$ on its tape.

Specifically, we can require that 

$$ q_0 d \overset{*}{\vdash}_M q_f f(d), q_f \in F $$

for all $d \in D$

### 10.1

Two automata are equivalent if they accept the same language.

Consider two classes of automata $C_1$ and $C_2$. If for every automation $M_1$ in $C_1$ there is an automation $M_2$ in $C_2$ s.t.

$$ L(M_1) = L(M_2) $$

we say that $C_2$ is at least as powerful as $C_1$.

If the converse also holds and for every $M_2$ in $C_2$ there is an $M_1$ in $C_1$ s.t. $L(M_1) = L(M_2)$, we say that $C_1$ and $C_2$ are equivalent.

### 10.2

A nondeterministic Turing machine is an automation as given by [Definition 9.1](#9-1), except $\delta$ is now a function

$$ \delta : Q \times \Gamma \to 2^{Q \times \Gamma \times \{ L, R \}} $$

A nondeterministic Turing machine is said to accept $w$ if there is any possible sequence of moves s.t

$$ q_o w \overset{*}{\vdash} x_1 q_f x_2 ~\text{with}~ q_f \in F $$

To prove nondeterministic Turing machine is no more powerful than a deterministic one, we need to provide a deterministic equivalent for the nondeterminism. ([Theorem 10.2](#10-2-v2))

### 10.3

A nondeterministic Turing machine $M$ is said to *accept* a language $L$ if, for all $w \in L$, <mark>at least one of possible configurations accept $w$</mark>. There may be branches that lead to nonaccepting configurations, while some may put the machine into an infinite loop. But these are irrelevant for acceptance.

A nondeterministic Turing machine $M$ is said to *decide* a language $L$ if, for all $w \in \Sigma^*$, <mark>each path leads to acceptance or rejection</mark>.

### 10.4

Preface: [Set Theory](#set-theory)

Let $S$ be a set of strings on some alphabet $\Sigma$. Then an **enumaration procedure** for $S$ is a Turing machine that can carry out the sequence of steps

$$ q_0 \square \overset{*}{\vdash} q_s x_1 ~\#~ s_1 \overset{*}{\vdash} q_s x_2 ~\#~ s_2 ... ~\text{with}~ x_i \in \Gamma^* - \{ \# \}, s_i \in S $$

in such a way that any $s$ in $S$ is produced in a finite number of steps.

The state $q_s$ is a state signifying membership in $S$; that is, whenever $q_s$ is entered, the string following $\#$ must be in $S$.

### 10.5

A linear bounded automation is a nondeterministic Turing machine as [Definition 10.2](#10-2), subject to the restriction that $\Sigma$ must contain two special symbols <mark>$[$ and $]$</mark>, s.t

$$ \delta(q_i, [) ~\text{can contain only elements of the form}~ (q_j, [, R) $$

and

$$ \delta(q_i, ]) ~\text{can contain only elements of the form}~ (q_j, ], L) $$

### 10.6

A string $w$ is accepted by a linear bounded automation if there is a possible sequence of moves

$$ q_0 [ w ] \overset{*}{\vdash} [ x_1 q_f x_2 ] ~\text{for some}~ q_f \in F, x_1, x_2 \in \Gamma^* $$

The language accepted by the LBA is the set of all such accepted strings.

## Theorem

### 10.1

The class of Turing machines with a stay-option is equivalent to the class of standard Turing machine.

$$ \delta: Q \times \Gamma \to Q \times \Gamma \times \{ L, R, S \} $$

Proof: Use **simulation**, check whether $\widehat{M}$ can mimic the computation of $M$

### 10.2

The class of deterministic Turing machines and the class of nondeterministic Turing machine are equivalent.

Proof: Use $\times$ to <mark>delimit the area of interest</mark>, while $+$ <mark>seperates individual instantaneous descriptions</mark>.

![Simulation by standard Turing machine](2019-06-07-15-07-22.png)

### 10.3

Preface: [Set Theory](#set-theory)

The set of all Turing machine, although infinite, is **countable**.

Proof: <mark>encode each Turing machine used $0$ and $1$</mark>, construct the following enumeration procedure.

1. Generate the next string in $\{ 0, 1 \}^+$ in proper order.
2. Check the generated string to see if it defines a Turing machine. If so, write it on the tape in the form required by [Definition 10.4](#10-4). If not, ignore the string.
3. Return to Step 1.

Since every Turing machine has a finite description, any specific machine will eventually be generated by this process.

## Variation

### with Stay option

$$ \delta: Q \times \Gamma \to Q \times \Gamma \times \{ L, R, S \} $$

### with Semi-Infinite Tape

![](2019-06-07-14-21-26.png)

$$ \widehat{\delta} (\widehat{q_i} , (a, b)) = (\widehat{q_j} , (c, b), L) ~\text{with}~ \widehat{q_i} \in Q_U, \widehat{q_j} \in Q_U $$

$$ \widehat{\delta} (\widehat{q_j} , (\#, \#)) = (\widehat{p_j} , (\#, \#), R) ~\text{with}~ \widehat{p_j} \in Q_L $$

![](2019-06-07-14-26-53.png)

### Off-line Turing machine

In such a machine, wach move is governed by the **internal state**, what is **currently read from the input file**, and what is **seen by the read-write head**.

![](2019-06-07-14-27-44.png)

![Simulation by four-track arrangement](2019-06-07-14-28-25.png)

1. input
2. marks the position at which the input is read
3. the tape of $M$
4. the position of $M$'s read-write head

### Multitape Turing machine

![](2019-06-07-14-39-21.png)

$$ \delta: Q \times \Gamma^n \to Q \times \Gamma^n \times \{ L, R \}^n $$

![Simulation by four-track arrangement](2019-06-07-14-40-46.png)

1. tape 1 of $M$
2. marks the position of $M$'s read-write head
3. tape 2 of $M$
2. marks the position of $M$'s read-write head

### Multidimensional Turing machine

![](2019-06-07-14-43-31.png)

$$ \delta: Q \times \Gamma \to Q \times \Gamma \times \{ L, R, U, D \} $$

![Simulation by two-track arrangement](2019-06-07-14-44-37.png)

1. store cell contents
2. keep associated address

e.g.

$(1, 2)$ contains $a$

$(10, -3)$ contains $b$

### Universal Turing machine

> Once $\delta$ is defined, the machine is restricted to carrying out one particular type of computation.
>
> Solution: design a <mark>reprogrammable Turing machine</mark>

![](2019-06-07-15-33-58.png)

1. encoded definition of $M$
2. tape contents of $M$
3. internal state of $M$

$M_u$ looks first at the contents of tape 2 and 3 to determine the configuration of $M$. It then consults tape 1 to see what $M$ would do in this configuration. Finally, tape 2 and 3 will be modified to reflect the result of the move.

## Concept

### Set Theory

Some sets are finite, but most of the interesting sets (and languages) are infinte. For infinite sets, we distinguish between sets that are <mark>countable</mark> or <mark>uncountable</mark>

A set is said to be countable if its elements can be put into a <mark>one-to-one correspondence with the positive integers</mark>.

> e.g. take the set of all quotients of the form $p/q$, where $p$ and $q$ are positive integers. Is this set countable?
>
> We can't use the sequence
> 
> $$ \frac{1}{1}, \frac{1}{2}, \frac{1}{3}, \frac{1}{4}... $$
>
> because then $\frac{2}{3}$ would never apper.
> 
> We use the following order to tell that the set is therfore countable.
> 
> ![](2019-06-07-16-17-56.png)

We can prove that a set is countable if we can produce a method by which its elements can be written in some sequence.

We call such a method an <mark>enumeration procedure</mark>.