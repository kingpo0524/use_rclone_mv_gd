# 透過Google Cloud Platform (GCP)及rclone搬移google driver資料

本文參考:https://omega.idv.tw/kdb120/viewthread.php?threadid=4749

本文僅做為紀錄使用，針對操作過程遇到的問題進一步紀錄及調整文字說明，方便自己查閱。

本文僅根據自己需求，紀錄google driver搬移到另一個google driver帳號的過程。

感謝文章原作dc提供的方式，解決我教育帳號大量資料搬移的問題。

## 說明
GCP(Google Cloud Platform)為Google提供的雲端平台。在GCP申請一台雲端虛擬機器(以下稱VM，Virtual Machine)，並透過該VM以rclone工具，可以很輕易的在雲端硬碟不同帳號搬移大量資料，並且不會耗用本機的空間及頻寬。會找到這個方法是因為google提供的各種方式，針對比較大的檔案(e.g. 10G~)都無法正常處，常常是下載的到一半出現403權限不足的錯誤，或是同步處理漏資料等等，詳細原因我不清楚，只知道這問題困擾我非常久。因此，找到上面連結提供的方式，真的是替我解決了一個大問題。在操作過程，有一些可能是版本不同的因素，導致執行的方式與文章說明不同，本身對rclone工具又不熟悉，因而卡關，經過一些摸索最終完成的移轉。因此想寫下這篇作為紀錄，提供給親友使用也方便後續查詢。

## 事前準備
* 申請 GCP，要有Google帳號，帳號不一定要與google driver帳號一致，可另外申請一個專門帳號作為搬移資料的用途。
* 準備信用卡，註冊時需輸入信用卡資訊，此為google的驗證機制，不會實際收費。google目前提供300美元抵用金額(約台幣9000)，可以在90天內試用Google Cloud產品。即使超過試用期也不會直接收費，除非手動升級至付費帳戶。這部分可以再申請頁面看到相關說明。
* 如果是透過VM將某一個帳號下的Google Driver資料傳輸到另一個Google Driver帳號，應該是不收費的。<br>
參考連結:https://cloud.google.com/vpc/network-pricing?hl=zh-tw
![image](https://user-images.githubusercontent.com/106213982/183279257-57b10103-6fcd-4110-b0e9-c340cdafc672.png)
* 本文使用 rclone 版本為 v1.59.0

## 一. 申請Google Cloud Platform (gcp)
1. 申請網址: https://cloud.google.com/free?hl=zh-tw<br>
直接點集"免費試用"，這邊可以看到新客戶可以有300美元抵免額。
![image](https://user-images.githubusercontent.com/106213982/183279285-7058ceb4-8348-44b6-b916-cbf1f4a62895.png)
2. 接著直接登入，可以使用現有帳號，或是建立新帳號。
3. "機構或需求"依個人情況隨意填寫，服務條款全部勾選，直接按"繼續"。重要的是旁邊的說明"免費試用期結束後不會自動向您收費"。
![image](https://user-images.githubusercontent.com/106213982/183279314-ac73d936-1cfc-472c-a515-e6f2873c28b5.png)
4. 接著進行手機驗證。
![image](https://user-images.githubusercontent.com/106213982/183279316-d7e1d8a7-f58f-4f17-9893-6c648efbd520.png)
5. 接著驗證付款資訊<br>
帳戶類型 -> 個人<br>
稅務資訊 -> 未登記稅籍的個人<br>
填好必要的資料後按下 [開始免費試用]<br>
![image](https://user-images.githubusercontent.com/106213982/183279319-3c89fba8-7042-45ea-9ea9-301de7189117.png)
6. 
![image](https://user-images.githubusercontent.com/106213982/183279412-11cecf6e-8d31-4156-bc31-84a38f632988.png)

![image](https://user-images.githubusercontent.com/106213982/183279458-46a55057-45c6-4217-80ec-7afb40e203dc.png)

![image](https://user-images.githubusercontent.com/106213982/183279652-a60d765b-6ffa-4f87-8f28-1e6bf09958f2.png)

![image](https://user-images.githubusercontent.com/106213982/183279844-8e86ece4-d280-4562-9bb9-00f7246dadda.png)

![image](https://user-images.githubusercontent.com/106213982/183279830-7112bda2-f076-44a0-a978-8c77359fb1ea.png)

![image](https://user-images.githubusercontent.com/106213982/183280184-312a01a1-7951-4d74-8f82-f7976009d197.png)

![image](https://user-images.githubusercontent.com/106213982/183280194-98143866-9513-4cdb-8964-88296430c93f.png)

![image](https://user-images.githubusercontent.com/106213982/183280207-1ec8ec39-1469-43f0-a0fd-37541ed1df14.png)

![image](https://user-images.githubusercontent.com/106213982/183280214-f1aa568f-75fa-4466-a2d2-0fe821cc4464.png)






