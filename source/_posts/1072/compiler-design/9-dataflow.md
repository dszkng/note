# Data-Flow Analysis

知道程式中的某個時間點，我想要的資訊是什麼

## p7

y 這個變數已經被定義了，待會要使用他

看每個 bb y 在使用前沒有被重新定義 >> live

如果被重新定義，離開 bb >> not live

一個變數定義之後，在使用之前就是 live，最後一次使用後變成 dead

## p8

r6 最上面的 assignment 是 dead code，不影響程式結果

## p10

DEF set : LHS variables

USE set : 在 BB 有用但在 BB 中沒有被重新定義

## p12

|A|
/ \
B C

out{A} = in{B} U in{C}

IN = USE (用到但沒有重新定義) + (OUT - DEF) (扣掉被重新定義的)

## p28

any path: union
all path: intersaction

## p30

只要有一條可以過來就是 reach
兩邊都有到的是 avaliable