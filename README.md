# GKE 操作

# 首頁

## 簡介

本專案是 GKE 的操作手冊，主要是以 GCP 的 GKE 服務為主，包含了 GKE 的網路設定、存儲設定、安全設定等等。


## 目錄

- [GKE setup](https://github.com/CUTe-CCNL/GKE-operate/GKE-setup/README.md)

- [terraform setup GKE](https://github.com/CUTe-CCNL/GKE-operate/terraform/README.md)


## GKE

## Cloud Identity

[Cloud Identity](https://cloud.google.com/identity/)
GCP 的雲端的身分證明分兩種，一種是自己的帳號(unmanaged user)、一種是受組織管理的帳號(Organization-managed users)。受組織管理的帳號也有多一些方便於合作專案的功能

Cloud Identity 是一款身分認證即服務(IDaaS)解決方案，可以集中管理哪些使用者或群組可以存取 GCP 及 G Suite 資源，也能作為第三方應用程式的 IdP。
[管理工具：Google Admin](https://workspace.google.com/products/admin)
可以申請自己的 Cloud Identity，但需要有自己或組織的網域名稱(Domain) 只要申請的人有就可以。

Cloud Identity 在 GCP 之外有另外的操作頁面

## Cloud IAM
Cloud IAM 管理範圍只有在，gcp內部

## 超級管理員最佳做法
在申請組織的 Cloud Identity 之後，會新增一個管理帳號 Organization Admin，他可以進行後續組織內使用者的新增修改刪除等權限操作

## 資源階層
在gcp 內部所有的resourse都是用 porject 來分類的，類似於 Kubernetes 中的 Namespace。

> 組織機構 > 資料夾 > 專案 > 資源

- 每個專案都是完全獨立的，
- 適用於根據功能和存取權將相關資源分組。


照片

Projects 可以使用多層資料夾(folder)分類、資料夾內可放相關的 Projects，而每個 Projects 個別持有需要的資源

相當於 Windows 存取權限管理，對於上層資料夾的操作權限，會繼承給其內含的所有資料夾與專案

和作業系統一樣，內部的資料夾會繼承上層資料夾的權限設定。


## IAM 政策

照片

"IAM 角色" 可以賦予操作權限，這個 IAM角色 又可以被賦予給使用者帳號或群組，該使用者或群組則獲得了這個 IAM角色 所持有的權限

講義 p.15
而 IAM 角色的權限設定也可以設定預設(原始角色)以及特定場合(預先定義角色、自訂角色)所持有的權限

## VPC (Vitual Private Cloud)

照片

### 區域 region 與 可用區 zone

在台灣這一個 region(asia-east1，在彰濱機房) 假設有三個 datacenter(大樓a、b、c)，也就是三個 zone，這三個 zone 各使用自己的設備資源，各個 zone 的資源是被隔離開的

區域代號可以參考[說明](https://cloud.google.com/about/locations?hl=zh-tw#asia-pacific)

VPC 在不同 region 所使用的不同的 subnet 之間的溝通，防火牆沒有擋的情況下，不須再建立 proxy，GCP 會負責不同 subnet 之間的串接（目前是 GCP 獨有的功能）

### 子網路建立模式

子網路建立有兩種方式：自動或自訂。自動模式會直接幫你切子網路及其大小（/20，最高可以擴充到/16），也有預設防火牆規則等等；自訂模式則需自己切子網路，大小符合 RFC-1918 即可，防火牆規則可以勾選或之後建立

- 設定 VPC 網路的名稱、說明
- MTU
封包的最大大小，預設1460，建議1500（許多通訊協定使用）
- 是否啟動 IPv6
這項設定啟動之後就不可關閉
- 選擇自訂或手動
- 設定子網路
  - 設定子網路名稱、說明
  - region：台灣選擇 asia-east1
    區域代號可以參考[說明](https://cloud.google.com/about/locations?hl=zh-tw#asia-pacific)
  - 選擇使用IPv4 或 IPv6
  - 設定 IPv4 範圍 (ex: 10.0.0.0/24)
  - ...
  - Flow Log
    google 會記錄 Log，**會產生額外的傳輸流量**
  - 防火牆規則
    有一些規則可以勾選，預設會有兩條優先度最後的規則：阻擋所有傳入的封包、允許所有傳出的封包，若沒有設定其他防火牆規則就會套用這兩條規則

## GKE

### kubernetes platform components

GKE 有 standard 與 auto pilot 兩種不同價格的模式

standard：
- 有 controlplane

網路設定時
cluster networking
選 public cluster 的話每個機器都會有外網IP
選 private cluster 另需指定 controlplane 的 IP(因 GKE 會託管 controlplane)