# 透過Google Cloud Platform (GCP)及rclone搬移google driver資料

本文參考:https://omega.idv.tw/kdb120/viewthread.php?threadid=4749

本文僅做為紀錄使用，針對操作過程遇到的問題進一步紀錄及調整文字說明，方便自己查閱。

本文僅根據自己需求，紀錄google driver搬移到另一個google driver帳號的過程。

感謝文章原作dc提供的方式，解決我教育帳號大量資料搬移的問題。

## 說明
GCP(Google Cloud Platform)為Google提供的雲端平台。在GCP申請一台雲端虛擬機器(以下稱VM，Virtual Machine)，並透過該VM以rclone工具，可以很輕易的在雲端硬碟不同帳號搬移大量資料，並且不會耗用本機的空間及頻寬。會找

