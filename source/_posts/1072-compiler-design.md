---
title: 編譯器設計
---

# 編譯器設計

課程名稱：編譯器設計 (Complier Design)

授課教師：游逸平

開課單位：資科工碩

教科書：Compilers: Principles, Techniques, and Tools (2nd edition) **（aka. 龍書）**

課程網站：[https://people.cs.nctu.edu.tw/~ypyou/courses/Compiler-grad-s19/](https://people.cs.nctu.edu.tw/~ypyou/courses/Compiler-grad-s19/)

## Introduction

### Overview

![High-level View of a Compiler](2019-04-13-01-54-12.png)

![Two-pass Compiler](2019-04-13-01-54-45.png)

* Use an **intermediate representation** \(IR\)
  * Reuse components

![](2019-04-13-01-55-09.png)

### Steps for Generating an Executable Program

a.c **---COMPILER---&gt;** a.s ---ASSEMBLER---&gt; a.o ---LINKER---&gt; a.out/a.exe ---LOADER---&gt; execution

### The Structure of a Compiler

![](2019-04-13-01-55-32.png)

* Symbol table
  * A data structure containing a record for each **identifier**, with fields for the **attributes**
  * identifier
    * lexical analysis
  * attributes
    * syntax analysis
    * semantic analysis
* Error Handler

### Lexical Analysis

divides program text into **"words"** or **"tokens"**

* A stream of tokens
* Whitespace and comments are removed

```c
if (b == 0) a = b;
```

| lexeme | if   | \(  | b   | ==  | 0   | \)  | a   | =   | b   | ;    |
| ------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
| token  | KWif | \(  | ID  | ==  | NUM | \)  | ID  | =   | ID  | SEMI |

### Syntax Analysis (Parsing)

understand sentence structures

* Abstract syntax tree built from parsing rules

```text
a := b + c * 60
```

```text
  :=
 /  \
a    +
    / \
   b   *
      / \
     c   60
```

### Semantic Analysis

try to understand "meaning" \(but hard to machine\)

* Compilers perform limited analysis to **catch inconsistencies**
* Types checked; references to symbols resolved

### Intermediate Code Generation

Constructs **intermediate representations \(IRs\)** for code optimization

```text
  :=
 /  \
a    +
    / \
   b   *
      / \
     c   60
```

```text
temp1 := c * 60
temp2 := b + temp1
a := temp2
```

### Code Optimization

Automatically modify programs so that they

* Run faster
* Use less memory
* Consume lower power
* Conserve some resource

### Code Generation

Produces **assembly code**

```text
temp1 := id3 * 60.0
id1 := id2 + temp1
```

```text
LDF R2, id3
MULF R2, R2, #60.0
LDF R1, id2
ADDF R1, R1, R2
STF id1, R1
```

---

## Lexical Analysis

```c
if (b == 0) a = b;
```

| lexeme | if   | \(  | b   | ==  | 0   | \)  | a   | =   | b   | ;    |
| ------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
| token  | KWif | \(  | ID  | ==  | NUM | \)  | ID  | =   | ID  | SEMI |

* Transform multi-character input stream to token stream
* Reduce length of program representation \(remove spaces\)

### Tokens, Patterns and Lexemes

* Tokens
  * Token is a sequence of characters that can be treated as a single logical entity.
  * e.g. identifier, keyword, operator
* Patterns
  * A set of strings in the **input for which the same token is produced as output**. \(**regular expression**\)
  * e.g. \[A−Za−z\_\]
* Lexemes \(詞位\)
  * A lexeme is a sequence of characters in the source program that is **matched by the pattern for a token**.
  * e.g. x, y; if, else

### Input Buffering

Sometimes lexical analyzer needs to **look ahead some symbols** to decide about the token to return.

```text
-, =, < could also beginning of a two-character operator like ->, ==, <-
```

#### Buffer pairs

Use **two-buffer scheme** to handle large lookaheads safely.

![input buffer with buffer pairs](2019-04-13-01-56-05.png)

* a buffer divided into two N-character halves \(e.g. N = 1024\)
* read N chars into buffer with system read command \(v.s. using one system call per character\)
  * if fewer than N chars, than marks **eof** at the end of the source file
* Two-pointers
  * **lexemeBegin** : marks the begining of the current lexeme
  * **forward** : scans ahead until a pattern match is found

> lexeme 找到後 lexemeBegin 移至剛剛找到的 lexeme 後方的字元，forward 移至其右端

#### Sentinels (哨兵)

Without sentinel, we need two tests for out-of-bound for every forward

* one for the end of the buffer
* one to determine what character is read

We can combine the buffer-end test with the test for the current character if we **extend each buffer to hold a sentinel character at the end (eof)**.

![input buffer with sentinels](2019-04-13-01-57-53.png)

#### Comparison

Input buffering with buffer pairs:

```c
// 1. Check the end of the buffer
if (forward at the end of first buffer) {
    reload second buffer
    forward += 1
}
else if (forward at the end of second buffer) {
    reload first buffer
    forward = begin of first buffer
}
else {
    forward += 1
}

// 2. read forward character
if (*foward == ...) {
    ...
}
```

Input buffering with sentinels:

```c
forward += 1
if (*forward == eof) {
    if (forward at the end of first buffer) {
        reload second buffer
        forward += 1
    }
    else if (forward at the end of second buffer){
        reload first buffer
        forward = begin of first buffer
    }
    else {
        terminate lexical analysis
    }
}
else if (*forward == ...) {
    ...
}
```

Visualized detail: [https://www.slideshare.net/dattatraygandhmal/input-buffering](https://www.slideshare.net/dattatraygandhmal/input-buffering)

### Regular Expressions

Use **regular expressions \(REs\)** to describe programming language tokens

$$
\varnothing : \{\}\\
a : ordinary\,character\\
\varepsilon : empty\,string\\
R|S : R\,or\,S\\
RS : R\,followed\,by\,S\\
R* : concatenation\,of\,R\,zero\,or\,more\,time
$$

#### Language

A regular expression R describes a **set of strings** of characters denoted **L\(R\)**, also called a **regular set**

| Regular Expression, R | Strings in L\(R\)     |
| --------------------- | --------------------- |
| a                     | "a"                   |
| ab                    | "ab"                  |
| a\|b                  | "a", "b"              |
| \(ab\)\*              | "", "ab", "abab", ... |
| \(a\|ε\)b             | "ab", "b"             |

Suppose **r** and **s** are RE denoting **L\(r\)** and **L\(s\)**

* \(r\) is a RE denoting L\(r\)
* \(r\)\|\(s\) is a RE denoting L\(r\)∪L\(s\)
* \(r\)\(s\) is a RE denoting L\(r\)L\(s\)
* \(r\) _is a RE denoting \(L\(r\)\)_

#### Algebraic Laws for REs

| LAW                                    | DESCRIPTION                         |
| -------------------------------------- | ----------------------------------- |
| r\|s = s\|r                            | \| is commutative                   |
| r\|\(s\|t\) = \(r\|s\)\|t              | \| is associative                   |
| r\(st\) = \(rs\)t                      | concatenation is associative        |
| r\(s\|t\) = rs\|rt; \(s\|t\)r = sr\|tr | concatenation distributes over \|   |
| εr = rε = r                            | ε is the identity for concatenation |
| r\* = \(r\|ε\)                         | ε is contained in \*                |
| r\*\* = r                              | \* is idempotent \(冪等\)           |

#### Extensions of REs

| REs      | DESCRIPTION                          | ALIAS                |
| -------- | ------------------------------------ | -------------------- |
| R+       | one or more strings of R             | R\(R\*\)             |
| R?       | optional R                           | R\|ε                 |
| \[abcd\] | one of listed characters             | \(a\|b\|c\|d\)       |
| \[a-z\]  | one character from this range        | \(a\|b\|c\|d...\|z\) |
| \[^ab\]  | anything but one of the listed chars |                      |
| \[^a-z\] | one character not from this range    |                      |

#### Regular Definitions

give **names** to regular expressions

```text
name -> regular expression
d1 -> r1
d2 -> r2
...
dn -> rn
```

```text
e.g. digit = [0-9], posint = digit+
```

#### Restrictions on REs

Regular expressions are not capable of describing most complete languages.

They describe languages composed of sets of strings of the form :

$$
S\to\alpha B\\
where\\
\alpha \in basic\,symbol\\
B\,is\,a\,regular\,definition\\
S\,is\,a\,regular\,definition\,and\,is\,not\,part\,of\,B\\
$$

They **cannot** describe:

* Balanced nesting constructs
  * e.g.  if ... then ... else ...
* Repetition of the same string
  * e.g.  {wcw \| w is a string of a's and b's}
* Constructs where the number of repetitions is fixed by the value of a part of the string
  * e.g.  nHa1a2 ...an

Anything that needs to "**memorize**" "**non-constant**" amount of information **happened in the past** cannot be recognized by regular expressions

#### Chomsky Hierarchy

![](2019-04-13-01-58-21.png)

* Unrestricted languages \(type 0\)
  * Turing machines
* Context-sensitive languages \(type 1\)
  * Linear bounded automata
* Context-free languages \(type 2\)
  * Pushdown automata
* Regular languages \(type 3\)
  * Finite automata

### Interaction between Scanner & Parser

![](2019-04-13-01-58-51.png)

**Attributes of Tokens**

| Lexemes  | Token Name | Attribute Value                                       |
| -------- | ---------- | ----------------------------------------------------- |
| if       | if         | -                                                     |
| then     | then       | -                                                     |
| else     | else       | -                                                     |
| id       | id         | Pointer to table entry                                |
| number   | number     | Pointer to table entry \(or the value of the number\) |
| &lt;     | relop      | LT                                                    |
| &lt;=    | relop      | LE                                                    |
| =        | relop      | EQ                                                    |
| &lt;&gt; | relop      | NE                                                    |
| &gt;     | relop      | GT                                                    |
| &gt;=    | relop      | GE                                                    |

### Implementation of Lexical Analysis

Use **Transition Diagram**

![](2019-04-13-01-59-17.png)

**Recognizes reserved words**

problem: keyword maybe recognize as an identifier

Two solutions:

* Install the reserved words in the symbol table initially
  * installID\(\) : place identifier in the symbol table if it is **not already there**, and return a pointer to the symbol-table entry for the lexeme found
  * getToken\(\) : return token name \(either **id** or one of the **keyword** tokens that was initially installed in the table\)

![](2019-04-13-01-59-35.png)

* Create separate transition diagrams for each keyword

![](2019-04-13-01-59-51.png)

### Finite Automata

Recognizers which simply say "**yes**" or "**no**" about each possible input string

![](2019-04-13-02-00-18.png)

**Definition**

$$
M = (S, \Sigma, S_0, F, T)\\
S : all\,state\,in\,the\,FA\\
\Sigma : all\,simbols\,accepted\,by\,the\,language\\
S_0 : Start\,state\\
F : Accepting\,states\\
T : All\,transitions
$$

**Epsilon Moves**

Machine can move from state A to state B without reading input

![](2019-04-13-02-00-32.png)

**Nondeterministic Finite Automata (NFA)**

![](2019-04-13-02-00-46.png)

* Can have multiple transitions for one input in a given state
* Can have ε-moves

**Deterministic Finite Automata (DFA)**

![](2019-04-13-02-01-06.png)

* One transition per input per state
* No ε-moves

**NFA vs DFA**

* DFA
  * Action on each input is fully determined
  * Easier to implement
* NFA
  * May have choices at each step
  * Accepts string if there is **ANY path to an accepting state**
  * More complex in implementation
    * May need to backtrack
    * May end up exploring all the paths in the NFA

**Transition Table**

![](2019-04-13-02-01-40.png)

| State | a      | b   | ε   |
| ----- | ------ | --- | --- |
| 0     | {0, 1} | {0} | ∅   |
| 1     | ∅      | {2} | ∅   |
| 2     | ∅      | {3} | ∅   |
| 3     | ∅      | ∅   | ∅   |

* Columns are input symbols
* Rows are current states
* Entries are resulting states
* Pros and Cons
  * Pro
    * easily find the transitions
  * Con
    * take lots of space

### RE to Automata

![](2019-04-13-12-09-18.png)

![High-level sketch](2019-04-13-12-06-45.png)

#### Thompson’s construction

$$
Regular\,Expression\xrightarrow{\text{Thompson's Construction}}NFA
$$

![For expression ε](2019-04-13-12-19-26.png)

![For any subexpression a in Σ](2019-04-13-12-19-41.png)

Suppose N\(s\) and N\(t\) are NFA's for RE s and t

* r = s\|t

![Alternation](2019-04-13-12-22-41.png)

* r = st

![Concatenation](2019-04-13-12-25-48.png)

* r = s\*

![Kleene closure](2019-04-13-12-26-22.png)

* Precedence of Operators
  1. Kleene closure \(\*\), ?, +
  2. concatenation
  3. alternation
* All operators are **left associative**

#### Subset Construction

$$
NFA\xrightarrow{\text{Subset Construction}}DFA
$$

* remove ε-transitions to get a DFA

![](2019-04-13-12-47-26.png)

#### Operations on NFA States

* ε-closure\(s\)
  * s 可以透過 ε-transition 到達的 NFA state 的 set
* ε-closure\(T\)
  * 在 set T 裡面的 NFA state s 可以透過 ε-transition 到達的 NFA state 的 set
* move\(T, a\)
  * 在 set T 裡面的 NFA state s 可以透過 input symbol a 到達的 NFA state 的 set

#### Converting NFA to DFA

* Construct the initial state of the DFA
  * finding **ε-closure\(s0\)**
* From a state _**T**_ in the DFA, for each input character _**a**_
  * finding **ε-closure\( move\(T, a\) \)**, say U
  * make U a state in DFA if it is not there yet
    * if U contains at least one final state in NFA, then mark it as a final state in DFA
  * make **T x a -&gt; U** a transition in DFA
* Repeat the step above for all states in DFA that has not been processed yet

#### Minimized DFA

同個 language 可以用不同的 DFA 來表示，所以我們可以找到最少 state 的 DFA 來優化從 NFA 轉過來的 DFA。

**key principle : merge 有相同行為的 state**

* final states 與 non-final states 不可 merge
* 對所有 x 從 state s 到 t，若所有 acceptance decision 相同，則 s 與 t 有相同的行為

流程：

* 初始化
  * 將 final state 與 non-final state 區格開
* 區分不同的群組 G
  * 對所有 input symbol a，在 G 中的兩個 state s 與 t 可以通過 a 轉到相同的群組，則 s 與 t 屬於同一個群組
  * 除此之外，將 s 與 t 放至不同群組中
* 重複直到群組不再變化

例子：

![before minimize](2019-04-13-13-33-06.png)

* Final-state: {E}

  Non-final-state: {A, B, C, D}

* Try to split {A,B,C,D}
  * for input a
    * A,B,C,D all go to {A,B,C,D} on a
  * for input b
    * A,B,C go to {A,B,C,D} on b
    * D goes to {E} on b
  * {A,B,C,D} → {A,B,C},{D},{E}
* Try to split {A,B,C}
  * for input a
    * A,B,C all go to {A,B,C} on a
  * for input b
    * A,C go to {A,B,C} on b
    * B goes to {D} on b
  * {A,B,C},{D},{E} → {A,C},{B},{D},{E}
* A,C go to the same states on each input

![after minimize](2019-04-13-13-31-33.png)

### Implementation of Lexical Analyzer Generator

![The structure of the generated analyzer](2019-04-13-13-45-10.png)

#### Ambiguity Resolution

##### Longest match

* match 最長的 pattern
* 在 accept 前需要 lookahead

![input: abbbbccd&#xFF0C;&#x82E5; abbbb &#x5F8C;&#x662F; ccd &#x5247; accept state 6&#xFF0C;&#x5426;&#x5247; accept state 3](2019-04-13-14-00-14.png)

##### First match

* conflict 時選最先列出的 rule

##### Lookahead

有些情形 longest match 和 first match 無法處理

> e.g. In fortran:
>
> treat IF a keyword: IF \(a = b\) THEN ...  
> treat IF a identifier: IF \(I, J\) = 3 ...

```text
IF / \( .* \) letter {return IF}
Otherwise, return identifier
```

/ 後方可以 match additional pattern 但是不 match lexeme，所以下次的 match 是從 \( 開始，而不是 letter 後方

#### Lexical Analyzer Generator: Lex

![How Does Lex Work?](2019-04-13-14-31-53.png)

當 RE 能有多個 match 情形時，考慮：

* Longest matching token
* Token specification order

---

## Syntax Analysis

**Goal – 判斷 input token stream 是否滿足程式語言的 syntax**

```text
if (b == 0) a = b;
```

Lexical Analyzer or Scanner:

| lexeme | if   | \(  | b   | ==  | 0   | \)  | a   | =   | b   | ;    |
| ------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
| token  | KWif | \(  | ID  | ==  | NUM | \)  | ID  | =   | ID  | SEMI |

Syntax Analyzer or Parser:

```text
     if
    /  \
  ==     =
 / \    / \
b   0  a   b
```

速成 Syntax Analysis: https://www.youtube.com/watch?v=uPnpkWwO9hE&list=PLW1OMpQZxu7xMh7nuDQYQ2mDcqY2hzBWk

### Syntax Error-Recovery Strategies

* Panic-mode Recovery (simple)
  * 忽略 input symbols 直到找到 synchronizing tokens (通常是 delimiter)
* Phrase-level Recovery
  * 局部修正錯誤
  * e.g. comma → semicolon, insert/delete semicolon
* Error-productions
  * 將可能的 error 情形寫成 productions 加到 grammar 中
  * 產生對應的 error diagnostics
* Global-correction (theoretical)
  * 選擇需要 cost 最小的地方做修正

### Context-Free Grammars

$$
G = (T, N, S, P)\\
T: terminals = token\,or\,\varepsilon\\
N: nonterminals = syntactic\,variables\\
S: start\,symbol = special\,nonterminal\\
P: productions\,of\,the\,form = head \to body
$$

More on CFGs, see: [正規語言概論](https://kaiiiz.github.io/NCTU-Coursenote/sophomore/intro-to-formal-language/)

### Grammar Transformations

#### 消除 ambiguity

> 有多種 parse trees 的 grammar 稱為 ambiguous grammar

**solution : 重寫 grammar**

**例子1: 對每個 precedence level 多加一個 nonterminal**

```
E → E + E | E * E | –E | (E) | id
轉為
E → E + E | T
T → T * T | F
F → –E | (E) | id
```

problem:

無法辨識有結合律的語句 (e.g. id + id + id)

solution:

E → E + E ，因為 + 的兩側都是 E ，需要將 E 轉成另一個 nonterminal 來解決結合律的問題: E → E + T | T

```text
E → E + E | T
T → T * T | F
F → –E | (E) | id
轉為
E → E + T | T
T → T * T | F
F → –E | (E) | id
```

**例子2: if 的巢狀結構** (重寫 grammar 有時可能需要對某些情形做特判)

```
stmt → if-stmt | while-stmt | ...
if-stmt → if expr then stmt else stmt | if expr then stmt
```

* input: if (a) then if (b) then x = c else x = d
* ambiguity:
  * if (a) then {if (b) then x = c} else x = d
  * if (a) then {if (b) then x = c else x = d}


solution:

```text
if-stmt → unmatched-stmt | matched-stmt
matched-stmt → if expr then matched-stmt else matched-stmt | others
unmatched-stmt → if expr then matched-stmt else unmatched-stmt
unmatched-stmt → if expr then if-stmt
```

一旦進入 matched-stmt 就無法再回到 unmatched-stmt

#### 消除 left recursion

> A grammar is left recursive if it has a nonterminal $A$ such that there's a derivation $A \overset{+}{\Rightarrow} A \alpha$ a for some string $\alpha$

Top-down parsing 無法處理 left-recursive grammars

solution:

由 $A \rightarrow A \alpha \, | \, \beta$ 轉為 $A \rightarrow \beta A'$、$A' \rightarrow \alpha A' | \varepsilon$

> In general:
> 
> $$
> A \to A \alpha_1| A \alpha_2 | ...|A \alpha_m | \beta_1 | \beta_2 | ... | \beta_n\\
> \downarrow\\
> A \to \beta_1 A' | \beta_2 A' | ... | \beta_n A'\\
> A' \to \alpha_1 A' | \alpha_2 A' | ... | \alpha_m A' | \varepsilon
> $$

problem:

不能消除需要多次轉換才出現的 left-recursive

> e.g. 
> 
> $$
> S \to Aa \, | \, b\\
> A \to Ac \, | \, Sd \, | \, \varepsilon\\
> \downarrow\\
> S \Rightarrow Aa \Rightarrow Sda
> $$

solution:

假設沒有 $A \overset{+}{\Rightarrow} A$ 及 $A \to \varepsilon$

> 將所有 nonterminal 排序： $A_1, A_2, ..., A_n$  
> for (each i from 1 to n) {  
> 　　for (each j from 1 to i - 1) { // j < i  
> 　　　將 $A_i \to A_j \gamma$、$A_j \to \delta_1 | \delta_2 | ... | \delta_k$ 取代成 $A_i \to \delta_1 \gamma |  \delta_2 \gamma | ... | \delta_k \gamma$   
> 　　}  
> 　　對所有 $A_i$ 的 productions 消除 immediate left recursion  
> }

#### Left factoring

用來產生適合 top-down parsing 的 grammar (predictive)

problem:

> $A \to a \beta_1 \, | \, a \beta_2$ 在碰到 $a$ 後不能馬上確定應該要 derive 成 $a \beta_1$ 或是 $a \beta_2$

solution:

> $$
> A \to a \beta_1 \, | \, a \beta_2\\
> \downarrow\\
> A \to a A'\\
> A' \to \beta_1 \, | \beta_2
> $$

### Parser

![Various kinds: LL(k), LR(k), SLR, LALR](2019-04-13-15-43-32.png)

#### Top-down parser (LL parser)

* 從 root 開始長出 leaves
* Left-to-right scan
* Leftmost derivation

> e.g. 
> 
> input: id + id
> 
> $E \to T + T\\T \to E | -E | id$
> 
> solution:
> 
> $E \underset{lm}{\Rightarrow} T + T \underset{lm}{\Rightarrow} id + T \underset{lm}{\Rightarrow} id + id$

#### Bottom-up parser (LR parser)

* 從 leaves 開始長出 root
* Left-to-right scan
* Rightmost derivation

### Top-Down Parsing

從 root 以 preorder 建立 parse tree

* 找 input string 的 leftmost derivation
* 遞迴尋找 nonterminal 可能的展開，錯誤時需要 backtrack

![](2019-04-14-17-07-27.png)

Predictive Parsing: 藉由 lookahead (k) 個 symbols 直接選擇正確的 production

![](2019-04-14-17-14-04.png)

lookahead (k) 個 symbols 的 LL parser 稱為 LL(k)

#### FIRST and FOLLOW Sets

* Sets of terminals
* 用途：透過下一個 input symbol 選擇要 apply 那個 production

![c in FIRST(A) and a in FOLLOW(a)](2019-04-14-17-48-46.png)

#### FIRST Sets

$$
First(\alpha): Set \, of \, terminals \, that \, begin \, strings \,derived \, from \, \alpha \\
\alpha \in terminals \, or \, nonterminals
$$

> if $\alpha \overset{*}{\Rightarrow} c \gamma \,$ ,then $c \in FIRST(\alpha)$  
> if $\alpha \overset{*}{\Rightarrow} \varepsilon \,$ ,then $\varepsilon \in FIRST(\alpha)$

#### FOLLOW Sets

$$Follow(A): Set \, of \, terminals \, \alpha \, that \, can \, appear \, immediately \, to \, the \, right \, of \, A \\
A \in nonterminals
$$

> if $S \overset{*}{\Rightarrow} \alpha A a \beta \,$ ,then $a \in FOLLOW(A)$  
> if $A$ can be the rightmost symbol in some sentential form, then $(endmarker) $\in FOLLOW(A)$

#### Why FIRST Set?

$$
A \to \alpha_1 \\
A \to \alpha_2 \\
... \\
A \to \alpha_k
$$

$$
current \, lookahead \, symbol \, is \, \alpha
$$

* 若只有一個 $\alpha \in FIRST(\alpha_i)$ 則選 $A \to \alpha_i$
* 若滿足多個 $FIRST(\alpha_i)$，則這個 grammar 不是 LL(1)

#### Why FOLLOW Set?

$$
A \to \alpha_1 \\
A \to \alpha_2 \\
... \\
A \to \alpha_k
$$

$$
current \, lookahead \, symbol \, is \, \alpha
$$

* 若只有一個 i 滿足 $\alpha \in FIRST(\alpha_i)$ 則選 $A \to \alpha_i$
* 若滿足多個 $FIRST(\alpha_i)$，則這個 grammar 不是 LL(1)
* 若沒有 i 滿足 $\alpha \in FIRST(\alpha_i)$，這個 grammar 仍為 LL(1)
  * 因為若有任何 $\alpha_i \overset{*}{\Rightarrow} \varepsilon$ 且 $a \in FOLLOW(A)$ 則可以用 $A \to \alpha_i$ 將 A 換成 $\varepsilon$

#### LL(1) Parsing

Recursive-Descent Parsing

looking **one** symbols ahead in the input (i.e., current input symbol)

* LL(1) 不能處理 left-recursive grammars (因為需要 lookahead 多位才知道要不要做 production) (Parsing Tree 會無限往下長)
* 

##### Grammar 滿足條件

對 $A \to \alpha | \beta$，LL(1) 需要滿足的條件：

1. $\alpha, \beta$ 不能 derive 出相同的 first terminal
2. $\alpha, \beta$ 只有一個可以 derive 出 $\varepsilon$
3. 若 $\beta \overset{*}{\Rightarrow} \epsilon$ 則 $\alpha$ derive 出的 first terminal 不能包含在 FOLLOW(A) 中 (因為 有兩條路：選 $\beta$ + FOLLOW(A) 或 FIRST($\alpha$))

##### Parse 規則

假設 $a$ 是 current input symbol 或 $\$$(endmarker)

若 $A \to \alpha$ 被選擇，需要滿足：

* $a \in FIRST(\alpha)$ 或
* $\varepsilon \in FIRST(A)$ 或
* $a = \$$ 且 $\$ \in FOLLOW(A)$

##### Predictive Parsing Table

M[A, a]: 對 nonterminal A 及 input symbol a 在 parsing table 存在的 production

演算法：

> 對所有 $A \to \alpha$：
> 
> 1. 對所有 $a \in FIRST(\alpha)$, 將 $A \to \alpha$ 加到 M[A, a]
> 2. 若 $\varepsilon \in FIRST(\alpha)$ 則：
>   1. 對所有 $b \in FOLLOW(A)$， 將 $A \to \alpha$ 加到 M[A, b]
>   2. 若 $\varepsilon \in FOLLOW(A)$， 將 $A \to \alpha$ 加到 M[A, $]
> 3. 將空的位置設為 error

若 parsing table 中一個 entry 有兩個 production (conflict)，表示這個 grammar 不是 LL(1) 能處理的，可能要考慮 LL(2)、LL(3)...

#### Nonrecursive Predictive Parsing

![parsing table + stack](2019-04-14-19-56-50.png)

不用 recursive 改用 stack 實做

#### Error Recovery: Panic mode

Review: [Syntax Error-Recovery Strategies](#syntax-error-recovery-strategies)

若 M[A, a] 為空且 $a \in FOLLOW(A)$ 則把 M[A, a] 設為 **synch**

策略：

另 A = top of stack、a = current input

* if A == Nonterminal
  * M[A, a] = {empty}: skip a (沒有 production 可做，忽略 input)
  * M[A, a] = {synch}: pop A (因為 $a \in FOLLOW(A)$，所以先把 A pop 掉，下一 run a 就可以接在 A 後面了)
* if A == terminal
  * A != a: pop A

### Bottom-Up Parsing

從 leave 長回 root

* LR parsing: Left-to-right scan, Rightmost derivation (反向)
* 過程像是將 input string "reduce" 回 start symbol

![](2019-04-14-23-01-01.png)

* Pros
  * 比 LL parser 強大，幾乎可以描述所有程式語言
  * 不需要 backtracing
* Cons
  * 建一個 LR parser 比較貴

#### Handle Pruning

$$
S \to aABe\\
A \to Abc | b\\
B \to d
$$

$$
input: abbcde
$$

| Right Sentential Form | Handle | Viable Prefix | Reducing Production |
| --------------------- | ------ | ------------- | ------------------- |
| a**b**bcde            | b      | ab            | A → b               |
| a**Abc**de            | Abc    | aAbc          | A → Abc             |
| aA**d**e              | d      | aAd           | B → d               |
| **aABe**              | aABe   | aABe          | S → aABe            |

![](2019-04-14-23-48-24.png)

Formally, 若 $S \overset{*}{\underset{rm}{\Rightarrow}} \alpha A w \underset{rm}{\Rightarrow} \alpha \beta w$，則 $\alpha \beta w$ 的 **handle**: 在 $\alpha$ 後方的位置做 production $A \to \beta$

**Viable Prefix**: handle 尾巴前的前綴

尋找 handle 並 reduce 它的過程稱作：**Handle Pruning**

#### Shift-Reduce Parsing

![Performs handle pruning with a stack](2019-04-15-00-09-58.png)

* 用 stack 持續觀察看到的 input
* 在 stack 中的 elements 一定是 viable prefix
* 4 actions
  * Shift: 將下一個 token 丟到 stack
  * Reduce: Handle 的最右端在 stack 的 top，尋找 handle 的左端並 reduce 之
  * Accept: parse 成功
  * Error: call error reporting/recovery

$$
E \to E + T | T\\
T \to T * F | F\\
F \to (E) | id
$$

$$
input: id*id
$$

| Stack   | Input    | Action                  |
| ------- | -------- | ----------------------- |
| $       | id * id$ | shift                   |
| $id     | * id$    | reduce by $F \to id$    |
| $F      | * id$    | reduce by $T \to F$     |
| $T      | * id$    | shift                   |
| $T *    | id$      | shift                   |
| $T * id | $        | reduce by $F \to id$    |
| $T * F  | $        | reduce by $T \to T * F$ |
| $T      | $        | reduce by $E \to T$     |
| $E      | $        | accept                  |

Compare: [LR(0) Parsing](#lr-0-parsing)

##### Issue

不確定什麼時候要 reduce 什麼時候要 shift

* 有時不可 reduce
* 有時可以 reduce 也可以 shift
* 有時有多種 reduce 的方式

##### Solution

* 做一張認得所有 viable prefiexes 的 DFA
* 用 stack 持續觀察看到的 input 決定下一個 state

#### LR(0) Automaton

##### Definition

* augmented grammar $G'$
  * 將 $G$ 的 start symbol 改為 $S'$ 且 $S' \to S$
* two function
  * CLOSURE
  * GOTO
* 當 reduce 到 $S' \to S$ 時可以 Accept
* state 接受認得的 viable prefixes

##### LR(0) item

$A \to XYZ$ 包含 4 個 items:

1. $A \to \cdot XYZ$：希望在 input 上看到 XYZ
2. $A \to X \cdot YZ$：剛看到 X 希望在 input 上看到 YZ
3. $A \to XY \cdot Z$：剛看到 XY 希望在 input 上看到 Z
4. $A \to XYZ \cdot$：reduce XYZ to A

$A \to \varepsilon$ 僅包含 $A \to \cdot$

item 又區分為兩類：

1. Kernel items: initial item ($S' \to \cdot S$) 及 dot 不在最左邊的 items
2. Nonkernel items: dot 在最左邊的 items，除了initial item ($S' \to \cdot S$)

##### CLOSURE(I)

$I$: grammar $G$ 的 items set  
$CLOSURE(I)$: 由 $I$ 構成的 items set

意義：在同一個 $CLOSURE(I)$ 的 items 都屬於同一個 parsing state，用其紀錄我們在過去看到什麼還有在未來期望看到什麼

##### GOTO(I, X)

$I$: grammar $G$ 的 items set  
$X$: $G$ 的 grammar symbol  
$GOTO(I, X)$: if $[A \to \alpha \cdot X \beta] \in I$,then $CLOSURE( \{ [A \to \alpha X \cdot \beta] \} ) \subseteq GOTO(I, X)$

意義：當看到 I 期望看到的 X，將 dot 往右移到 X 右方，並對其做 CLOSURE (像是 state transition)

##### LR(0) Collection

> $C = \{ CLOSURE(\{ S' \to \cdot S \}) \}$  
> repeat  
> 　　for(each $I$ in $C$ and each grammar symbol $X$)  
> 　　　　if($GOTO(I, X)$ is not empty and not in $C$)  
> 　　　　　　add $GOTO(I, X)$ to $C$  
> until no more set of LR(0) items can be added to C

#### LR(0) Parsing

LR parsing without lookahead symbols

* 在 State $Ii$ 的決策：
  * 若 $[A \to \alpha \cdot a \beta] \in Ii$，當看到 terminal $a$ 時做 shift
      * 從 $Ii$ 走到 $CLOSURE(\{ [A \to \alpha a \cdot \beta] \})$
  * 若 $[A \to \beta \cdot] \in Ii$，做 reduce by $A \to \beta$
      * 從 $Ii$ 走到 $GOTO(I, A)$,$I$ 是在 remove $\beta$ 後 stack 的 top

See example in [Shift-Reduce Parsing](#shift-reduce-parsing)

$$
E' \to E\\
E \to E + T | T\\
T \to T * F | F\\
F \to (E) | id
$$

$$
input: id*id
$$

![Parsing table](2019-04-15-02-15-34.png)

| Stack    | Symbol  | Input    | Action              | Transition              |
| -------- | ------- | -------- | ------------------- | ----------------------- |
| 0        | $       | id * id$ | shift to 5          | I0 → I5                 |
| 0 5      | $id     | * id$    | reduce by F → id    | I5 → GOTO(I0, F) -- I3  |
| 0 3      | $F      | * id$    | reduce by T → F     | I3 → GOTO(I0, T) -- I2  |
| 0 2      | $T      | * id$    | shift to 7          | I2 → I7                 |
| 0 2 7    | $T *    | id$      | shift to 5          | I7 → I5                 |
| 0 2 7 5  | $T * id | $        | reduce by F → id    | I5 → GOTO(I7, F) -- I10 |
| 0 2 7 10 | $T * F  | $        | reduce by T → T * F | I10 → GOTO(I0, T) -- I2 |
| 0 2      | $T      | $        | reduce by E → T     | I2 → GOTO(0, E) -- I1   |
| 0 1      | $E      | $        | accept              |

#### SLR Parsing

Simple LR Parsing

**LR(0) + FOLLOW Set = SLR(1)** (簡稱SLR)

* 在 State $Ii$ 遇到 terminal $a$ 的決策：
  * 若 $[A \to \alpha \cdot a \beta] \in Ii$，shift
      * 從 $Ii$ 走到 $CLOSURE(\{ [A \to \alpha a \cdot \beta] \})$
  * 若 $[A \to \beta \cdot] \in Ii$ 且 $a \in FOLLOW(A)$，做 reduce by $A \to \beta$
      * 從 $Ii$ 走到 $GOTO(I, A)$,$I$ 是在 remove $\beta$ 後 stack 的 top

##### Basic

SLR parsing 之所以可以判斷要做 shift 或 reduce，是基於 LR(0) automata 識別 viable prefixes 的能力：

$S \overset{*}{\underset{rm}{\Rightarrow}} \alpha A w \underset{rm}{\Rightarrow} \alpha \beta_1 \beta_2 w$

* $[A \to \beta_1 \cdot \beta_2]$ is **valid** for $\alpha \beta_1$
  * 若 $\beta_2 \neq \varepsilon$ 代表還沒 shift handle 進 stack，所以做 shift
  * 若 $\beta_2 = \varepsilon$ 代表 $\beta_1$ 為 handle，所以做 reduce

##### Model

維護兩個表 **ACTION TABLE** 及 **GOTO TABLE**

![](2019-04-15-03-09-26.png)

##### SLR Parsing Table

* 建構 LR(0) collection $C = \{ I_0, I_1, ... , I_n \}$ for $G'$  
* $ACTION[i, a] , a \in terminal$ 
  * 若 $[A \to \alpha \cdot a \beta] \in I_i$ 且 $GOTO(I_i, a) = I_j$
      * $ACTION[i, a]$ = "shift $j$"
  * 若 $[A \to \alpha \cdot] \in I_i$ 且 $A \neq S$
      * $ACTION[i, a]$ = "reduce by $A \to \alpha$" for all $a \in FOLLOW(A)$
  * 若 $[S' \to S \cdot] \in I_i$
      * $ACTION[i, a]$ = "accept"
* $GOTO[i, A] , A \in nonterminal$
  * 若 $GOTO[I_i, A] = I_j$
      * $GOTO[i, A] = j$
* 空的皆為 "error"
* Initial state $I_0$ 包含 $[S' \to S]$

若 step 2 出現 conflict，則這個 grammar 不是 SLR

##### Example

![](2019-04-15-02-15-34.png)

![](2019-04-15-03-54-49.png)

| Stack    | Symbol  | Input         | Action |
| -------- | ------- | ------------- | ------ |
| 0        | $       | id * id + id$ | s5     |
| 0        | $id     | * id + id$    | r6     |
| 0 3      | $F      | * id + id$    | r4     |
| 0 2      | $T      | * id + id$    | s7     |
| 0 2 7    | $T *    | id + id$      | s5     |
| 0 2 7 5  | $T * id | + id$         | r6     |
| 0 2 7 10 | $T * F  | + id$         | r3     |
| 0 2      | $T      | + id$         | r2     |
| 0 1      | $E      | + id$         | s6     |
| 0 1 6    | $E +    | id$           | s5     |
| 0 1 6 5  | $E + id | $             | r6     |
| 0 1 6 3  | $E + F  | $             | r4     |
| 0 1 6 9  | $E + T  | $             | r1     |
| 0 1      | $E      | $             | acc    |

##### Conflicts

* shift/reduce conflict: state 不知道該做 shift 還是 reduce
* reduce/reduce conflict: state 不知道該用哪個 reduce
* 若 grammar G 的 SLR parsing table 有 conflict，則 G 非 SLR Grammar

## IR

### p14

呼叫P的結果當作呼叫F的parameter

### p16

y 的 R-value assign 給 x

a = b 等號右手邊在乎的是 R-value 左手邊在乎的是 L-value

### p17

i * 8: 算 offset

### p18

如果 IR 沒有分 level，且一開始 IR 就很接近 machine code，一些 high-level 的 Info 就會丟失(ex. loop...) > Optimize 會有困難

### p19

triple: 不需要 dst，存在 OP 的位置

### p20

左上還未優化的 IR

左下圍它的 syntax tree

triple 直接把值存在 OP 不用 Tmp

### p21

quadruple 可能比較適合做 optimization

e.g. instruction reorder (較慢的先做) 任意調

triple 對調指令時連存放的地方都要改，麻煩 >> 使用 indirect triple

### p22

維護一個 array of pointer，指向原本的 triple

### p23

每一個變數或temp只會被assign一次

左邊: 

read after write: flow dependence 妳寫完的東西我要讀
write after read: anti dependenae，妳讀完後我要寫 (因為用了相同的名字才有dependence)
write after write (?output dependence)

dependence 減少比較容易 optimization

fi function 接收兩個值得其中一個值

### p25

型別檢查

data 再 memory 怎麼擺放，存相對的位置

### p27

prgram 的 layout 由 compiler 決定 (右邊)

static: 再 runtime 之前可以決定且部會變化的變數 (全域或static的區域變數)

function push 到 stack，區域變數存在其中，位置隨 runtime 改變

heap: dynamic data (e.g. malloc) 可以增減，有memory manager 負責維護那些空間是空的

### p28

2 個 ( 3 個 int 的 array ) 的 array

### p30

s x t: 兩個 type expression 兩歌argument分別是s和t的型別

動態型別仍要做型別檢查 (引入 type variables)

### p32

structurally equivalent: 結構一樣就一樣

e.g. int A[10], int B[10]: array of 10 int

name equivalent: 名字一樣才一樣 (?)

e.g. typedef int dollars: int, dollar 不同 type

### p33

看到資訊保留到 symbol table 中，不需要產生 IR 來說明這個 declaration

### p34

這個名字再 storage 的 Layout

型別也代表了寬度

### p35

symbol table 也存了擺放位置的資訊

> java bytecode 不需要考慮 memory 擺放的問題 (?)

### p36

擺放的位置可能需要被 4, 8 整除 >> alignment

e.g. 起始位置不能被 4 整除，可能需要 2 個 op 才能 access >> 加上 padding

### p37

存位置的目的? 為了將來 load data 到 reg 或 store data

### p38

計算 type expression 及 type 寬度 >> 透過撰寫 SDT

* 當看到 Int reduce 回 B 時，知道 type:int, 寬度:4
* 遞迴到右下的 C
* reduce 回 C: array(3, int), w:12
* reduce 回 C: array(2, array(3, int)), w:24
* reduce 回 T

### p39

看到新的 compound stmt，push 一張新的 symbol table 到 stack

### p40

new 出一張新的 symbol table，進入裏頭的 symbol table 時先處理完外面的 symbol table，進入後歸零 offset

### p41

> Summary: 當看到變數宣告就把變數家道 symbol table，存 type, offset

Heap 裏頭的擺放不是由 compiler 管理，e.g. malloc 就交給 malloc 的 library 管理

### p43

1. 先執行 E 的 IR 存到 E 的 address

### p44

1. -E reduce 回 E > t1
2. E + E reduce 回 E > t2
3. id = E reduce 回 S > a = t2

### p48

算陣列的 Address 位置

### p49

L.array: 是放在symbol table的哪個entry
L.type: subarray的型別
L.addr: temp, 計算offset是多少

### p52

使用者寫的與預期的是否一致

一個健全的 type system 可以在 compile time 就檢查完 type error

c 不是 strongly type language

```c
union flexType {
    int intElement;
    float floatElement;
};

union flexType element;
float x;

element.ineElement = 27;
x = element.floatElement; \\ element 存 int 但以 float 看待它 (讀出來不是 27)
```

### p53

type synthesis: static-typed
type inference: dynamic-typed

### p55

型別轉換

### p57

### p59

function overloaded

### p60

adhoc polymorphic

$\forall \alpha list(\alpha) \to integer$

### p61

推導type expression

用 type inference rule (for 動態型別語言)

### p62

3) boolean condition + 2個type variables

### p63

unify: 要把 type variable 取代成實際的 variable

$E1(E2)$ 

### p64

$S(t)$ t 的 instance，把 t 實體化

sub: type expression 做實體化
某一個 sub 是兩個 Type expression 的 unifier

### p65

### p66

給 2 個 node 判斷能不能 unify

### p67

編號: unique number

檢查1.9可不可以被unify

### p71

control flow statement

### p77

fall through 簡化 goto

### p83

可不可以一邊parsing的過程產生右邊的IR?

### p84

backpatching

jump 與 target 做 match

先產生goto，taget先不要填，等address知道後再填

### p85

不用 Inherit attribute

truelist，一堆goto，當b是true會執行的goto
falselist，...，當b是false會執行的goto

