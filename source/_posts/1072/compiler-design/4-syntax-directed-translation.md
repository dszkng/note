# Syntax-Directed Translation

## Overview

### Why?

有些性質無法直接用 context-free grammar(以下簡稱CFG) 表示，舉例來說：**變數不能在宣告前被使用**，就不是 CFG 的性質。

Solutions:

* 用 context-sensitive grammars (貴)
* 用 CFG 加上 attributes 及 semantic rules(semantic actions)
  * <mark>Syntax-directed definitions (SDD)</mark>
  * <mark>Syntax-directed translation scheme (SDT)</mark>

### Attributes

為 identifier 加上一些性質：

* 常數 (numbers or strings)
* 型別 (for type checking)
* scope 的資訊 (判斷 local/global)
* 記憶體位置 (local 變數或 function 參數的 frame index)
* Intermediate program 表示法

### Syntax-Directed Translation

* Grammar symbols 用 Attribute 來知道 programming language 的 structure
* 透過 Semantic rules (與 production rules 關聯) 來決定這些 Attribute 的值
* 透過這些 Semantic rule
  * 生成 intermidiate code
  * 生成 symbol table
  * 做 type checking
  * 生成 error message
  * ...

### SDD & SDT

當我們用 Semantic rules 關聯 Production rules 時，使用以下兩種表示法：

* Syntax-Directed Definitions (SDD)
  * 提供 high-level 的規範
  * 將 production rule 與一組 semantic rules 做關聯，但沒有指定 rule 的檢查順序
* Syntax-Directed Translation Schemes (SDT)
  * SDD 的補充
  * 在 production rule 中加入 semantic actions (一段程式)
  * 指定 semantic actions 的檢查順序

#### SDD

<mark>SDD = CFG + Attributes + Semantic Rules</mark>

e.g. infix-to-postfix translation ($9-5+2 \Rightarrow 95-2+$)

| Production               | Semantic Rules                                   |
| ------------------------ | ------------------------------------------------ |
| $expr \to expr_1 + term$ | $expr.t = expr_1.t \, \| \, term.t \, \| \, '+'$ |
| $expr \to expr_1 - term$ | $expr.t = expr_1.t \, \| \, term.t \, \| \, '-'$ |
| $expr \to term$          | $expr.t = term.t$                                |
| $term \to 0$             | $term.t = \, '0'$                                |
| $term \to 1$             | $term.t = \, '1'$                                |
| ...                      | ...                                              |
| $term \to 9$             | $term.t = \, '9'$                                |

![Annotated parse tree](2019-05-05-20-23-05.png)

#### SDT

<mark>補充 SDD，指定 semantic action 在何時執行</mark>，舉例來說：$rest \to + \, term \, {print('+')} \, rest_1$：

![](2019-05-05-20-30-45.png)

semantic action 會在 term 被 derived 後執行

再以上面 SDD 的例子來說，SDT 會長成這個樣子：

> $
> expr \to expr_1 + term \, { print('+') } \\
> expr \to expr_1 - term \, { print('-') } \\
> expr \to term \\
> term \to 0 \, { print('0') } \\
> term \to 1 \, { print('1') } \\
> ... \\
> term \to 9 \, { print('9') } \\
> $

就可以指定要在什麼時候執行 semantic action 了

![](2019-05-05-20-30-09.png)

## Syntax-Directed Definitions

<mark>SDD = CFG + Attributes + Semantic Rules</mark>

* Semantic rules 與 Production rules 相關
* Attrubutes 與 Grammar symbols 相關
  * Synthesized attribute
  * Inherited attribute
  * Terminal 只能有 Synthesized attribute 不能有 Inherited attribute

### Types of Attributes

#### Synthesized Attributes

for nonterminal $A$，如果這個 Node 的 attribute 是透過<mark>他自己或是他的 children</mark>得到，則稱作 Synthesized Attributes

![](2019-05-23-16-41-30.png)

#### Inherited Attributes

for nonterminal $B$，如果這個 Node 的 attribute 是透過<mark>他的 parents 或是他的 sibling</mark>得到，則稱作 Inherited Attributes

![](2019-05-23-16-43-52.png)

### Evaluation Orders

依據 dependency 來 evaluate，但有 cycle 的 dependency 無法被 evaluate

![](2019-05-23-16-52-06.png)

| Production | Semantic Rules |
| ---------- | -------------- |
| $A \to B$  | $A.s = B.i$    |

![3x5 的 Annotated parse tree](2019-05-23-16-53-41.png)

* $T, F$ 有 Synthesized Attributes: $val$
* $digit$ 有 Synthesized Attributes: $lexval$
* $T'$ 有
  * Synthesized Attributes: $syn$
  * Inherited Attributes: $inh$

> Inherited attributes are useful when the structure of a parse tree does not "match" the abstract syntax of the source code

用 topological sort 來尋找 evaluation 的順序，若存在環，則沒有 topological sort，無法 evaluate SDD

![](2019-05-24-12-27-22.png)

* 1, 2, 3, 4, 5, 6, 7, 8, 9
* 1, 3, 5, 2, 4, 6, 7, 8, 9
* ...

但只給定 SDD，很難直接去判斷說到底有沒有環，所以還是要建 parse tree、檢查沒有環，才能 evaluatte attribute。最理想的情況是可以一邊 parse 一邊 evaluation，因此有了以下兩種固定 evaluation order 的 SDD:

* S-attributed SDD (S for synthesized)
  * 僅包含 synthesized attributes
* L-attributed SDD (L for left-to-right)
  * 包含 inherited、synthesized attributes
  * dependency-graph 只能 left-to-right

所有 S-attributed SDD 都為 L-attributed SDD

### S-Attributed Definitions

當所有 Attribute 都為 Synthesized，則這個 SDD 為 S-Attributed，<mark>可以用 bottom-up 的 evaluation</mark>，剛好在 bottom-up parsing 的時候 (postorder traversal) 可以順便進行 evaluation

### L-Attributed Definitions

Attribute 必須是:

* Synthesized
* Inherited，且滿足條件:
  * 假設有 production: $A \to X_1 X_2 ... X_n$，且有一個 inherited attribute $X_i.a$ 透過以下其中一種 rule 算出:
    * Inherited attributes 與 head $A$ 有關聯
    * Inherited 或 Synthesized attributes 與左方的 $X_1, X_2, ... , X_{i-1}$ 有關聯
    * Inherited 或 Synthesized attributes 與自己 $X_i$ 有關聯，但 $X_i$ 的 Attributes 在 dependency graph 中不能有 cycle

For example:

| Production        | Semantic Rules                   |
| ----------------- | -------------------------------- |
| $T \to F T'$      | $T'.inh = F.val$                 |
| $T' \to * F T_1'$ | $T_1'.inh = T'.inh \times F.val$ |

滿足 L-Attributed 的定義，每個 rules 都滿足 <mark>from above or from the left</mark>

| Production  | Semantic Rules                   |
| ----------- | -------------------------------- |
| $A \to B C$ | $A.s = B.b \\ B.i = f(C.c, A.s)$ |

不滿足 L-Attributed

### Semantic Rules with Side Effects

Side effect: 沒有被 attribute 定義的 statement。(e.g. print(...))

| Production  | Semantic Rules |
| ----------- | -------------- |
| $L \to E n$ | $print(E.val)$ |

Side effect 會被當作 production head 的 dummy synthesized attributed

> 沒有 side effect 的 SDD 被稱作 <mark>attribute grammar</mark>

![parse tree for float id1, id2, id3](2019-05-24-13-07-25.png)

## Applications of Syntax-Directed Translation

* Type checking
* Intermediate-code generation
* Construction of syntax trees

### Abstract Syntax Tree (AST)

Abstract Syntax Tree: 將 Parse Tree 的資訊精簡

![Parse Tree](2019-05-24-14-05-24.png)

![Abstract Syntax Tree](2019-05-24-14-05-41.png)

### Construction of Syntax Trees

#### S-Attributed SDD

Bottom-up parsing

| Production         | Semantic Rules                                 |
| ------------------ | ---------------------------------------------- |
| 1) $E \to E_1 + T$ | $E$.node = new Node(‘+’, $E_1$.node, $T$.node) |
| 2) $E \to E_1 - T$ | $E$.node = new Node(‘–’, $E_1$.node, $T$.node) |
| 3) $E \to T$       | $E$.node = $T$.node                            |
| 4) $T \to ( E )$   | $T$.node = $E$.node                            |
| 5) $T \to id$      | $T$.node = new Leaf($id$, $id$.entry)          |
| 6) $T \to num$     | $T$.node = new Leaf($num$, $num$.val)          |

![the syntax tree for a – 4 + c](2019-05-24-14-41-23.png)

$$
T \to id \xRightarrow{1} T \to num \xRightarrow{2} E \to E_1 - T \xRightarrow{3} T \to id \xRightarrow{4} E \to E_1 + T \xRightarrow{5} 
$$

1. p1 = new Leaf(id, entry-a);
2. p2 = new Leaf(num, 4);
3. p3 = new Node(‘–’, p1, p2);
4. p4 = new Leaf(id, entry-c);
5. p5 = new Node(‘+’, p3, p4);

#### L-Attributed SDD

Top-down parsing

| Production           | Semantic Rules                                                          |
| -------------------- | ----------------------------------------------------------------------- |
| 1) $E \to TE'$       | $E.node = E'.syn \\ E'.inh = T.node$                                    |
| 2) $E' \to + TE_1$   | $E1'.node$ = new Node( '$+$', $E'.inh$, $T.node$) $\\ E'.syn = E1'.syn$ |
| 3) $E' \to - TE_1$   | $E1'.node$ = new Node( '$-$', $E'.inh$, $T.node$) $\\ E'.syn = E1'.syn$ |
| 4) $E' \to \epsilon$ | $E'.syn = E'.inh$                                                       |
| 5) $T \to ( E )$     | $T.node = E.node$                                                       |
| 6) $T \to id$        | $T.node$ = new Leaf($id$, $id.entry$)                                   |
| 7) $T \to num$       | $T.node$ = new Leaf($num$, $num.entry$)                                 |

![](2019-05-24-15-06-05.png)

### Structure of a Type

Inherited attributes are useful when the structure of the **parse tree** differs from the **abstract syntax** of the input

![array references abstract syntax tree: int[2][3]](2019-05-24-15-10-07.png)

![array references](2019-05-24-15-09-00.png)

Attribute 可以從一個 subtree 帶 information 到另一個 subtree

## Syntax-Directed Translation Schemes

<mark>SDT = CFG + Program Fragments (semantic actions)</mark>，actions 以 preorder traversal 執行

![](2019-05-24-15-23-50.png)

SDT 會在 parsing 的時候 implement，將 program fragment 表示成一個 nonterminal $M \to \epsilon$

SDT 可以 implement 兩種 SDD：

* LR-parsable grammar: S-attributed SDD
* LL-parsable grammar: L-attributed SDD

## Implementing SDD's/SDT's

### S-Attributed SDD's

Simplest SDD:

1. parse the grammar bottom-up
2. SDD is S-attributed

S-attributed SDD $\to$ SDT:

1. action 放在 production 後面
2. 當 reduction 時執行那個 action

若所有 action 都在 right end，則這個 SDT 稱作 <mark>postfix SDT's</mark>

![](2019-05-24-15-36-32.png)

#### Parser-Stack

Attribute 在 production 被 parse 的時候會 push 到 stack，reduction 時會 pop

![](2019-05-24-15-47-53.png)

![](2019-05-24-15-49-40.png)

### Traversing An Annotated Parse Tree

#### Actions Inside Productions

$$ B \to X ~ \{ action \} ~ Y $$

* Bottom-up parsing: <mark>action 會在 $X$ 出現在 parsing stack 的 top 時立刻執行</mark>
* Top-down parsing: <mark>action 會在嘗試將 Y 展開、檢查 input 之前執行</mark>

#### Conflict Problem

![Infix-to-prefix translation](2019-05-24-16-05-02.png)

parser 不知道是要 reduce 回 $M_1$ 或是 reduce 回 $M_2$，導致 reduce/reduce conflict

solution: 對 action 改用 <mark>marker nonterminals</mark>

#### General SDT Implementation

1. 建一個 parse tree (忽略 actions)
2. 將 action 加到 node 中
3. 執行 preorder traversal

![3 * 5 + 4 (infix) > + * 3 5 4 (prefix)](2019-05-24-16-12-01.png)

### Eliminating Left Recursion

$$
\begin{aligned}
    A \to A \alpha | \beta \Rightarrow  &A \to \beta R \\
                                        &R \to \alpha R | \epsilon
\end{aligned}
$$

把 actions 當作 terminal symbols 做 eliminating left recursion，舉例來說:

$$
\begin{aligned}
    &E \to E + T \{ print('+'); \} \\
    &E \to T
\end{aligned}
\Rightarrow
\begin{aligned}
    &E \to TR \\
    &R \to + T \{ print('+'); \} R \\
    &R \to \epsilon
\end{aligned}
$$

#### General Case

當 action 做的不只是 print output，有包含 attribute 的計算時，需要考慮 elimination 時 action 的位置

![Parsing for XYY](2019-05-25-00-16-05.png)

![Before Elimination](2019-05-25-00-16-42.png)

![After Elimination](2019-05-25-00-17-17.png)

### L-Attributed SDD's

Rule:

* 將有關 noterminal $A$ 的 inherited attributes 的相關計算，放在 $A$ 之前
* 將有關 production head 的 synthesized attribute 的相關計算，放在 body 最後面

![](2019-05-25-20-02-21.png)

以 $S \to while ( C ) S_1$ 為例：

```
S1.next:
  C.code
  (jump to C.true or C.false)
C.true:
  S1.code
  (jump to S1.next)
C.false:
S.next:
...
```

* Inherited attributes:
  * $S.next$: 標記 S 做完之後的位置
  * $C.true$: 標記 condition C 為 true 後的位置
  * $C.false$: 標記 condition C 為 false 後的位置
* Synthesized attributes:
  * $S.code$: $S$ 的 code，及 jump 到 $S.next$ 
  * $C.code$: $C$ 的 code，及 jump 到 $C.true$ 或 $C.false$

![](2019-05-25-20-20-03.png)

#### Implementing L-Attributed SDD’s

* Traversing parse Tree:
  * [Annotated parse tree](#evaluation-orders)
    * For any **noncircular SDD**
  * 建 parse tree，加上 actions，以 preorder 執行 actions ([General SDT Implementation](#general-sdt-implementation) + [L-Attributed SDD’s](#l-attributed-sdd-s))
    * For any **L-attributed SDD**
* Translation during parsing:
  * Recursive-descent parser
  * Generate code on the fly (using recursive-descent parser)
  * 實作 SDT 在 LL-parser
  * 實作 SDT 在 LR-parser

#### During Top-Down Parsing

在 recursive-descent parser 中，對所有 nonterminal $A$ 都有對應到一個 function $A$

* function $A$ 的 arguments 為 nonterminal $A$ 的 inherited attributes
* function $A$ 的 return-value 為 nonterminal $A$ 的 synthesized attributes 的集合

以前面 [while](#l-attributed-sdd-s) 的例子：

![](2019-05-25-22-45-31.png)

在 attribute 裡面再宣告變數放 code 其實沒有必要，直接用 main attribute (synthesized): $S.code$, $C.code$ (<mark>不懂</mark>)

![](2019-05-25-22-44-37.png)

#### During Bottom-Up Parsing

