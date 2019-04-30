# VLAN

## 區域網路?

<mark>區域網路中的所有網路設備都在同樣的廣播領域中</mark>，也就是說當這些設備送出一個廣播封包時，在廣播領域中的其他設備也都會收到這個廣播。

## 虛擬區域網路

如果沒有 VLAN，Switch 就會將它所有的介面視為相同的廣播網域。有了 VLAN 後，就能將特定的介面設定在同一個廣播領域中，這時在同一個 Switch 就會有多個廣播領域產生。

![](2019-04-30-23-33-51.png)

## VLAN Trunking

現在考慮一個問題：如果要在不同 Switch 中用相同的 VLAN 怎麼辦？在兩個 Switch 相連的 Interface 上要設定什麼 VLAN？

為了解決這個問題，便需要使用 VLAN Trunking 協定。<mark>這個協定透過在傳送的訊框中加上 VLAN 標頭 (包含VLAN ID)</mark>，讓 Trunk 接收端的 Switch 能夠知道訊框是來自哪個 VLAN，這時接收端的 Switch 就能將訊框轉送到相應的 VLAN 中。

![](2019-04-30-23-41-25.png)

Trunking 協定主流的有 Cisco 制定的 ISL，及 IEEE 802.1Q

### ISL (Inter-Switch Link)

ISL 將原始的乙太網路訊框完整的封裝在 ISL 標頭和標尾之間，**不會修改原始的乙太網路訊框**，DA 與 SA 分別是傳送端和接收端 Switch 的 MAC Address。

![](2019-04-30-23-46-30.png)

### IEEE 802.1Q

**將額外的 4 個位元組的 VLAN 標頭插入到原始訊框的標頭中**。和 ISL 不同的地方在於，802.1Q 修改過的訊框仍保有和原始訊框相同的 MAC Address。

由於標頭變長的緣故，封裝時會強制乙太網路訊框標尾的原始訊框檢查碼(FCS, Frame Check Sequence)必須根據整個訊框的內容來重新計算。

![](2019-04-30-23-48-37.png)

### ISL vs 802.1Q
    
1. 皆可設定 4094 個 VLAN (2^12^ - 兩個保留值(0, 4095))
    * VLAN ID 1 ~ 1005: 標準範圍
    * VLAN ID > 1005: 延伸範圍
2. 皆可針對不同 VLAN 支援獨立的 **擴展樹通信協定** ([STP](/NCTU-Coursenote/network-associate/layer2/stp))
    * 不同 VLAN 在 Switch 間的溝通可以走不同路徑，最大化網路的使用
3. <mark>IEEE 802.1Q 支援 **原生VLAN** (native VLAN)</mark>
    * 預設，802.1Q 在每個 Trunking 上定義了一個原生 VLAN (VLAN 1)，在原生 VLAN 傳送的訓框 **不會** 被加入 802.1Q 的標頭
    * 這時會有兩種情形，Trunk 另一端的 Switch：
        1. 支援 802.1Q：當收到一個沒有 VLAN 標頭的訊框時，就知道這個是原生 VLAN 的訊框，於是就將訊框往指定的 VLAN 轉送 (當然首先兩個 Switch 間必須先協商哪個 VLAN 才是原生 VLAN)
        2. 不支援 802.1Q：因為沒有加上 VLAN 標頭，所以 Switch 仍然可以辨識這個訊框
    * 原生 VLAN 的功能就是要讓 Switch 至少可以在一個 VLAN 裡傳送訊框

## IP Subnet 與 VLAN

<mark>在同一個 VLAN 的設備最好在相同的 Subnet 下</mark>，如果一個 VLAN 有包含多個 Subnet，表示不同 Subnet 間可以不用上 Router 或 Firewall 就可以進行溝通，可能會有安全性的疑慮，除此之外也會造成更多廣播的流量。

![不同 Subnet 下還是一定要上 Router](2019-05-01-00-18-20.png)