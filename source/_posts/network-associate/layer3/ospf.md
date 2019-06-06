# OSPF

OSPF 主要分為三個階段：

1. 尋找鄰居：在兩台路由器上判斷是否需要交換拓樸資料
2. 交換拓樸資料庫：路由器將資料存在 LSDB 上，LSDB 包含：
    * Router 的資訊
        * RID、Interface、IP、Netmask、Subnet
    * 在每個 Interface 可以到達的 Router 清單
3. 路徑計算：由自己為出發點計算到其他 Router 的最短距離，選出每個子網路最好的 net-hop，將之加入 IP 路由表中

## 區域

每個 Router 上的 Interface 都屬於同一個 Area，而一些特殊的<mark>**區域邊界路由器**（Area Border Routers, ABR）</mark>則位在區域的界線上。

* 區域0稱為<mark>**骨幹區域**（backbone area）</mark>，負責連接所有其他區域。
* 在兩個非骨幹區域之間通過的封包必須經過至少一台的骨幹路由器。

![典型的階層式OSPF設計](2019-06-04-00-09-43.png)

在同一個區域的 Router 互相交換詳細的拓樸資訊，不同區域則透過 ABR 做簡短的通知（子網路/遮罩），但不包含其他區域的拓樸細節。

![Summary](2019-06-04-00-10-46.png)

以拓樸來看，來自其他區域的子網路均視為直接連接到此 ABR。

## OSPF術語名稱

SPF (Shortest Path First，最短路徑優先)

> OSPF分析LSDB的演算法，以便決定最佳(最低成本)的路徑。

LSDB (Link-state database，鏈路狀態資料庫)

> OSPF存放拓樸資料的資料結構。

LSU (Link-state Update，鏈路狀態更新)

> OSPF封包的名稱。包含拓樸的詳細資訊，特別是LSA。

LSA (Link-state Advertisement，鏈路狀態通告)

> 具有拓樸資訊的OSPF資料結構。LSA存放在LSDB中，在網路上透過LSU訊息傳送。

ABR (Area Border Router，區域邊界路由器)

> 至少連接到兩個區域的路由器，ABR保存每個區域的拓樸資料、計算每個區域的路徑，並負責交換區域之間的路徑。

DR (Designated Router，委任路由器)

> 同一個網段中可能有多個 Router，假設每個 Router 需要建立 (n-1) 條連線，若 Fully Mesh 的連線總數為 n(n-1)/2
> 
> 當 Router 越多連線數就越多，如此便多了很多不必要的通告。於是 OSPF 就在這些 Router 中選出一位 DR，所有 Router 只需與 DR 建立連線。
> 
> 每次有 Routing Update，Router 只需把 Update 傳給 DR，再由 DR 統一發放給其他 Router，這樣，除了 DR 外， Area 裡每隻 Router 只需處理一條連線。

BDR (Backup Designated Router，備用委任路由器)

> 監控 DR，並在 DR 故障時接替它。

## OSPF功能摘要

傳輸

> IP Protocal 89 (不使用 UDP、TCP)

權值 (Metric)

> 路徑中所有<mark>出境介面</mark>的成本總和

Hello Interval

> 路由器送出 OSPF Hello 的時間間隔

Dead Interval

> 路由器如果沒有在時間內收到任何 OSPF 訊息，會中止鄰居關係

Update 封包之目的位址

> 通常會送給 224.0.0.5 (所有SPF路由器) 以及 224.0.0.6 (所有的DR)

完整或部份更新

> 當探索到新鄰居後，執行完整更新。其餘使用部分更新

驗證

> MD5/plain-text

VLSM/無級式

> 每一條路徑包含遮罩，也允許 OSPF 支援不連續的網路和 VLSM

路徑標籤

> 當路徑重分配 (redistribute) 到 OSPF 時，允許 OSPF 為路徑加上標籤

Net-hop 欄位

> 若通告此筆路徑的路由器，與此路徑所指向的 Net-hop 不同時，就會用到此欄位

手動路徑彙整 (route summarization)

> 只允許在 ABR 上做路徑彙整

## 在 LAN 上的 OSPF

OSPF 在所有鄰居參數的核對都通過後，就會存在兩種鄰居關係：**鄰居**(neighbor)和**完全相鄰**(fully adjacent)關係。最後，OSPF 會使用具有 8 個狀態的 FSM，用於描述每一個 OSPF 鄰居目前的狀態。

### 啟用鄰居探索

1. 透過 `network` 或 `ip ospf area` 啟用 Interface 上的鄰居探索
2. Interface 尚未透過 `passive-interface` 設定為被動模式

當滿足以上條件，OSPF 就會傳送 Hello 到 224.0.0.5 (OSPF的群播位置)，接著檢查 Hello 本身的 RID，以及路由器分配給 LAN 的 OSPF Area。

![群播 vs 廣播](2019-06-04-00-40-35.png)

### 建立鄰居關係所必須相符的設定

當收到其他路由器的 Hello，OSPF 就會發現準鄰居，接著就會開始比對資料，判斷是否可以成為鄰居。以下為參考資料：

* OSPF RID
* 末梢（Stub）區域標籤
* Hello interval
* Dead interval
* Subnet mask
* Interface 可到達的鄰居清單
* Area ID
* Router 的 Priority
* DR 的 IP
* BDR 的 IP
* 驗證摘要

### OSPF RID 的唯一性

每個 OSPF 路由器都會為自己分配一個 RID，尋找順序如下：

1. 首先尋找 `router-id <rid>`
2. 尋找狀態為 up 的 loopback 最大的 IP
3. 尋找 up 的 interface 最大的 IP

若 OSPF RID 不遵守唯一性，學習網段時會與與形成鄰居的順序有關，如此會造成不可預期的結果。

> 相同 OSPF RID 的兩台路由器無法形成鄰居關係。

### 使用相同的 IP MTU

介面的 MTU 是設定從 Interface 轉送出去的封包的最大尺寸。當路由器轉送的封包超過 MTU 時，路由器會對封包進行分段，如果 IP header 中的 `DF` (Don't Fragment) 有設定，則會進行丟棄。

當兩個 OSPF 鄰居之間的 MTU 值不同時，其中一台路由器會試圖和其他 MTU 不同的路由器成為鄰居。

雖然路由器會出現在鄰居清單中 (透過 `show ip ospf neighbor` 列出的清單)，但這兩台路由器間<mark>不會交換拓樸資訊</mark>。且這兩台路由器計算路徑時，並不會將對方視為 Net-hop。

> 拓樸資料的交換永遠不會成功，鄰居狀態：EXSTART > DOWN > 其中一台 Router 會再度嘗試 > INIT

### 形成鄰居的必要條件

* Interface 的 primary IP 必須屬於相同的子網路
* 直連介面不可以是 passive
* 相同的 Area
* Hello interval 或 Dead interval 相符
* RID 唯一
* IP MTU 相符
* 必須通過鄰居的驗證（如果有設定的話）

## 在 WAN 上的 OSPF

須滿足[在 LAN 形成鄰居的必要條件](#形成鄰居的必要條件)，此外在不同類型的 WAN 鏈路上仍須考量其他因素：

與 OSPF 網路類型相關:

* 路由器是否使用 OSPF Hello 群播訊息來探索彼此，或者需要預先定義鄰居?
* 路由器會試著選出 DR 嗎？如果是的話，什麼樣的路由器有資格成為 DR?

視 WAN 服務而定:

* 每個路由器會和其他哪些路由器成為 OSPF 鄰居?

### OSPF 網路類型

網路類型 (依介面設定) 主導了 OSPF 的運行方式：

* 路由器是否能使用 Hello 探索鄰居
* 在連接的 Subnet 中是否存在兩台以上的 OSPF 路由器
* 在某介面中是否應該選出 OSPF DR

舉例來說，點對點鏈路和點對點 WAN 子介面預設使用的網路類型是「點對點」，代表除了該子網路裡只能存在二部OSPF路由器，以及透過 Hello 動態地探索到鄰居之外，這二台路由器並不會選出 DR。

| 介面類型       | 使用 DR/BDR | 預設的 Hello Interval | 鄰居的動態探索 | 相同子網路支援多台路由器 |
| -------------- | ----------- | --------------------- | -------------- | ------------------------ |
| 廣播           | 是          | 10                    | 是             | 是                       |
| 點對點         | 否          | 10                    | 是             | 否                       |
| Loopback       | 否          | -                     | -              | 否                       |
| 非廣播 (NBMA)  | 是          | 30                    | 否             | 是                       |
| 點對多點       | 否          | 30                    | 是             | 是                       |
| 點對多點非廣播 | 否          | 30                    | 否             | 是                       |

#### 點對點(P2P)

![](2019-06-04-01-14-03.png)

* 這條 link 兩端只有 2 個 point，互為 neighbor
* 在其中一端送，只會有另一端收

#### 廣播(broadcast)

![](2019-06-04-01-14-54.png)

* default network type
* 透過 broadcast 的方式傳送 OSPF packet
* 高擴展性，加入新 router 時，只需要設定新 router 即可
* 選 DR 和 BDR，防止更新路徑時傳送過多的 packet

#### 非廣播(Non Broadcast)

![](2019-06-04-01-15-18.png)

* 處理網路 protocol 不支援 broadcast，EX : frame-relay, ATM (PVCs)
* 在交流之前，需要先知道對方是誰
* 要傳給 n 個 neighbor，需要 ruplicate n 次

又分為:

* Point to Broadcast
  * 不模擬 broadcast 的方式，而是把多個 point-to-point 的組成 PVCs(Permenent Virtual Circuit)
  * 需要 ruplicate OSPF packet 傳送給多個 neighbor
  * 不選 DR/BDR
  * 這些 point-to-point link 可以使用同一個子網路
* Non-Broadcast Multi-Access (NBMA)
  * 需要設定每個 neighbor 的 IP
  * 用 unicast 模擬 broadcast 的方式
  * 有選 DR/BDR

### 鄰居關係

#### 訊框中繼點對點連線

將一對 VPC 兩端的路由器視為點對點拓樸，並分配一個子網路給每一對的路由器。

![](2019-06-04-01-23-17.png)

#### MLPS VPN

> 多重協定標籤交換技術 (Multiprotocol Label Switching，MLPS)

使用 Layer 3 傳遞，所以客戶端的路由器不用事先定義 PVC。

在服務供應商的網路邊界上，有 Layer 3-aware 的 PE (Provider edge)。與另一端 Layer 3-aware 的 CE (Customer edge) 相連，形成 OSPF 鄰居。

![](2019-06-04-01-30-44.png)

#### 都會型乙太網路

因為都會型乙太網路提供 Layer 2 的連線，所以客戶的路由器不會和服務供應商網路內的路由器形成 OSPF 鄰居，而是在客戶端間產生，使得該服務就像是大型的 WAN。

![](2019-06-04-01-31-43.png)

## OSPF 虛擬鏈路

在某些情況可能存在兩個骨幹區域，換句話說，有時非骨幹區域可能不方便直接連接到骨幹區域，舉例來說：

1. 現有網路需要加入新的區域，但新的區域沒有直接連接到Area 0

![](2019-06-04-01-36-32.png)

2. Area 0 中的鏈路斷路，形成一分為二的拓樸

![](2019-06-04-01-36-44.png)

### 概念

OSPF 虛擬鏈路支援兩台連接到<mark>相同非骨幹區域的 ABR</mark>，即使被許多其他路由器與子網路隔開，也能經由非骨幹區域建立鄰居關係。好比兩台路由器間位在 Area 0 的虛擬的點對點連線。

![在 Area 0 的路由器不但可以建立鄰居關係，也能在鏈路上 flood LSA](2019-06-04-01-39-20.png)

* 若要設定虛擬鏈路，兩台路由器都要設定對方的 RID，並參考到虛擬鏈路所通過的 Area
* 兩台路由器傳送一般的 OSPF 訊息類型，封裝在單點傳送 IP 封包，目的地為虛擬鏈路另一端的 IP 位址

虛擬鏈路兩端的 ABR 與其他普通的 ABR 運作大抵相同，除了：

1. 以<mark>單點</mark>傳送將所有的 OSPF 訊息傳送到鏈路另一端
2. 會在 LSA 上標示 DNA（Do Not Age，沒有期限），代表虛擬鏈路另一端的路由器，<mark>不必在平常 30 分鐘重整間隔期間重新洪泛 LSA 到虛擬鏈路上</mark>。
3. 路由器可以替虛擬鏈路指定 OSPF 成本

### 設定

使用 `area virtual-link` 命令加上參數

注意:

* 兩台路由器連接的中繼區域不可以是末梢(stubby)區域
* OSPF 鄰居驗證參數 (optional)
* Hello interval, Dead interval (optional)
* OSPF 成本

### 檢驗

`show ip ospf virtual-links`

`show ip ospf neighbor`

## LSA

| LSA 類型 | 一般名稱              | 描述                                                                                  |
| -------- | --------------------- | ------------------------------------------------------------------------------------- |
| 1        | Router                | 每個路由器都會建立自己的第 1 型 LSA, 用來描述本身連接到的每個區域。                   |
| 2        | Network               | 每個傳輸網路會有一個第 2 型 LSA。                                                     |
| 3        | Net Summary           | 由 ABR 建立並通告到其他區域，描述一個區域的第 1、2 型 LSA 所列的子網路。              |
| 4        | ASBR Summary          | 類似於第 3 型 LSA, 但會通告一條用來到達 ASBR 的主機路徑。                             |
| 5        | AS External           | 由 ASBR 建立，負責將外部路徑傳到 OSPF 裡。                                            |
| 6        | Group Membership      | 針對MOSPF而定義，Cisco IOS並不支援。                                                  |
| 7        | NSSA External         | 由NSSA區域內的ASBR所建立，以代替第5型LSA。                                            |
| 8        | Link LSA              | 只存在於本地鏈路的第 8 型 LSA· 路由器用來通告其鏈路區域位址給相同鏈路上的其他路由器。 |
| 9        | Intra-Area Prefix LSA | 用來傳送關於連接路由器的1Pv6網路（包括末梢網路）， 類似1Pv4網路的第1型LSA。           |
| 10-11    | Opaque                | 作為一般用途的LSA, 方便OSPF日後的擴充。                                               |

### 內部 LSA

OSPF 使用第 1、2、3 型 LSA 來計算 OSPF 領域內所有路由的最佳路徑。在後面會介紹使用 4、5、7 型 LSA 來計算外部路徑 (將路徑重新分配到 OSPF)。

#### 第 1 型

又稱為 Router LSA，根據 OSPF RID 來識別 OSPF 路由器。<mark>每個路由器</mark>本身會建立第 1 型LSA。並將此 LSA 洪泛到相同的整個區域。

OSPF 使用 32-bits 的 LSID (link-state identifier) 來辨識第一型 LSA，建立 LSID 時，會使用自己的 RID 當作 LSID。

* 區域內部的每個路由器本身會建立一個第 1 型 LSA。
* ABR 會建立多個第 1 型 LSA：每個區域一個。

傳送資訊:

* 對於每個沒有選出 DR 的介面，第 1 型 LSA 列有該路由器的 interface network id / netmask 與 interface 的 OSPF 成本。OSPF將這些子網路稱之為**末梢綱路**。
* 對於每個選出 DR 的介面，則列舉 DR 的 IP 位址與連接至傳輸網路的標記 (意指該網路存在第2型LSA)。
* 對於每個無 DR 的介面，若可以到達鄰居，則會列舉該鄰居的 RID。

![](2019-06-06-16-24-15.png)

以區域 5 為例，有一台內部路由器 R5，以及兩台 ABR R1、R2，這三台路由器各自建立第 1 型 LSA 並 flood 到區域 5 中，且都知道這 3 個相同的第 1 型 LSA。

![在區域 5 中的 3 個第 1 型 LSA](2019-06-06-16-28-57.png)

R5 的第 1 型 LSA 資訊:

* LSID 5.5.5.5
* 連結到末梢網路的三條鏈路，每條都包含子網路/遮罩
* 兩條鏈路連到鄰居 R1、R2

#### 第 2 型

又稱為 Network LSA，SPF 要求 LSDB 建構出來的拓樸必須具備節點 (路由器) 以及節點之間的連接 (鏈路)。OSPF必須以某種方式建構出區域網路，使得拓樸能夠呈現節點與鏈路。為了滿足需求，OSPF 使用了第 2 型 Network LSA 來解決問題。

> 實際上 ，OSPF 是根據介面上是否選出 DR，來選擇多重存取網路是否使用第 2 型 LSA。

##### DR

使用DR的主要目的：

* 建立第 2 型 Network LSA 並 flood 到該子網路。
* 協助子網路上資料庫交換

當 DR 和 BDR 都還不存在的時候，路由器使用以下規則：

* 選擇具有最高優先權（`ip ospf priority value`，default 1，max 255) 的路由器
* 若優先權相同，則選擇 RID 最高的路由器
* 根據次佳的優先權來選擇 BDR。如果優先權相同，則選擇次佳的 RID

當 DR 和 BDR 都存在的情況：

* DR 斷線，BDR 成為 DR，選新 BDR
* BDR 斷線，直接選 BDR

##### 概念

為了滿足「一條鏈路只能連到兩個節點」，OSPF 使用第 2 型 LSA 來建構多重存取網路 (超過2部以上的路由器連到相同子網路)

![](2019-06-06-17-00-34.png)

以上圖為例，由於 OSPF 無法讓 4 台路由器只透過一條鏈路連接到同一個子網路，因此 OSPF 定義了第 2 型 Network LSA 來做為虛擬節點 (pseudonode)。每個路由器的第 1 型 Router LSA 會列出至此虛擬節點的連結，再利用第 2 型 Network LSA 來建立它。

![](2019-06-06-17-03-32.png)

##### 例子

![](2019-06-06-16-24-15.png)

以區域 34 為例，R3、R4 連到相同的區域網路，代表會選出 DR (當至少有 2 台路由器滿足建立鄰居的必要條件時，就會成為鄰居，接著 OSPF 會在 LAN 上選出 DR)，若 R3、R4 優先權相同，則 R4 成為 DR。所以，R4 在該子網路中建立了第 2 型 LSA 並 flood，如下圖所示：

![](2019-06-06-17-10-41.png)

由此可知，OSPF 利用第 1 型、第 2 型 LSA 來建構出單一區域內所有的拓樸。一旦路由器使用 SPF 程序建立拓樸模型後，接下來就可以計算該區域裡每個子網路的「最佳路徑」。

#### 第 3 型

又稱為 Summary LSA，為了不讓所有路由器都知道 OSPF 領域中全部的第 1、2 型 LSA，ABR 並不會從某個區域將第 1、2 型 LSA 轉送到另一個區域中，反之亦然。如此可以縮小 LSDB，進而節省記憶體與縮短收斂時間等。

ABR 不將第 1、2 型 LSA 轉送到另一個區域中，而是用第 3 型 LSA (Summary LSA) 來通告這些區域的路徑。ABR 會替一個區域裡的<mark>每個子網路</mark>產生一個第 3 型 LSA，再把每個第 3 型 LSA 通告到其他區域裡。第3型 LSA (Summary LSA) 不包含所有拓樸的詳細資訊，僅負責彙整網段的資訊。

![](2019-06-06-16-24-15.png)

![單個區域的 LSDB](2019-06-06-17-25-01.png)

![三個區域的 LSDB](2019-06-06-17-26-07.png)

ABR 負責建立第 3 型 LSA 並 flood 到下一個區域裡。ABR 會替通告的子網段位址冠上 LSID，並將自己的 RID 新增到 LSA。(這樣路由器才知道是哪個 ABR 通告了該路徑)

> 使用第 3 型 LSA 之目的並非為了彙整路徑。這裡的 "Summary" 只是相較第 1、2 型 LSA 的詳細資訊表示較精簡的意思。

#### 限制 LSA 數量

`max-lsa number`，避免記憶體不足，假設超過 LSA 上限：

1. log
2. 忽略與等待所有鄰居關係中止
3. 刪除 LSDB
4. 重新新增鄰居

#### 摘要

| 類型 | 名稱    | 意義                 | LSID                | 建立者                 |
| ---- | ------- | -------------------- | ------------------- | ---------------------- |
| 1    | Router  | 一台路由器           | 路由器的 RID        | 每台路由器都會自行產生 |
| 2    | Network | 存在 DR 的子網路     | 改子網路中 DR 的 IP | 子網路裡的 DR          |
| 3    | Summary | 另一個區域內的子網路 | 子網路編號          | ABR                    |

## OSPF LSDB

