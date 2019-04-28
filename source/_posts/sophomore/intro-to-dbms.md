---
title: 資料庫系統概論
type: docs
---

# 資料庫系統概論

## Relation Model

* Superkey:unique
* Candidate:minimal superkey
* Primary:only 1,not null candidate
* Foreign:primary key in another schema

## SQL

### NULL

* count會數NULL,其他func不會
* NULL=NULL(算是重複列),除了where內

### INSERT UPDATE DELETE

### Constraint

### Assertion

### Trigger

## Storage

Data are stored and retrieved in units called **disk blocks** or **pages**.

### Components of disk

* Platters(磁盤): 一片一片的
* Tracks(磁軌): 一片上面的一圈
* Sector(磁區): 一圈上面的一部分
* Cylinder(磁柱): 不同Platters上同個Sector

### Accessing disk page

Time to access (read/write) a disk block:

* **seek time** (moving arms to position disk head on track) : ~20ms
* **rotational delay** (waiting for block to rotate under head) : ~10ms
* **transfer time** (actually moving data to/from disk surface) : ~1ms

> Key to lower I/O cost: reduce seek/rotation delays!

Next block concept:

1. blocks on **same track**, followed by
2. blocks on **same cylinder**, followed by
3. blocks on **adjacent cylinder**

### Buffer Management

Disk ⇄ Main Memory

若請求的 page 不在 buffer pool:

1. 選擇一個 frame 來 replace
2. 若 frame 是 dirty(被修改過)，寫回 disk
3. 讀取 page 到 frame 中

pin page 並 return address

> pin count：有人要求這個page的話++, 釋放\-\-。=0時可以被替換

### Record: Record Formats

#### Fixed length

![](/NCTU-Coursenote/img/computer-organization/2019-04-24-01-53-23.png)

* Information about field types same for all records in a file; stored in system catalogs.
* Finding i’th field requires scan of record.

#### Variable length

![](/NCTU-Coursenote/img/computer-organization/2019-04-24-02-01-35.png)

### Record: Page Formats

#### Fixed length

![](/NCTU-Coursenote/img/computer-organization/2019-04-24-01-53-44.png)

Record id = \< page id, slot #>

#### Variable length

![](/NCTU-Coursenote/img/computer-organization/2019-04-24-02-01-47.png)

### Files of Records

higher levels of DBMS operate on **records**, and **files of records**.

FILE: A collection of pages, each containing a collection of records.

#### Heap File

randomly ordered file, Implemented as a List

![](/NCTU-Coursenote/img/computer-organization/2019-04-24-02-14-17.png)

#### Heap File Using a Page Directory

![](/NCTU-Coursenote/img/computer-organization/2019-04-24-02-15-56.png)

### Indexes

k* = Data entry, k = search key value (用k找到k*)

* Primary index
    * Search Key 裡有 Primary Key
    * 其餘都是 Secondary index
    * Unique index: Search key contains a candidate key
* Clustered index
    * order of data records is close to order of data entries in the index
    * 其餘都是 Unclustered index
* Dense index
    * An index record appears for every search key value in file
    * 其餘都是 Sparse index
    * Every sparse index is clustered


## Query Processing

### Selection

* Sequential Scan
    * block transferred: br
    * seek: 1
* Binary Search
    * block transferred: log2(br)
    * seek: log2(br)

### Join

`$b_x = \# \, of \, block \, x$`  
`$n_x = \# \, of \, record$`  
`$t_T = 1 \, block \, transfer \, time$`  
`$t_S = 1 \, block \, seek \, time$`

* **Nested-Loops Join**
    * 用多層 for 迴圈
    * Case1: Minimal memory required = 3 blocks (R, S, result)
        * Blocks transferred: `$n_r * b_s + b_r$`
            * Scan R tuple once: `$b_r$`
            * Each record in R, must scan S: `$n_r * b_s$`
        * Seeks: `$n_r + b_r$`
    * Case2: S fits in memory
        * Blocks transferred: `$bs + br$`
        * Seeks: 2 (R & S ordered)
* **Block Nested-loops Join**
    * 將外層的結果存到 join buffer，內層的每一個 record 與整個 buffer 比，可以減少內層的迴圈數
    * Case1: Minimal memory required = 3 blocks
        * Blocks transferred: `$b_r * b_s + b_r$`
        * Seeks: `$b_r + b_r$`
    * Case2: S fits in memory
        * cost same as such in nested-loop join
    * Case3: M blocks (`$b_s > M - 2$`)
        * Blocks transferred: `$b_r * (b_s / (M - 2)) + b_r$`
        * Seeks: 2 (R & S ordered)
* **Index Nested-loops Join**
    * TOTAL COST:
        * `$b_r * (t_T + t_S) + n_r * c$` (c == the cost of index access)
* **Hash Join**
    * Case1: Smaller relation (S) fits in memory
        * for each tuple r, use the hash index on S to find tuples such that S.a = r.a
        * cost same as such in nested-loop join
    * Case2: Smaller relation (S) does NOT fit in memory 
        * Phase 1: Partition the relation R&S into disk using one hash func
          (Each S partition must fits in memory)
        * Phase 2: Read Si into memory, and build a hash index on it(another hash func)
          Read Ri, use the hash index to find matches

### B+ tree

* order = d, n = # of entries
    * root: 1 <= n <= 2d 
    * internal & leaf: d <= n <= 2d 
* all left child < K, right child >= K
* if overflow, internal node is pushed up, leaf node is copied up


## Index Structure

## ER Model

* strong entity
    * 具有key attribute的entity
    * 可獨立存在
* weak entity
    * 不具有key attribute的entity
    * 無法獨立存在的entity,需依附在其他entity才可存在的entity
