# Introduction

## Overview

![High-level View of a Compiler](2019-04-13-01-54-12.png)

![Two-pass Compiler](2019-04-13-01-54-45.png)

* Use an **intermediate representation** \(IR\)
  * Reuse components

![](2019-04-13-01-55-09.png)

## Steps for Generating an Executable Program

a.c **---COMPILER---&gt;** a.s ---ASSEMBLER---&gt; a.o ---LINKER---&gt; a.out/a.exe ---LOADER---&gt; execution

## The Structure of a Compiler

![](2019-04-13-01-55-32.png)

* Symbol table
  * A data structure containing a record for each **identifier**, with fields for the **attributes**
  * identifier
    * lexical analysis
  * attributes
    * syntax analysis
    * semantic analysis
* Error Handler

## Lexical Analysis

divides program text into **"words"** or **"tokens"**

* A stream of tokens
* Whitespace and comments are removed

```c
if (b == 0) a = b;
```

| lexeme | if   | \(  | b   | ==  | 0   | \)  | a   | =   | b   | ;    |
| ------ | ---- | --- | --- | --- | --- | --- | --- | --- | --- | ---- |
| token  | KWif | \(  | ID  | ==  | NUM | \)  | ID  | =   | ID  | SEMI |

## Syntax Analysis (Parsing)

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

## Semantic Analysis

try to understand "meaning" \(but hard to machine\)

* Compilers perform limited analysis to **catch inconsistencies**
* Types checked; references to symbols resolved

## Intermediate Code Generation

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

## Code Optimization

Automatically modify programs so that they

* Run faster
* Use less memory
* Consume lower power
* Conserve some resource

## Code Generation

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