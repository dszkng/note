# IR

## p14

呼叫P的結果當作呼叫F的parameter

## p16

y 的 R-value assign 給 x

a = b 等號右手邊在乎的是 R-value 左手邊在乎的是 L-value

## p17

i * 8: 算 offset

## p18

如果 IR 沒有分 level，且一開始 IR 就很接近 machine code，一些 high-level 的 Info 就會丟失(ex. loop...) > Optimize 會有困難

## p19

triple: 不需要 dst，存在 OP 的位置

## p20

左上還未優化的 IR

左下圍它的 syntax tree

triple 直接把值存在 OP 不用 Tmp

## p21

quadruple 可能比較適合做 optimization

e.g. instruction reorder (較慢的先做) 任意調

triple 對調指令時連存放的地方都要改，麻煩 >> 使用 indirect triple

## p22

維護一個 array of pointer，指向原本的 triple

## p23

每一個變數或temp只會被assign一次

左邊: 

read after write: flow dependence 妳寫完的東西我要讀
write after read: anti dependenae，妳讀完後我要寫 (因為用了相同的名字才有dependence)
write after write (?output dependence)

dependence 減少比較容易 optimization

fi function 接收兩個值得其中一個值

## p25

型別檢查

data 再 memory 怎麼擺放，存相對的位置

## p27

prgram 的 layout 由 compiler 決定 (右邊)

static: 再 runtime 之前可以決定且部會變化的變數 (全域或static的區域變數)

function push 到 stack，區域變數存在其中，位置隨 runtime 改變

heap: dynamic data (e.g. malloc) 可以增減，有memory manager 負責維護那些空間是空的

## p28

2 個 ( 3 個 int 的 array ) 的 array

## p30

s x t: 兩個 type expression 兩歌argument分別是s和t的型別

動態型別仍要做型別檢查 (引入 type variables)

## p32

structurally equivalent: 結構一樣就一樣

e.g. int A[10], int B[10]: array of 10 int

name equivalent: 名字一樣才一樣 (?)

e.g. typedef int dollars: int, dollar 不同 type

## p33

看到資訊保留到 symbol table 中，不需要產生 IR 來說明這個 declaration

## p34

這個名字再 storage 的 Layout

型別也代表了寬度

## p35

symbol table 也存了擺放位置的資訊

> java bytecode 不需要考慮 memory 擺放的問題 (?)

## p36

擺放的位置可能需要被 4, 8 整除 >> alignment

e.g. 起始位置不能被 4 整除，可能需要 2 個 op 才能 access >> 加上 padding

## p37

存位置的目的? 為了將來 load data 到 reg 或 store data

## p38

計算 type expression 及 type 寬度 >> 透過撰寫 SDT

* 當看到 Int reduce 回 B 時，知道 type:int, 寬度:4
* 遞迴到右下的 C
* reduce 回 C: array(3, int), w:12
* reduce 回 C: array(2, array(3, int)), w:24
* reduce 回 T

## p39

看到新的 compound stmt，push 一張新的 symbol table 到 stack

## p40

new 出一張新的 symbol table，進入裏頭的 symbol table 時先處理完外面的 symbol table，進入後歸零 offset

## p41

> Summary: 當看到變數宣告就把變數家道 symbol table，存 type, offset

Heap 裏頭的擺放不是由 compiler 管理，e.g. malloc 就交給 malloc 的 library 管理

## p43

1. 先執行 E 的 IR 存到 E 的 address

## p44

1. -E reduce 回 E > t1
2. E + E reduce 回 E > t2
3. id = E reduce 回 S > a = t2

## p48

算陣列的 Address 位置

## p49

L.array: 是放在symbol table的哪個entry
L.type: subarray的型別
L.addr: temp, 計算offset是多少

## p52

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

## p53

type synthesis: static-typed
type inference: dynamic-typed

## p55

型別轉換

## p57

## p59

function overloaded

## p60

adhoc polymorphic

$\forall \alpha list(\alpha) \to integer$

## p61

推導type expression

用 type inference rule (for 動態型別語言)

## p62

3) boolean condition + 2個type variables

## p63

unify: 要把 type variable 取代成實際的 variable

$E1(E2)$ 

## p64

$S(t)$ t 的 instance，把 t 實體化

sub: type expression 做實體化
某一個 sub 是兩個 Type expression 的 unifier

## p65

## p66

給 2 個 node 判斷能不能 unify

## p67

編號: unique number

檢查1.9可不可以被unify

## p71

control flow statement

## p77

fall through 簡化 goto

## p83

可不可以一邊parsing的過程產生右邊的IR?

## p84

backpatching

jump 與 target 做 match

先產生goto，taget先不要填，等address知道後再填

## p85

不用 Inherit attribute

truelist，一堆goto，當b是true會執行的goto
falselist，...，當b是false會執行的goto