# Lexical Analysis

```c
if (b == 0) a = b;
```

| lexeme | if   | \(  | b   | ==  | 0   | \)  | a   | =   | b   | ;    |
| ------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
| token  | KWif | \(  | ID  | ==  | NUM | \)  | ID  | =   | ID  | SEMI |

* Transform multi-character input stream to token stream
* Reduce length of program representation \(remove spaces\)

## Tokens, Patterns and Lexemes

* Tokens
  * Token is a sequence of characters that can be treated as a single logical entity.
  * e.g. identifier, keyword, operator
* Patterns
  * A set of strings in the **input for which the same token is produced as output**. \(**regular expression**\)
  * e.g. \[A−Za−z\_\]
* Lexemes \(詞位\)
  * A lexeme is a sequence of characters in the source program that is **matched by the pattern for a token**.
  * e.g. x, y; if, else

## Input Buffering

Sometimes lexical analyzer needs to **look ahead some symbols** to decide about the token to return.

```text
-, =, < could also beginning of a two-character operator like ->, ==, <-
```

### Buffer pairs

Use **two-buffer scheme** to handle large lookaheads safely.

![input buffer with buffer pairs](2019-04-13-01-56-05.png)

* a buffer divided into two N-character halves \(e.g. N = 1024\)
* read N chars into buffer with system read command \(v.s. using one system call per character\)
  * if fewer than N chars, than marks **eof** at the end of the source file
* Two-pointers
  * **lexemeBegin** : marks the begining of the current lexeme
  * **forward** : scans ahead until a pattern match is found

> lexeme 找到後 lexemeBegin 移至剛剛找到的 lexeme 後方的字元，forward 移至其右端

### Sentinels (哨兵)

Without sentinel, we need two tests for out-of-bound for every forward

* one for the end of the buffer
* one to determine what character is read

We can combine the buffer-end test with the test for the current character if we **extend each buffer to hold a sentinel character at the end (eof)**.

![input buffer with sentinels](2019-04-13-01-57-53.png)

### Comparison

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

## Regular Expressions

Use **regular expressions \(REs\)** to describe programming language tokens

$$
\varnothing : \{\}\\
a : ordinary\,character\\
\varepsilon : empty\,string\\
R|S : R\,or\,S\\
RS : R\,followed\,by\,S\\
R* : concatenation\,of\,R\,zero\,or\,more\,time
$$

### Language

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

### Algebraic Laws for REs

| LAW                                    | DESCRIPTION                         |
| -------------------------------------- | ----------------------------------- |
| r\|s = s\|r                            | \| is commutative                   |
| r\|\(s\|t\) = \(r\|s\)\|t              | \| is associative                   |
| r\(st\) = \(rs\)t                      | concatenation is associative        |
| r\(s\|t\) = rs\|rt; \(s\|t\)r = sr\|tr | concatenation distributes over \|   |
| εr = rε = r                            | ε is the identity for concatenation |
| r\* = \(r\|ε\)                         | ε is contained in \*                |
| r\*\* = r                              | \* is idempotent \(冪等\)           |

### Extensions of REs

| REs      | DESCRIPTION                          | ALIAS                |
| -------- | ------------------------------------ | -------------------- |
| R+       | one or more strings of R             | R\(R\*\)             |
| R?       | optional R                           | R\|ε                 |
| \[abcd\] | one of listed characters             | \(a\|b\|c\|d\)       |
| \[a-z\]  | one character from this range        | \(a\|b\|c\|d...\|z\) |
| \[^ab\]  | anything but one of the listed chars |                      |
| \[^a-z\] | one character not from this range    |                      |

### Regular Definitions

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

### Restrictions on REs

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

### Chomsky Hierarchy

![](2019-04-13-01-58-21.png)

* Unrestricted languages \(type 0\)
  * Turing machines
* Context-sensitive languages \(type 1\)
  * Linear bounded automata
* Context-free languages \(type 2\)
  * Pushdown automata
* Regular languages \(type 3\)
  * Finite automata

## Interaction between Scanner & Parser

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

## Implementation of Lexical Analysis

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

### Thompson’s construction

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

### Subset Construction

$$
NFA\xrightarrow{\text{Subset Construction}}DFA
$$

* remove ε-transitions to get a DFA

![](2019-04-13-12-47-26.png)

### Operations on NFA States

* ε-closure\(s\)
  * s 可以透過 ε-transition 到達的 NFA state 的 set
* ε-closure\(T\)
  * 在 set T 裡面的 NFA state s 可以透過 ε-transition 到達的 NFA state 的 set
* move\(T, a\)
  * 在 set T 裡面的 NFA state s 可以透過 input symbol a 到達的 NFA state 的 set

### Converting NFA to DFA

* Construct the initial state of the DFA
  * finding **ε-closure\(s0\)**
* From a state _**T**_ in the DFA, for each input character _**a**_
  * finding **ε-closure\( move\(T, a\) \)**, say U
  * make U a state in DFA if it is not there yet
    * if U contains at least one final state in NFA, then mark it as a final state in DFA
  * make **T x a -&gt; U** a transition in DFA
* Repeat the step above for all states in DFA that has not been processed yet

### Minimized DFA

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


Final-state: {E}
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

## Implementation of Lexical Analyzer Generator

![The structure of the generated analyzer](2019-04-13-13-45-10.png)

## Ambiguity Resolution

### Longest match

* match 最長的 pattern
* 在 accept 前需要 lookahead

![input: abbbbccd&#xFF0C;&#x82E5; abbbb &#x5F8C;&#x662F; ccd &#x5247; accept state 6&#xFF0C;&#x5426;&#x5247; accept state 3](2019-04-13-14-00-14.png)

### First match

* conflict 時選最先列出的 rule

### Lookahead

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

## Lexical Analyzer Generator: Lex

![How Does Lex Work?](2019-04-13-14-31-53.png)

當 RE 能有多個 match 情形時，考慮：

* Longest matching token
* Token specification order