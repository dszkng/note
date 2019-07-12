---
title: 'VRF'
---

# VRF

企業網路需要隔離各種應用程式類型，使資料流量保持獨立。可以採用 Cisco 虛擬路由與轉送 (Virtual Routing and Forwarding, VRF)，VRF 讓單一實體的路由器扮演多個路由器的腳色。這些虛擬路由器的邏輯各自獨立，並擁有各自的路由表。

## 設定

`ip vrf {vrf-name}`

`ip vrf forwarding {vrf-name}`：介面或子介面的子命令，用來指派介面或子介面給 VRF 執行個體

`router ospf {process-id} vrf {vrf-name}`

## 檢驗

`show ip vrf`

`show ip route vrf {vrf-name}`

`ping vrf {vrf-name} {destination-ip}`