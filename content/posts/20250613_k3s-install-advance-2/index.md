---
date: 2025-06-13T03:30:00
draft: false
tags:
- k3s
title: k3s install advance 2
---
![alt](images/banner.png)  

<!--more-->
這一篇將深入探討如何在 K3s 叢集中整合進階元件，以提升其功能性、網路效能、負載平衡能力以及持久化儲存的彈性。我們將導入以下關鍵元件：

- CNI (Container Network Interface): Cilium (Without kube-proxy)
- Ingress Controller: HAProxy Ingress
- CSI (Container Storage Interface): Longhorn, Rook, OpenEBS

## introduce
### CNI: Cilium Without kube-proxy
Cilium 是一個基於 eBPF 技術的開源軟體，用於提供、保護和可觀察 Kubernetes 叢集中的網路連接。它提供了高性能的網路功能，並能實現細粒度的網路策略。

**Cilium Without kube-proxy**：
傳統上，Kubernetes 使用 kube-proxy 來處理服務的負載平衡和網路轉發。然而，Cilium 能夠利用 eBPF 直接在 Linux 核心中處理這些功能，從而繞過 kube-proxy。這樣做的好處包括：  
- **更高的性能**：減少了網路路徑中的跳數，降低了延遲。
- **更低的資源消耗**：無需運行額外的 kube-proxy 進程。
- **更強大的可觀察性**：eBPF 提供了對網路流量更深入的洞察。
- **更靈活的網路策略**：能夠實現更複雜和細緻的網路安全策略。

### ingress class: haproxy ingress
HAProxy Ingress 是一個基於 HAProxy 的 Kubernetes Ingress Controller。Ingress Controller 負責將外部流量路由到叢集內部的服務。HAProxy 以其高性能、高可靠性和靈活性而聞名，作為 Ingress Controller，它能夠提供：
- **高效的負載平衡**：支持多種負載平衡算法。
- **SSL/TLS 終止**：在 Ingress 層處理加密和解密。
- **基於內容的路由**：根據請求的 URL、Host 等信息將流量導向不同的服務。
- **高級流量管理**：如重寫 URL、添加自定義頭等。

### CSI: Longhorn
Longhorn 是一個輕量級、可靠且易於使用的 Kubernetes 分佈式區塊儲存系統。它由 Rancher Labs 開發，並作為 CNCF 沙箱項目。Longhorn 通過 CSI 接口為 Kubernetes 提供持久化儲存，其主要特點包括：
- **分佈式塊儲存**：將多個節點的儲存空間聚合起來。
- **快照和備份**：支持數據的快照、備份和恢復。
- **災難恢復**：提供跨節點的數據複製和故障轉移。
- **易於部署和管理**：通過 Kubernetes 原生方式進行部署和操作。

