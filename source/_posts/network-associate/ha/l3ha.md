---
title: 'Layer 3 HA'
---

# Layer 3 HA

為了提供高可用性，多層交換器必須提供防止 GW 出現故障而導致整個 VLAN 被隔離的機制。

本章討論路由器備援的方式有三種，包括：

* 熱備援路由器協定 (HSRP, Hot Standby Router Protocol)
* 虛擬路由器備援協定 (VRRP, Virtual Router Redundancy Protocol)
* 閘道負載平衡協定 (GLBP, Gateway Load Balancing Protocol)

這些通常也被稱為首站備援協定 (FHRP, first-hop redundancy protocol)，因為第一個路由器站提供了高可用性

## 熱備援路由器協定

HSRP 是 Cisco 專屬的協定，它讓多台 Router (或 L3 Switch) 使用同一個 Gateway IP。RFC 2281 更詳盡地描述了這種協定。

為了使用同－個 GW IP，提供備援的所有路由器都會分配到同一個 HSRP 群組。

* 其中一台當選為 active HSRP router
* 另一台作為 standby HSRP router
* 其他路由器則處於聆聽 HSRP 狀態

路由器定期地交換 HSRP Hello 訊息，以便能夠知道彼此及 active 路由器的存在。

![只有交換器 A 會執行轉送](2019-07-24-09-36-26.png)

* 可以分配 0 ~ 255 任何的群組編號給 HSRP 群組
* 大多數 Catalyst 交換器最多只支援 16 個不同的 HSRP 群組編號
* HSRP 群組只在介面本身上才有意義，VLAN 10 介面上的 HSRP 群組 1 和 VLAN 11 介面上的 HSRP 群組 1 是各自獨立的

### 選舉流程

根據群組中每個路由器設定的優先權 0 ~ 255 來選舉，預設優先權為 100

* 優先權最高的路由器成為群組中的 active router
* 優先權次高的路由器成為群組中的 standby router
* 若優先權都相等，則 HSRP 介面的 IP 位址最高者便成為 active router

參與 HSRP 的設備之介面必須依序經歷下列狀態始能進入 active 狀態：

1. 停用 (Disabled)
2. 初始化 (Init)
3. 聆聽 (Listen)
4. 交談 (Speak)
5. 待命 (Standby)
6. 現用 (Active)

細節：

* 只有 standby router 會監控來自 active router 的 Hello 訊息
* 預設 Hello 訊息每隔 3 秒鐘發送一次
* 如果在 Hold-Time (預設為 10 秒鐘或 Hello 計時器的 3 倍) 內沒有收到 Hello 訊息
  * 將認為 active router 已失效，standby router 將承擔 active 角色
  * 如果其他路由器處於聆聽狀態，優先權次高的路由器將成為 standby router
* 在 active router 故障後又恢復後，無法重新恢復 active 狀態
* 最先啟動的路由器就算權值較低，仍會成為 active router
* 如果某台路由器的優先權在任何時候都是最高的，可將路由器設定為接管 active 角色：
  * `Switch(config-if)# standby <group> preempt [delay [minimum <seconds>] [reload <seconds>]]`
  * 預設情況將立即接管，delay 可以延遲接管
    * minimum：在嘗試取代優先權較低的 active 路由器以前所等待的秒數 (0~3600)
    * reload：指定路由器重新啟動後所等待的秒數 (0~3600)

設定：

* 權值：`Switch(config-if)# standby <group> priority <priority>`
* 計時器：`Switch(config-if)# standby <group> timers [msec] hello [msec] holdtime`
  * hello 和 hold-Time 計時器的值以秒或毫秒為單位，若要以毫秒為單位，需要在值前面指定關鍵字 msec
  * hello 計時器的範圍為 1 ~ 254 秒或 15 ~ 999 毫秒。
  * hold-Time 至少應為 hello 計時器的 3 倍，其範圍為 1 ~ 255 秒和 50 ~ 3000 毫秒

### HSRP驗證

防止非核准的設備進行欺騙或參與 HSRP。在同一個 standby 群組中，所有路由器都必須使用相同的驗證方法和金鑰。

#### 明文HSRP驗證

發送的 HSRP 訊息包含明文金鑰字串 (最多8個字元)，這是一種簡單的驗證 HSRP 鄰居的方法。如果訊息中的金鑰字串與 HSRP 鄰居上設定的金鑰相同，訊息將被接受。

* 明文驗證很容易被搆截並用來假冒合法鄰居
* 明文驗證旨在防止採用預設值的鄰居參與 HSRP
* Cisco 設備使用的預設金鑰字串為 `cisco`

`Switch(config-if)# standby <group> authentication <string>`

#### MD5驗證

將密鑰以 MD5 算出雜湊值傳出，收到的路由器以自己的密鑰計算雜湊值，並比對訊息內容的雜湊值，如果相同則接受訊息。

`Switch(config-if)# standby <group> authentication md5 key-string [0 | 7] <string>`

> 0: string 為明文指定，在 config 為加密儲存
> 7: string 為加密指定

另外，也可以將 MD5 金鑰字串定義為金鑰鍊(keychain)中的金鑰。這種方法更靈活，讓你能夠在交換器上定義多個金鑰：

```
Switch(config)# key chain <chain-name>
Switch(config-keychain)# key <key-number>
Switch(config-keychain-key)# key-string [0 | 7] <string>
Switch(config)# interface <type> <mod/num>
Switch(config-if)# standby <group> authentication md5 key-chain <chain-name> 
```

如果需要修改金鑰，只要在金鑰鍊中加入新金鑰，並刪除舊金鑰即可。

### 重新選舉

HSRP 提供偵測鏈路故障並重新選舉的機制，HSRP 會追蹤特定的介面時，只要發現介面失效，就會根據設定值來調降優先權的大小；如果追蹤多個介面，每當一個介面發生故障時，就降低優先權一次；如果介面恢復，則優先權就會將同樣的值增加回來。

欲配置介面追蹤，可使用：`Switch(config-if) # standby <group> track <type> <mod/num> [decrementvalue]`，預設 decrementvalue 為 10。

另一台路由器僅在下面兩個條件滿足時才能接管 active 角色：

* 另一台路由器的 HSRP 優先權現在變得比較高了
* 路由器在自己的 HSRP 組態中設定了接管

### HSRP閘道定址

* HSRP 群組中的每一部路由器，都分配給介面唯一的 IP 位址。該位址用於所有路由協定，以及往返於該路由器的管理資料流。
* 每台路由器都有一個共同的閘道 IP 位址 (虛擬路由器位址) ，該位址也被稱為 HSRP 位址或 standby 位址。用戶端可以將這個虛擬路由器位址作為 default GW，由 HSRP 確保該位址的可用性。

`Switch(config-if)# standby <group> ip <ip-address> [secondary]`

> 對於虛擬路由器位址來說，HSRP 為其定義了 0000.0c07.acxx 的特殊 MAC 位址，其中 xx 代表 HSRP 群組編號是以十六進位的二位數表示
> 例如，HSRP 群組1為 0000.0c07.ac01、HSRP 群組16為 0000.0c07.ac10

### HSRP負載平衡

採用兩個 HSRP 群組 (一個群組分配一台交換器):

* 一個群組裡的一台交換器成為active路由器
* 另一個群組裡的一台交換器成為active路由器

如此，可以同時使用兩個不同的虛擬路由器或閘道位址。

> 使用單一的 HSRP 群組時，無法在連接到兩台 HSRP 路由器的 uplink 之間進行負載平衡。

另一個方式，讓每一台交換器成為另個 HSRP 群組的 standby router。換句話說，每台路由器在某個群組中是 active 對另一個群組而言則是standby。

![採用兩個HSRP群組的負載平衡](2019-07-24-10-48-09.png)

### 檢驗

`Router# show standby [brief] [vlan <vlan-id> | <type> <mod/num]`

* `show standby brief`
* `show standby interface <type> <member/module/name>`

## 虛擬路由器備援協定

VRRP 由 IETF 標準 RFC 2338 所定義的協定，可取代 HSRP。

VRRP 與 HSRP 的差異：

* active router 被稱為主要路由器 (master router)，其他所有路由器都處於備用狀態(backup state)
* VRRP 群組編號的範圍為 0~255
* 路由器的優先權範圍為 1~254 (預設為100)
* 虛擬路由器的 MAC 位址 0000.5e00.01xx
* VRRP 的通告每隔 1 秒鐘發送一次，備用路由器可從主要路由器得知通告的時間間隔
* 預設情況下，如果 VRRP 路由器的優先權更高，將接管目前主要路由器的角色

設定：

* 權值：`Switch(config-if)# vrrp <group> priority <level>`
* 計時器：`Switch(config-if)# vrrp <group> timers advertise [msec] <interval>`
* 從 master 路由器得知通告的時間閒隔：`Switch(config-if)# vrrp <group> timers learn`
* 停用接管(預設為接管)：`Switch(config-if)# no vrrp <group> preempt`
* 修改接管延遲(預設為0秒)：`Switch(config-if)# vrrp <group> preempt [delay <seconds>] `
* 對通告進行驗證：`Switch(config-if)# vrrp <group> authentication <string>`
* 指定虛擬IP位址：`Switch(config-if)# vrrp <group> ip <ip-address> [secondary]`
* 追蹤物件：`Switch(config-if)# vrrp <group> track <object-number> [decrement <priority>]`

檢驗：

* `show vrrp brief all`
* `show vrrp interface <type> <member/module/name>`

## 閘道負載平衡協定

GLBP 是 Cisco 專屬的協定，來克服現有的備援路由器協定的缺點。(Cisco I0S > 12.2(14)S)

為了提供虛擬路由器，得將多台交換器(路由器)分配到同個 GLBP 中。群組中所有的路由器都能夠參與負載平衡，分攤轉送部分的資料流，並非單獨讓 active 路由器代表虛擬路由器位址來轉送資料流。

這種做法的優點是，用戶端無須指向特定的閘道位址，所有用戶端的<mark>預設閘道位址都相同</mark>，那就是虛擬路由器的IP位址。

原理：當用戶端發送查詢虛擬路由器位址的 ARP 請求時，GLBP 傳回一個 ARP 回應，其中包含從群組中選擇的路由器之虛擬 MAC 位址 (virtual MAC address)。

### 現用虛擬閘道

其中一台路由器被選作 (優先權或IP位址在群組中最高) 現用虛擬閘道 (AVG, active virtual gateway)

AVG負責：

* 回應所有關於虛擬路由器位址的 ARP 請求，其回送的 MAC 位址取決於配置的負載平衡演算法
* 分配必要的虛擬 MAC 位址給 GLBP 群組中的每部路由器
  * 每個群組最多可使用 4 個虛擬 MAC 位址
  * 每台路由器都被稱為現用虛擬轉送器 (AVF, active virtual forwarder)，負責轉送虛擬 MAC 位址上所收到的資料流
* 分配次要角色，群組中的其他路由器都是備援或次要虛擬轉送器，以防AVG的停擺。

設定：

* 優先權：`Switch(config-if)# glbp <group> priority <level>`
  * 群組編號的範圍為 0 ~ 1023
  * 路由器的優先權為 1 ~ 255，預設為 100
* 啟用接管：`Switch(config-if)# glbp <group> preempt [delay minimum <seconds>]`
* 調整計時器：`Switch(config-if)# glbp <group> timers [msec] <hellotime> [msec] <holdtime>`
  * hellotime 範圍為 1 ~ 60 秒或 50 ~ 60000 毫秒，預設為 3 秒
  * holdtime 必須比 hellotime 大，最大為 180 秒，預設為 10 秒，最好設定為 hellotime 的 3 倍

### 現用虛擬轉送器

GLBP 群組中的每一台路由器都可成為 AVF (假如 AVG 指派該角色及虛擬 MAC 位址給它)。虛擬 MAC 位址永遠為 0007.b4xx.xxyy 之形式：

* xx.xx 代表的 16 位元值相當於 6 位元的 0 加上 10 位元的 GLBP 群組編號
* 8 位元的 yy 值是虛擬轉送器的編號

預設情況，GLBP 也使用定期的 Hello 訊息來偵測 AVF 故障。

1. GLBP 群組中的每一台路由器都向其他所有 GLBP 鄰居發送 Hello 訊息，並期望收到來自其他所有鄰居的 Hello 訊息
2. 如果在 Hold-Time 計時器到期之前，沒有收到來自 AVF 的 Hello 訊息，AVG 將認定 AVF 出現了故障，進而將 AVF 角色分配給另一台路由器

被授予新 AVF 角色的路由器，可能已擁有另一個虛擬 MAC 位址，AVG 維護了兩種計時器，來解決重複的問題：

* 重新導向 (redirect) 計時器
  * 用來決定 AVG 何時會停止在 ARP 回應中使用舊的虛擬 MAC 位址
  * 符合舊位址的 AVF 將繼續扮演用戶端的閘道角色
  * 預設為 600 秒 (10分鐘)，範圍 0 ~ 3600秒(1小時）
* 逾時 (timeout) 計時器
  * 到期後將從所有 GLBP 鄰居刪除舊的 MAC 位址並使用它的虛擬轉送器
  * AVG 假設出現故障的 AVF 不會恢復，因此必須收回分配給它的資源
  * 此時，仍使用舊 MAC 位址的用戶端，必須在他們的 ARP 快取中更新該項目，以獲得新的虛擬 MAC 位址
  * 預設為 14400 秒 (4小時），範圍 700 ~ 64800秒(18小時） 

手動設定計時器：`Switch(config-if)# glbp <group> timers redirect <redirect> <timeout>`

> 雖然可以重複的部分可以偽裝為提供多個虛擬 MAC 位址，但長期下來沒有意義

GLBP 還可以使用權重函數來決定哪一台路由器可以成為群組中虛擬 MAC 位址的 AVF。每台路由器開始都有一個最大權重，當特定介面失效時，則根據設定值來降低權重。GLBP 使用臨界值來決定路由器能否成為AVF：

* 如果權重低於臨界值的下限，路由器必須放棄AVF角色
* 如果權重超出臨界值的上限，則路由器可繼續承擔其AVF角色。

首先，必須定義想要追蹤的介面：`Switch(config)# track <object-number> interface <type> <member/module/number> {line-­protocol丨ip routing}`

> object-number 是個任意的索引值 (1~500)，用於調整權重
> 觸發的條件可以是 line-protocol (介面的線路協定狀態變成up)或 ip routing (啟用IP路由、替介面指定IP位址及介面處於 up 狀態)

接著，定義介面的權重值：`Switch(config-if)# glbp <group> weighting <maximum> [lower <lower>] [upper <upper>]`

> 最大權重的範圍 1 ~ 254 (預設為100)

最後，對介面配置 GLBP，使其知道要追蹤哪些介面以調整權重：`Switch(config-if)# glbp <group> weighting track <object-number> [decrement <value>]`

> 追蹤的介面失效時，權重將降低 value (1 ~ 254，預設為10)

### GLBP負載平衡

AVG 利用固定方式將虛擬路由器 MAC 位址分配給用戶端以建立負載平衡。

顯然的，AVG 首先必須告訴群組中的每個 AVF 應該使用的虛擬 MAC 位址。每個群組最多可使用4個虛擬 MAC 位址，這些位址是依照順序分配的。

可使用下面的負載平衡方法之一：

* 輪替式(Roundrobin)
  * 預設的方法
  * 收到新的虛擬路由器位址 ARP 請求後，在回應中提供下一個可用的虛擬 MAC 位址
* 權重(Weighted)
  * GLBP 群組介面的權重決定了應該被傳送至 AVF 之資料流所占的比例
  * 權重越高，包含路由器虛擬 MAC 位址的 ARP 回應頻率就越高
  * 如果沒有配置追蹤介面，則使用最大權重值來設定 AVF 之間的相對比例
* 主機的依賴性(Host-dependent)
  * 為了得知虛擬路由器位址而產生 ARP 請求的每個用戶端，總是希望收到相同的虛擬 MAC 位址回應

`Switch(config-if)# glbp <group> load-balancing [round-robin | weighted | host-dependent]`

### 啟用GLBP

分配一個虛擬IP位址給群組：`Switch(config-if)# glbp <group> ip [<ip-address> [secondary]]`

> 如果沒有指定ip-address，將從群組中其他路由器那裡得知
> 如果路由器成為 AVG 必須明確地配置IP位址；否則，沒有路由器知道這個值是多少

### 例子

![](2019-07-24-22-20-41.png)

* A 被選為 AVG，從事協調整個 GLBP 程序的工作
* AVG 負責回應所有關於虛擬路由器 192.168.1.1 的 APR 請求，它將自己、交換器B和C定義為該群組的 AVF
* 使用輪替式負載平衡，每個用戶端 PC 依次查詢虛擬路由器位址，依序從左到右；AVG 每次回應時，都將下一個虛擬 MAC 位址回送給用戶端

![](2019-07-24-22-28-35.png)

如果某個 AVF 出現故障，有些用戶端仍記住上次得知的虛擬 MAC 位址。因此，另一台路由器將接替失效的路由器之 AVF 角色，得以持續供應虛擬 MAC 位址

1. A 的 GLBP 優先權最高，因此成為 AVG
2. A 出問題後，B 成為 AVG，並使用合適的虛擬 MAC 位址回應有關閘道 192.168.1.1 的 ARP 請求
3. A 曾經是個 AVF，參與過閘道負載平衡，因此 B 也將接管這類職責

### 檢驗

`show glbp [<group>] [brief]`