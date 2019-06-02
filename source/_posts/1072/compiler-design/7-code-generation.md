# Code Generation

## p3

1. 選擇 instruction 實作 high-level IR
2. register 有限 variable 無限，要 register allocation
3. 對 instruction 順序做排序 (VLIW)

## p5

相同的事情可能可以有很多種不同的做法

## p10

多種 solution 選 instruction 最少的

## p11

產生額外的 load/store，可能有不同的 optimize

## p13

pipeline IF/ID/EX/MEM/WB

s1 與 s5 共用同一個 register: 因為在 pipeline 一個讀一個寫沒有重疊

## p14

edges: live range 有 overlap

相鄰兩個點不能放在同一個 register

相鄰兩個點不能塗同個顏色

## p17

有 k 個顏色，找到一個 node 最多有 k-1 個 edges，把 node 拿掉放到 stack，重複做 reduction 直到圖空了

從 stack pop 出來的順序圖顏色

## p30

不能塗色: 每個 Node 最少有 k 個 edges

## p40

register 不夠用了

## p43

想辦法挑一個暫時不用的 node spill 出去

## p46

sequential code > assembly code

真正的 schedule: 硬體
compiler: optimization

## p47

compiler: 必要的，產生 parallel machine
硬體設計簡單，不用做 schdule

## p48

hardware binding

## p49

compiler binding，再 compile 的時候就決定在哪個時間點執行甚麼指令

instruction 與 functional unit 是誰來做? hardware or compiler

## p45

instruction schedule 不一定必要，做 optimization

* structural hazard: 想要執行兩道乘法，硬體需要有兩個乘法器，判斷是否有足夠硬體資源可以同步執行
* data hazard: operation 有 dependency
* control hazard: 與 control flow 有關係，e.g. conditional branch，要走哪邊 depend on conditional 的值

## p51

basic block: single entry singe exit
     _
--> | | -->

local: 同一個 basic block
global: 跨 basic block

## p53

把 dependency 關係畫成 DAG

## p54

ready: 沒有 parent 的 node

## p56

有很多 ready state 要挑哪個?

* 挑到 leaf 最長的 (很有可能是 critical length)
* 有最多的 child

## p64

constant folding: 把 constant 再 compile time 算完

