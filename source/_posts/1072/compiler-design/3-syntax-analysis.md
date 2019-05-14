# Syntax Analysis

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

## Syntax Error-Recovery Strategies

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

## Context-Free Grammars

$$
G = (T, N, S, P)\\
T: terminals = token\,or\,\varepsilon\\
N: nonterminals = syntactic\,variables\\
S: start\,symbol = special\,nonterminal\\
P: productions\,of\,the\,form = head \to body
$$

More on CFGs, see: [正規語言概論](https://kaiiiz.github.io/NCTU-Coursenote/1072/intro-to-formal-language/0-couseinfo/)

## Grammar Transformations

### 消除 ambiguity

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

### 消除 left recursion

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

### Left factoring

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

## Parser

![Various kinds: LL(k), LR(k), SLR, LALR](2019-04-13-15-43-32.png)

### Top-down parser (LL parser)

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

### Bottom-up parser (LR parser)

* 從 leaves 開始長出 root
* Left-to-right scan
* Rightmost derivation

## Top-Down Parsing

從 root 以 preorder 建立 parse tree

* 找 input string 的 leftmost derivation
* 遞迴尋找 nonterminal 可能的展開，錯誤時需要 backtrack

![](2019-04-14-17-07-27.png)

Predictive Parsing: 藉由 lookahead (k) 個 symbols 直接選擇正確的 production

![](2019-04-14-17-14-04.png)

lookahead (k) 個 symbols 的 LL parser 稱為 LL(k)

### FIRST and FOLLOW Sets

* Sets of terminals
* 用途：透過下一個 input symbol 選擇要 apply 那個 production

![c in FIRST(A) and a in FOLLOW(a)](2019-04-14-17-48-46.png)

### FIRST Sets

$$
First(\alpha): Set \, of \, terminals \, that \, begin \, strings \,derived \, from \, \alpha \\
\alpha \in terminals \, or \, nonterminals
$$

> if $\alpha \overset{*}{\Rightarrow} c \gamma \,$ ,then $c \in FIRST(\alpha)$  
> if $\alpha \overset{*}{\Rightarrow} \varepsilon \,$ ,then $\varepsilon \in FIRST(\alpha)$

### FOLLOW Sets

$$Follow(A): Set \, of \, terminals \, \alpha \, that \, can \, appear \, immediately \, to \, the \, right \, of \, A \\
A \in nonterminals
$$

> if $S \overset{*}{\Rightarrow} \alpha A a \beta \,$ ,then $a \in FOLLOW(A)$  
> if $A$ can be the rightmost symbol in some sentential form, then $(endmarker) $\in FOLLOW(A)$

### Why FIRST Set?

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

### Why FOLLOW Set?

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

### LL(1) Parsing

Recursive-Descent Parsing

looking **one** symbols ahead in the input (i.e., current input symbol)

* LL(1) 不能處理 left-recursive grammars (因為需要 lookahead 多位才知道要不要做 production) (Parsing Tree 會無限往下長)
* 

#### Grammar 滿足條件

對 $A \to \alpha | \beta$，LL(1) 需要滿足的條件：

1. $\alpha, \beta$ 不能 derive 出相同的 first terminal
2. $\alpha, \beta$ 只有一個可以 derive 出 $\varepsilon$
3. 若 $\beta \overset{*}{\Rightarrow} \epsilon$ 則 $\alpha$ derive 出的 first terminal 不能包含在 FOLLOW(A) 中 (因為 有兩條路：選 $\beta$ + FOLLOW(A) 或 FIRST($\alpha$))

#### Parse 規則

假設 $a$ 是 current input symbol 或 $\$$(endmarker)

若 $A \to \alpha$ 被選擇，需要滿足：

* $a \in FIRST(\alpha)$ 或
* $\varepsilon \in FIRST(A)$ 或
* $a = \$$ 且 $\$ \in FOLLOW(A)$

#### Predictive Parsing Table

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

## Bottom-Up Parsing

從 leave 長回 root

* LR parsing: Left-to-right scan, Rightmost derivation (反向)
* 過程像是將 input string "reduce" 回 start symbol

![](2019-04-14-23-01-01.png)

* Pros
  * 比 LL parser 強大，幾乎可以描述所有程式語言
  * 不需要 backtracing
* Cons
  * 建一個 LR parser 比較貴

### Handle Pruning

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

### Shift-Reduce Parsing

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

#### Issue

不確定什麼時候要 reduce 什麼時候要 shift

* 有時不可 reduce
* 有時可以 reduce 也可以 shift
* 有時有多種 reduce 的方式

#### Solution

* 做一張認得所有 viable prefiexes 的 DFA
* 用 stack 持續觀察看到的 input 決定下一個 state

### LR(0) Automaton

#### Definition

* augmented grammar $G'$
  * 將 $G$ 的 start symbol 改為 $S'$ 且 $S' \to S$
* two function
  * CLOSURE
  * GOTO
* 當 reduce 到 $S' \to S$ 時可以 Accept
* state 接受認得的 viable prefixes

#### LR(0) item

$A \to XYZ$ 包含 4 個 items:

1. $A \to \cdot XYZ$：希望在 input 上看到 XYZ
2. $A \to X \cdot YZ$：剛看到 X 希望在 input 上看到 YZ
3. $A \to XY \cdot Z$：剛看到 XY 希望在 input 上看到 Z
4. $A \to XYZ \cdot$：reduce XYZ to A

$A \to \varepsilon$ 僅包含 $A \to \cdot$

item 又區分為兩類：

1. Kernel items: initial item ($S' \to \cdot S$) 及 dot 不在最左邊的 items
2. Nonkernel items: dot 在最左邊的 items，除了initial item ($S' \to \cdot S$)

#### CLOSURE(I)

$I$: grammar $G$ 的 items set  
$CLOSURE(I)$: 由 $I$ 構成的 items set

意義：在同一個 $CLOSURE(I)$ 的 items 都屬於同一個 parsing state，用其紀錄我們在過去看到什麼還有在未來期望看到什麼

#### GOTO(I, X)

$I$: grammar $G$ 的 items set  
$X$: $G$ 的 grammar symbol  
$GOTO(I, X)$: if $[A \to \alpha \cdot X \beta] \in I$,then $CLOSURE( \{ [A \to \alpha X \cdot \beta] \} ) \subseteq GOTO(I, X)$

意義：當看到 I 期望看到的 X，將 dot 往右移到 X 右方，並對其做 CLOSURE (像是 state transition)

#### LR(0) Collection

> $C = \{ CLOSURE(\{ S' \to \cdot S \}) \}$  
> repeat  
> 　　for(each $I$ in $C$ and each grammar symbol $X$)  
> 　　　　if($GOTO(I, X)$ is not empty and not in $C$)  
> 　　　　　　add $GOTO(I, X)$ to $C$  
> until no more set of LR(0) items can be added to C

### LR(0) Parsing

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

### SLR Parsing

Simple LR Parsing

**LR(0) + FOLLOW Set = SLR(1)** (簡稱SLR)

* 在 State $Ii$ 遇到 terminal $a$ 的決策：
  * 若 $[A \to \alpha \cdot a \beta] \in Ii$，shift
      * 從 $Ii$ 走到 $CLOSURE(\{ [A \to \alpha a \cdot \beta] \})$
  * 若 $[A \to \beta \cdot] \in Ii$ 且 $a \in FOLLOW(A)$，做 reduce by $A \to \beta$
      * 從 $Ii$ 走到 $GOTO(I, A)$,$I$ 是在 remove $\beta$ 後 stack 的 top

#### Basic

SLR parsing 之所以可以判斷要做 shift 或 reduce，是基於 LR(0) automata 識別 viable prefixes 的能力：

$S \overset{*}{\underset{rm}{\Rightarrow}} \alpha A w \underset{rm}{\Rightarrow} \alpha \beta_1 \beta_2 w$

* $[A \to \beta_1 \cdot \beta_2]$ is **valid** for $\alpha \beta_1$
  * 若 $\beta_2 \neq \varepsilon$ 代表還沒 shift handle 進 stack，所以做 shift
  * 若 $\beta_2 = \varepsilon$ 代表 $\beta_1$ 為 handle，所以做 reduce

#### Model

維護兩個表 **ACTION TABLE** 及 **GOTO TABLE**

![](2019-04-15-03-09-26.png)

#### SLR Parsing Table

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

#### Example

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

#### Conflicts

* shift/reduce conflict: state 不知道該做 shift 還是 reduce
* reduce/reduce conflict: state 不知道該用哪個 reduce
* 若 grammar G 的 SLR parsing table 有 conflict，則 G 非 SLR Grammar