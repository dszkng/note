# Run-Time Environments

## p3

管理 runtime 的資源

* data layout
* 存取 variable
* 呼叫 function call 時要做什麼管理

compile time 與 run time 的合約

## p6

變數存在 memory 的位置是靜態還是動態 > static(global or static variables in C) or stack

malloc 管理 heap，哪裡有人使用哪裡沒有

## p7

## p9

機器狀態

local variables

spill: 把 reg spill 到 memory

why 要存機器狀態?

## p10

## p11

每個 function 有自己的空間 (disjoint)

```
main(){
    foo();
    bar();
}
```

浪費空間，有些 function 可能不需要同時存在，foo bar 原本可以用同一塊空間

不支援遞迴

## p20

call: preorder
return: postorder

## p25

access link: static link: 指向另一個 activation record 的pointer (當我要access 另外一個人的local variable)
control link: dynamic link

function 內再宣告 function

```
foo(){
    int x;
    bar(){

    }
}
```

return value: 有些存在 Register 不存在 Memory(速度較快) e.g. ARM R0

actural parameters: e.g. ARM R0~R3

## p26

擺放位置也有道理

caller 可以直接拿 return value

top_sp: 放在 saved machine statuc 與 Local data 間，方便存取 local variables

## p28

盡可能安插 sequence 在 callee，減少重複的 sequence

## p35

about access link

## p37

q 不見得是 p 的 caller，就無法存取 q 的 activation record >> 用 access link 存

## p42

存離最近的 parent 的 activation record

從 p 跳 np-nq 次

## p44

透過深度計算該指向誰

## p45

function argument 像是 pointer (run-time 才知道呼叫哪個 function)

## p49

program efficeienvy: data allocated，常用的放在一起

## p52

管理 Data 在 processor 該擺在哪個 register

## p53

把 basic block 放在同個 page : 第一道指令被執行，接下來東西會放到 cache 或 Physical memory

```
struct {
    int foo;
    int bar; // 不常執行
    int baz;
}
```

把 foo baz 擺在鄰近的位置

## p54

互調 i, j

m x n cache miss >> m cache miss

## p61

mark-sweep: heap 滿了之後

## p62

對每個 cell 維護一個 counter，紀錄有幾個 pointer 指向它

## p63

root set: aggregation record or stack (全域變數或是區域變數)

## p64

circular connect: memory leak

## p65

假設所有的 cell 都是 garbage

從 root set 出發，可以到達的地方標示它不是 garbage

剩下沒被標示的都是 garbage

缺點: 回收時機不頻繁