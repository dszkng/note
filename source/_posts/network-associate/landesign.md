# LAN Design

## Introduction

建置網路不是只有簡單的配線、接線工程，還有包含很多其他議題；scaling, redundancy, protection, load balance...也同樣重要。

Campus local area network (園區區域網路) 的設計可以從簡單的一台 switch 的設定到支援多棟大樓、教室、辦公室等的大型網路。

本章將探討如何有系統的設計及選擇建置網路的策略。

## 園區有線網路設計

### 擴展網路的需求

當一個企業不斷成長，雇用更多的員工、開設分公司、擴展到全球企業，這些關鍵因素都會直接影響到網路的設計以及擴展性。

一個好的企業網路需要有：

* 使 critical applications 能穩定運作
* 支援 converged network (combination of data, voice and video traffic)
* 支援分散的企業服務
* 集中管理網路設備

園區網路的設計往往包含分散在不同地理位置的許多小型網路(許多LAN)。

### 階層式網路架構

階層式網路架構通常包含三層：

* Access layer
* Distribution layer
* Core layer

每個 layer 各司其職：

* Access layer 提供 endpoint 使使用者連結到網路
* Distribution layer 聚合 access layer 並提供與各 service 間的連接
* Core layer 為整個 LAN 提供 distribution layer 的連接

有些小型企業將 Distribution layer 和 Core layer 結合，以節省成本與複雜度。

### 擴展網路

#### Scalability

* 使用可擴展、模組化的設備，在升級時不會動到主服務
* 使用 clustered devices，簡化管理及設定的複雜度
* 使用階層式網路架構
* 對 IP 做好管理，減少重新分配 IP 的問題
* 選擇 router 或 multilevel switch，減少 broadcast，並過濾掉不想要的封包

進階需求

* 在 criical service 間及 access layer 及 core layer 間做 network redundant
* 使用 etherchannel 提高 bandwidth 及 loadbalance
* 選擇合適的路由協定，減少更新路由表並縮小路由表的大小
* 無線網路

#### Redundency

* Redundant device: 安裝重複的設備並提供網路連線失敗時接管的功能
* Redundant path: 當網路連線失敗時，走另一條 physical path

Redundant path 在 switch 中可能導致 Layer 2 loops，於是便需要使用 [STP](/NCTU-Coursenote/network-associate/layer2/stp) 來解決這個問題。(簡單來說就是 redundant path 只有在需要的時候才通，不需要時 disable)

#### Failure Domains

Failure domain: 當 critical device 或 network service 有錯誤時造成的影響範圍

一個好的網路架構也要有較小的 failure fomain，減少網路錯誤時對用戶的影響。

#### 提升 Bandwidth

bandwidth 太小可能造成瓶頸，使用 etherchannel 將多個實體 interface 結合成一個邏輯的 interface。

#### 擴展 Access Layer

提供無線網路

#### 選擇 Routing Protocal

OSPF/EIGRP

## 選擇網路設備

### Switch

* Campus LAN Switches
  * 提供 3 個 Layer
  * Cisco 2960, 3560, 3650, 3850, 4500, 6500, 6800
* Cloud-Managed Switches
  * Cisco Meraki
* Data Center Switches
  * Cisco Nexus, Cisco 6500
* Service Provider Switches
  * Aggregation switches: aggregate edge traffic
  * Ethernet access switches
* Virtual Networking
  * Cisco Nexus

考慮：

* Port Density
* Forwarding Rates
  * switch 一秒可以處理多少資料
  * 100Mb/s, 1Gb/s, 10Gb/s, 100Gb/s
* Power over Ethernet
  * 提供電力給裝置
  * e.g. IP phone, wireless access points
* Multilayer Switching
  * 通常部屬在 core/distribution layer

### Router

為什麼需要 Router：

* 連結不同區域的網路 (e.g. 公司到家裡、公司內部的不同區域)
* 提供 Redundant path
* 連結 ISP
* 轉換不同介面的封包 (e.g. ethernet to serial network)

Cisco Router 主要分三類：

* Branch Routers
  * optimize branch service on a single platform while delivering an optimal application experience accross branch and WAN infrastructure.
* Network Edge Routers
  * enable the network edge to deliver high-performance, highly secure, and reliable services that unite campus, data center, and branch network.
* Service Provider Routers
  * differentiate the service portfolio and increase revenues by delivering end-to-end scalable solutions and subscriber-aware service.

## Summary

* 階層式網路分為 Access layer、Distribution layer、Core layer，每個 layer 各司其職
* 一個設計良好的網路有
  * 小的 failure domain (Router 與 Switch 成對部署，當一個設備壞掉時不會使整個服務癱瘓)
  * IP 分配規則
  * 可擴展性
  * 合適的協定
  * 模組化 / 叢集化
* 重要的服務應該要有 redundency
