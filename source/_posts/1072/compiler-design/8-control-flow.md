# Control Flow

## Basic block

在同一個 basic block 指令有相同的 execuation condition

single entry single exit 中間不會跳離

* 程式第一道指令一定是 bb
* branch target 一定是 bb
* branch 的下一道指令一定是 bb

## p13

Profile-guided opt (PGO)

profile edge 會執行幾次

## p16

自己會 dominate 自己
x > y, y > z >> x > z
x > z, y > z >> x > y or y > x

到現在的 BB 前一定會執行的 BB

## p17

右圖: BB6: BB1, BB2

## p19

離我最近的 dominator

## p20

反過來想 exit

## p28

透過 dominate 知道 bakedge 再知道 loop

## p29

BB4>BB3: continue

## p32

compiler 無法優化

## p36

remainder loop 

## p37

iteration 不 depend on 某個數

## p41

把前 p 個指令拿出來

把不被整除的先拿出來，剩下的可以做 vectorization

## p43

目標是減少 branch 數量

## p46

branch 與 fall through 都跳到 L2

跳到 L2 做一些事情，直接搬 L2 的事過來

## p47

dead code elimination: dead code 有可能會執行，但執行不影響程式的執行結果

unreachable code: 不會執行的 code

## p48

對最常跑的路徑做 optimization

考慮 side entrance、side exit

