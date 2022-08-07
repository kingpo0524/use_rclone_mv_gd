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

![image](https://user-images.githubusercontent.com/106213982/183280304-43793c7d-9bd8-46fc-b1fe-40a9ab7e9e19.png)

## Rclone
![image](https://user-images.githubusercontent.com/106213982/183281693-699c4da2-2d6f-42a6-8b76-2009c1e273fc.png)

![image](https://user-images.githubusercontent.com/106213982/183281699-39a2eb80-5536-4e00-989b-5cf80a6693dc.png)

![image](https://user-images.githubusercontent.com/106213982/183281775-99a7cc49-06da-4740-bc06-a39c7a90d0c7.png)

![image](https://user-images.githubusercontent.com/106213982/183281890-7b9d27f2-4f9c-40d7-b0b0-0dca7efe7925.png)

![image](https://user-images.githubusercontent.com/106213982/183281900-c5d92688-e05f-4b38-991f-b1504113c88a.png)

### Rclone設定
1. 
![image](https://user-images.githubusercontent.com/106213982/183283192-18591747-a4a8-4641-a4b8-ffa8ebf38a3e.png)
2. 
![image](https://user-images.githubusercontent.com/106213982/183282567-6133ad7e-fb4b-4eaf-8ab9-3a0cfbfac23b.png)
3. 
![image](https://user-images.githubusercontent.com/106213982/183282586-5b6046cb-47d8-4c80-80e8-6027d6488eb7.png)
4. 
![image](https://user-images.githubusercontent.com/106213982/183283302-8376ef12-d7d5-4721-a0a7-66ddfe204476.png)
5. 
![image](https://user-images.githubusercontent.com/106213982/183282631-36cf3c3a-a190-4c7e-9e19-2345219c30d6.png)
6. 
![image](https://user-images.githubusercontent.com/106213982/183282646-e6db7727-8778-40fe-9890-d7aa4456a3b4.png)
7. 
![image](https://user-images.githubusercontent.com/106213982/183282658-777762bc-91cb-4660-8105-d7d71ce5b356.png)
8. 
![image](https://user-images.githubusercontent.com/106213982/183282672-117d0f41-6cca-409c-b537-ca33bb432d82.png)
9. 
![image](https://user-images.githubusercontent.com/106213982/183282695-b25cbcff-7ace-4027-be44-53b157448e98.png)
10. 
![image](https://user-images.githubusercontent.com/106213982/183282740-0ea65c45-ef1d-4874-a8fb-2cadd51029ee.png)
11. 
![image](https://user-images.githubusercontent.com/106213982/183282762-1a4cab0f-47a9-4324-89e1-176e3acfc24f.png)
12. 
![image](https://user-images.githubusercontent.com/106213982/183283399-d12f2d93-0310-4f03-a8f1-3d69999951f4.png)
13. 
![image](https://user-images.githubusercontent.com/106213982/183282780-ba054d70-9526-4c7e-ac1a-42288d5ad507.png)
14. 
![image](https://user-images.githubusercontent.com/106213982/183282796-ee4723a1-ceee-4481-95fa-fc346acba208.png)
15.
![image](https://user-images.githubusercontent.com/106213982/183282804-af6f9e7a-6034-4d5d-8798-f43f4fabe8e7.png)
16. 
![image](https://user-images.githubusercontent.com/106213982/183282831-aa86f83c-1bf8-4f69-9777-c5fb161bbea9.png)
17. 
![image](https://user-images.githubusercontent.com/106213982/183282836-dcd5e51c-fd09-45dd-9950-84278558ea61.png)
18. 
![image](https://user-images.githubusercontent.com/106213982/183282872-429ebd91-2197-4152-a702-3088a5792b64.png)
19. 
![image](https://user-images.githubusercontent.com/106213982/183282924-861a29b1-28ed-4fb9-af6a-74682891d432.png)
20. 
![image](https://user-images.githubusercontent.com/106213982/183282961-f945e299-ebf0-4199-a399-66046f03abd3.png)
21. 
![image](https://user-images.githubusercontent.com/106213982/183282994-6c928d5d-9ecf-42a8-917d-985a461e7ccc.png)
22. 
![image](https://user-images.githubusercontent.com/106213982/183283023-91af3164-07d7-4a26-8747-22d5a045a418.png)











