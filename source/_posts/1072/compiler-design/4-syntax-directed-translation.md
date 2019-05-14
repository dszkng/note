# Syntax-Directed Translation

## Overview

## Why?

有些性質無法直接用 context-free grammar(以下簡稱CFG) 表示，舉例來說：**變數不能在宣告前被使用**，就不是 CFG 的性質。

Solutions:

* 用 context-sensitive grammars （貴）
* 用 CFG 加上 attributes 及 semantic rules(semantic actions)
  * <mark>Syntax-directed definitions (SDD)</mark>
  * <mark>Syntax-directed translation scheme (SDT)</mark>

## Attributes

為 identifier 加上一些性質：

* 常數（numbers or strings)
* 型別（for type checking)
* scope 的資訊（判斷 local/global）
* 記憶體位置（local 變數或 function 參數的 frame index）
* Intermediate program 表示法

## Syntax-Directed Translation

* Grammar symbols 用 Attribute 來知道 programming language 的 structure
* 透過 Semantic rules (與 production rules 關聯) 來決定這些 Attribute 的值
* 透過這些 Semantic rule
  * 生成 intermidiate code
  * 生成 symbol table
  * 做 type checking
  * 生成 error message
  * ...

## SDD & SDT

當我們用 Semantic rules 關聯 Production rules 時，使用以下兩種表示法：

* Syntax-Directed Definitions (SDD)
  * 提供 high-level 的規範
  * 將 production rule 與一組 semantic rules 做關聯，但沒有指定 rule 的檢查順序
* Syntax-Directed Translation Schemes (SDT)
  * SDD 的補充
  * 在 production rule 中加入 semantic actions (一段程式)
  * 指定 semantic actions 的檢查順序

## SDD

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

## SDT

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