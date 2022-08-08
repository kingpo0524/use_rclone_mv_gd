# 透過Google Cloud Platform (GCP)及Rclone搬移Google Drive資料

本文參考:https://omega.idv.tw/kdb120/viewthread.php?threadid=4749

本文僅做為紀錄使用，針對操作過程遇到的問題進一步紀錄及調整文字說明，方便自己查閱。

本文僅根據自己需求，紀錄Google Drive搬移到另一個Google Drive帳號的過程。

感謝文章原作dc提供的方式，解決我教育帳號大量資料搬移的問題。

## 說明
GCP(Google Cloud Platform)為Google提供的雲端平台。在GCP申請一台雲端虛擬機器(以下稱VM，Virtual Machine)，並透過該VM以Rclone工具，可以很輕易的在雲端硬碟不同帳號搬移大量資料，並且不會耗用本機的空間及頻寬。會找到這個方法是因為google提供的各種方式，針對比較大的檔案(e.g. 10G~)都無法正常處理，常常是下載的到一半出現403權限不足的錯誤，或是同步處理漏資料等等，詳細原因我不清楚，只知道這問題困擾我非常久。因此，找到上面連結提供的方式，真的是替我解決了一個大問題。在操作過程，有一些可能是版本不同的因素，導致執行的方式與文章說明不同，本身對Rclone工具又不熟悉，因而卡關，經過一些摸索最終完成移轉。因此想寫下這篇作為紀錄，提供給親友使用，也方便後續查詢。

## 事前準備
* 申請 GCP，要有Google帳號，帳號不一定要與Google Drive帳號一致，可另外申請一個專門帳號作為搬移資料的用途。
* 準備信用卡，註冊時需輸入信用卡資訊，此為google的驗證機制，不會實際收費。google目前提供300美元抵用金額(約台幣9000)，可以在90天內試用Google Cloud產品。即使超過試用期也不會直接收費，除非手動升級至付費帳戶。這部分可以再申請頁面看到相關說明。
* 如果是透過VM將某一個帳號下的Google Drive資料傳輸到另一個Google Drive帳號，應該是不收費的。<br>
參考連結:https://cloud.google.com/vpc/network-pricing?hl=zh-tw
![183279257-57b10103-6fcd-4110-b0e9-c340cdafc672-compressed](https://user-images.githubusercontent.com/106213982/183292546-3ef47166-9eda-4fcb-8348-adf96fc76de1.jpg)
* 本文使用 rclone 版本為 v1.59.0

## 一. 申請Google Cloud Platform (gcp) 並建立虛擬機器VM
1. 申請網址: https://cloud.google.com/free?hl=zh-tw<br>
直接點擊"免費試用"，這邊可以看到新客戶可以有300美元抵免額。
![183279285-7058ceb4-8348-44b6-b916-cbf1f4a62895-compressed](https://user-images.githubusercontent.com/106213982/183292618-20b5b00f-3a09-43c5-b3da-d364f1896d40.jpg)
2. 接著直接登入，可以使用現有帳號，或是建立新帳號。
3. "機構或需求"依個人情況隨意填寫，服務條款全部勾選，直接按"繼續"。重要的是旁邊的說明"免費試用期結束後不會自動向您收費"。
![183279314-ac73d936-1cfc-472c-a515-e6f2873c28b5-compressed (4)](https://user-images.githubusercontent.com/106213982/183292846-51adb4cf-a101-43fe-8142-b1e57bb7a094.jpg)
4. 接著進行手機驗證。
![183279316-d7e1d8a7-f58f-4f17-9893-6c648efbd520-compressed](https://user-images.githubusercontent.com/106213982/183292886-1e1a90fd-d75b-4257-ac8c-df2c094c1e29.jpg)
5. 接著驗證付款資訊<br>
帳戶類型 -> 個人<br>
稅務資訊 -> 未登記稅籍的個人<br>
填好必要的資料後按下 [開始免費試用]<br>
![183279319-3c89fba8-7042-45ea-9ea9-301de7189117-compressed](https://user-images.githubusercontent.com/106213982/183292912-c0c66c0a-7f77-426e-baf8-ff62c53c98a4.jpg)
6. 回答問題，可隨意填寫
![183279412-11cecf6e-8d31-4156-bc31-84a38f632988-compressed](https://user-images.githubusercontent.com/106213982/183292926-f75ee9ee-6b00-4077-a7d7-4225a5476dbe.jpg)
7. 直接按"暫時略過"
![183279458-46a55057-45c6-4217-80ec-7afb40e203dc-compressed](https://user-images.githubusercontent.com/106213982/183292998-8da11744-ce63-4abd-857d-3874da14281c.jpg)
8. Compute Engine -> VM執行個體
![183279652-a60d765b-6ffa-4f87-8f28-1e6bf09958f2-compressed](https://user-images.githubusercontent.com/106213982/183293048-419a41c4-b8ad-43a6-a2ba-d3a0e6bdc114.jpg)
9. 啟用
![183279844-8e86ece4-d280-4562-9bb9-00f7246dadda (1)-compressed](https://user-images.githubusercontent.com/106213982/183295043-2e109bd0-0584-4136-9773-d1fef51c908f.jpg)
10. 建立執行個體
![183279830-7112bda2-f076-44a0-a978-8c77359fb1ea (1)-compressed](https://user-images.githubusercontent.com/106213982/183295064-bf479161-b5bb-4b2a-8482-7071a64764ec.jpg)
11. 機器設定<br>
名稱：自訂<br>
區域：us-central1 ， us-central1-a<br>
機器設定 -> 機器系列 -> 一般用途<br>
系列：N1<br>
機器類型：f1-micro<br>
![183280184-312a01a1-7951-4d74-8f82-f7976009d197 (2)-compressed](https://user-images.githubusercontent.com/106213982/183295089-bbbb67bd-0a66-49fc-82e7-cc500c117121.jpg)
12. 開機磁碟 -> [變更]<br>
![183280194-98143866-9513-4cdb-8964-88296430c93f (2)-compressed](https://user-images.githubusercontent.com/106213982/183295379-0e2f838e-fb8f-43e7-bd34-e6a278c58e29.jpg)
13. 開機磁碟<br> 
開機磁碟：公開映像檔<br>
作業系統：CentOS<br>
版本：CentOS 7<br>
開機磁碟類型：標準永久磁碟<br>
大小(GB)：30<br>
![183280207-1ec8ec39-1469-43f0-a0fd-37541ed1df14-compressed](https://user-images.githubusercontent.com/106213982/183295439-de6db5b0-4e0c-4835-99b2-028bbf7dabbf.jpg)
14. 安全性<br>
安全性 -> 受防護的 VM<br>
啟用 vTPM -> 取消勾選<br>
啟用完整性監控功能 -> 取消勾選<br>
右邊可以看到預估每月花費，但是因為我們在試用期，並且是Google Drive對傳，如果沒有手動啟用升級，是不會被收費的。<br>
點擊"建立"
![183280214-f1aa568f-75fa-4466-a2d2-0fe821cc4464 (1)-compressed](https://user-images.githubusercontent.com/106213982/183295642-2293cc62-5533-4c0b-875c-18bcea750caa.jpg)
15. 開啟VM終端機<br>
選擇"在瀏覽器視窗中開啟"<br>
到這裡，我們完成在GCP上建立虛擬機器VM，並且設定VM執行的作業系統為Linux CentOS7，透過瀏覽器視窗可以開啟連接VM的指令介面終端機。接著，我們可以開始操作VM，安裝Rclone等相關工具。然後就可以使用Rclone開始搬移Google Drive的資料。
![183280304-43793c7d-9bd8-46fc-b1fe-40a9ab7e9e19-compressed](https://user-images.githubusercontent.com/106213982/183295672-7a8f2c82-80f9-454e-b6e0-77aab76e23aa.jpg)

## 二. 安裝Rclone及其他方便的工具
1. 我們先安裝必要的 unzip 與 screen，unzip為解壓縮軟體，screen指令可以讓我們在斷開連線後再重新連回執行中的畫面查看執行結果<br>
輸入指令：sudo yum install -y unzip screen
![183281693-699c4da2-2d6f-42a6-8b76-2009c1e273fc-compressed](https://user-images.githubusercontent.com/106213982/183295723-a1bd8f81-8800-43a4-9070-02ec5adfe5e1.jpg)
2. 看到Complete! 即表示安裝完成。
![183281699-39a2eb80-5536-4e00-989b-5cf80a6693dc-compressed](https://user-images.githubusercontent.com/106213982/183295752-678adcd3-0c42-4f2f-9633-127ebb6c4486.jpg)
3. 將時間格式調整為台灣時間<br>
輸入指令：sudo ln -sf /usr/share/zoneinfo/Asia/Taipei /etc/localtime<br>
輸入指令：date<br>
![183281775-99a7cc49-06da-4740-bc06-a39c7a90d0c7-compressed](https://user-images.githubusercontent.com/106213982/183295785-d3805368-55fd-48d7-b814-3bb813e3d0a1.jpg)
4. 安裝Rclone<br>
輸入指令：curl https://rclone.org/install.sh | sudo bash
![183281890-7b9d27f2-4f9c-40d7-b0b0-0dca7efe7925-compressed](https://user-images.githubusercontent.com/106213982/183295804-cf7f568e-8b6b-45f1-8a18-8ba04630f2be.jpg)
5. 安裝完畢，查看Rclone版本，本文使用的版本為v1.59.0<br>
輸入指令：rclone version
![183281900-c5d92688-e05f-4b38-991f-b1504113c88a-compressed](https://user-images.githubusercontent.com/106213982/183295839-17325fa9-6a4e-4add-9543-2e792ee38c6e.jpg)

## 三. Rclone設定
1. 啟動Rclone設定<br>
輸入指令：rclone config<br>

n) New remote<br>
我們輸入 n，建立新的遠端設定<br>
n/s/q> n<br>

![183283192-18591747-a4a8-4641-a4b8-ffa8ebf38a3e-compressed](https://user-images.githubusercontent.com/106213982/183295936-021415b9-7e25-423a-bf6e-4827a6e830ab.jpg)
2. 輸入自訂名稱<br>
  
name> gd01<br>

![183282567-6133ad7e-fb4b-4eaf-8ab9-3a0cfbfac23b-compressed](https://user-images.githubusercontent.com/106213982/183295965-1b2219c1-0c54-4562-9a1d-04465d7bfefe.jpg)
3. Rclone v1.59 會列出49個選項，Google Drive在第18個選項，所以輸入18。如果因為版本差異，請根據文字選項選擇<br>

Storage> 18<br>

![183282586-5b6046cb-47d8-4c80-80e8-6027d6488eb7-compressed](https://user-images.githubusercontent.com/106213982/183296000-8ea0731b-2e4e-4e9f-9fc9-06f5d3b90339.jpg)

4.  client_id輸入預設值，保留空白直接按Enter。使用預設值，會使用內部提供的key，效能會比較差，如果在意的話，可以設定自己的clinet_id。<br>
參考連結:https://rclone.org/drive/#making-your-own-client-id
![183283302-8376ef12-d7d5-4721-a0a7-66ddfe204476-compressed](https://user-images.githubusercontent.com/106213982/183296031-98fcaf8f-f4e0-4ab8-97ab-b3d507d0cae9.jpg)

5. client_secret輸入預設值，保留空白直接按Enter。同上，如果在意效能可以參考連結自行設定。<br>
![183282631-36cf3c3a-a190-4c7e-9e19-2345219c30d6-compressed](https://user-images.githubusercontent.com/106213982/183296058-99f825a3-5914-42dd-b35f-3f037e47d300.jpg)

6. 選擇權限。<br>
如果設定的是來源Google Drive，建議選2，Read-only，這樣可以避免誤刪檔案。<br>
如果設定的是目的地Google Drive，建議選1，Full access，這樣可以確保寫入的權限。<br>
![183282646-e6db7727-8778-40fe-9890-d7aa4456a3b4-compressed](https://user-images.githubusercontent.com/106213982/183296089-28c6ae75-169a-4c9d-bffa-af624fd05e73.jpg)

7. service_account_file輸入預設值，保留空白直接按Enter。
![183282658-777762bc-91cb-4660-8105-d7d71ce5b356-compressed](https://user-images.githubusercontent.com/106213982/183296133-d1d92436-1a69-4ded-a95c-d9ab99e07b08.jpg)

8. 設定進階選項，輸入n<br>

Edit advanced config? (y/n)<br>
y) Yes<br>
n) No (default)<br>
y/n>n<br>

![183282672-117d0f41-6cca-409c-b537-ca33bb432d82-compressed](https://user-images.githubusercontent.com/106213982/183296248-a12da879-32a6-4523-9ef4-bd950e8dde36.jpg)

9. auto config 這邊選n，手動設定。這裡開始，也是我之前稍微卡關的地方，因此會寫詳細一點。<br>

Use auto config?<br>
* Say Y if not sure<br>
* Say N if you are working on a remote or headless machine<br>
y) Yes (default)<br>
n) No<br>
y/n> n<br>

![183282695-b25cbcff-7ace-4027-be44-53b157448e98-compressed](https://user-images.githubusercontent.com/106213982/183296278-03126f96-4166-4a03-a0da-1ddd8ee6b264.jpg)

10. 畫面要求我們輸入 config_token>，我們需透過另一台電腦取得token，這點後續說明。首先我們複製紅框框住的文字，找一個記事本存下來。<br>
    authorize "drive" "eyJz-------------------------"<br>
![183282740-0ea65c45-ef1d-4874-a8fb-2cadd51029ee-compressed](https://user-images.githubusercontent.com/106213982/183296319-fbdc3a5e-4627-495a-ba8c-9c1eef3e3d00.jpg)

11. 因為我們是透過雲端平台建立的VM，無法開啟瀏覽器，因此要讓Rclone可以取得Google Drive的授權，我們需要另外找一台可以開啟瀏覽器的電腦，可以是你的本機Windows作業系統，也可以是任何Linux作業系統。我們這邊將以Windows的操作為例。到Rclone官網(https://rclone.org/downloads/) 下載安裝檔，這邊選擇Window 64 Bit安裝檔。解壓縮後直接是可以執行的指令，不需要安裝，將目錄放在你想放置的位置。如下圖，透過瀏覽列，複製指令的路徑。
![183282762-1a4cab0f-47a9-4324-89e1-176e3acfc24f-compressed](https://user-images.githubusercontent.com/106213982/183296344-457ab632-8c68-49d4-a8bc-dcbf5d1b2603.jpg)

12. 接著於作業系統左下角搜尋"PowerShell"，開啟Windows PowerShell<br>
![183283399-d12f2d93-0310-4f03-a8f1-3d69999951f4-compressed](https://user-images.githubusercontent.com/106213982/183296380-5c3215d2-cffa-40fd-b8c2-1ab889a150ba.jpg)

13. 切換執行目錄，貼上之前複製的指令路徑(要改為自己的儲存路徑)。<br>
輸入指令：cd E:\tool\rclone-v1.59.0-windows-amd64\
![183282780-ba054d70-9526-4c7e-ac1a-42288d5ad507-compressed](https://user-images.githubusercontent.com/106213982/183296412-8db5573a-8ebc-4f7b-8809-0a654d8a3b8b.jpg)

14. 執行rclone.exe指令，並貼上之前儲存在記事本的授權金鑰。<br>
authorize "drive" "eyJz-------------------------"<br>
輸入指令：./rclone.exe authorize "drive" "eyJz-------------------------"<br>
![183282796-ee4723a1-ceee-4481-95fa-fc346acba208-compressed](https://user-images.githubusercontent.com/106213982/183296469-93133af8-f830-46c2-948b-71341b18049b.jpg)

15. 接著，瀏覽器會自動開啟，讓你進行Google Drive登入。
這邊注意，登入的帳號就是你要授權Rclone存取的Google Drive的帳號，這邊gd01是設定來源的Google Drive，所以就要登入來源的Google Drive。如果你要設定目的地的Google Drive，e.g. gd02，就要記得登入目的地的Google Drive。
![183282804-af6f9e7a-6034-4d5d-8798-f43f4fabe8e7-compressed](https://user-images.githubusercontent.com/106213982/183296506-ef9d6eb6-8569-451e-ab64-3093d766dd5b.jpg)

16. 接著，按"允許"，完成授權。
![183282831-aa86f83c-1bf8-4f69-9777-c5fb161bbea9-compressed](https://user-images.githubusercontent.com/106213982/183296535-90fa25d8-1203-4d3e-8d4a-7f3a41c4def5.jpg)

17. 網頁出現Success，顯示授權完成。
![183282836-dcd5e51c-fd09-45dd-9950-84278558ea61-compressed](https://user-images.githubusercontent.com/106213982/183296563-9c1d8427-c4a9-41c5-87e3-6101fa0dcd08.jpg)

18. 接著，回到PowerShell，紅框處產生的一連串英文，也就是我們在VM上的Rclone設定，所需要的config_token。我們將這串文字複製起來，貼到VM上。
![183282872-429ebd91-2197-4152-a702-3088a5792b64-compressed](https://user-images.githubusercontent.com/106213982/183296588-5b7c6bc1-96df-4948-87c1-931cb76b2b63.jpg)

19. 將字串貼到config_token>輸入處，按Enter<br>
![183282924-861a29b1-28ed-4fb9-af6a-74682891d432-compressed](https://user-images.githubusercontent.com/106213982/183296616-5ee6b632-dfbc-4b01-88d1-62c803e8e07e.jpg)

20. 接下來詢問是否設定 Team Drive(小組雲端硬碟)，我沒有使用，所以不設定，有需要可以參考原文。<br>
y/n> n
![183282961-f945e299-ebf0-4199-a399-66046f03abd3-compressed](https://user-images.githubusercontent.com/106213982/183296663-fb9a0f2f-d9f0-4a65-ae42-3480b7dfbd20.jpg)

21. 最後，確認設定訊息，都沒問題，選y，完成設定<br>
y/e/d> y
![183282994-6c928d5d-9ecf-42a8-917d-985a461e7ccc-compressed](https://user-images.githubusercontent.com/106213982/183296688-44cb79ac-b813-448f-b1cf-12777ef177be.jpg)

22. 接著，在畫面上可以看到gd01已經設定完成。
![183283023-91af3164-07d7-4a26-8747-22d5a045a418-compressed](https://user-images.githubusercontent.com/106213982/183296708-f52fb8ea-bd55-4c0f-af6b-e586d717d8a5.jpg)

23. 以上，我們完成的其中一個來源遠端硬碟gd01設定，依同樣方式，我們建立目的端的遠端硬碟gd02。建立gd02需要留意的是第6點，選擇權限的地方要選擇1，Full access，確保寫入權限，以及第15點，Rclone授權需登入的是目的地端Google Drive帳號。完成以後，如下圖。
![2022-8-8 上午 09-20-41 (2)](https://user-images.githubusercontent.com/106213982/183321343-5ca2ca35-22f0-4c38-8459-f021719f864f.png)

## 四. Rclone資料搬移教學
1. 在開始透過Rclone搬移資料前，我們先透過screen開啟多重視窗，輸入以下指令可以建立一個視窗，方便我們斷線後還可以連回來。輸入以下指令，會直接建立gd01togd02視窗，並自動連入<br>
輸入指令：screen -S gd01togd02<br>

![2022-8-8 上午 09-46-03](https://user-images.githubusercontent.com/106213982/183323402-ab68a164-8a46-422d-b200-240aba4ec2d5.png)

2. 進到gd01togd02視窗後，我們先測試gd01及gd02設定是否正確，看看能否連的上。以下兩個指令可以分別列出gd01及gd02的目錄，如果成功列出，就是設定正確。<br>
輸入指令1：rclone lsd gd01:/<br>
輸入指令2：rclone lsd gd02:/<br>
![2022-8-8 上午 09-51-08](https://user-images.githubusercontent.com/106213982/183323416-e212d648-09d9-4d73-a01a-15d7948924e7.png)

3. 接著我們可以開始進行資料搬移，因為本次的目的是要把gd01所有資料全部搬到gd02，因此在gd01及gd02的路徑都設定成"/"，這樣會把gd01完整目錄結構複製到gd02。後續會在說明針對特定目錄複製的相關指令。<br>
輸入指令：rclone -vv copy --drive-server-side-across-configs gd01:/ gd02:/<br>
參數說明：-vv 印出詳細執行過程，透過執行過程可以確認傳輸進度。<br>
參數說明：--drive-server-side-across-configs 在 Server 端傳輸檔案，不會使用到 Client 流量。<br>
![2022-8-8 上午 09-55-00](https://user-images.githubusercontent.com/106213982/183323421-ede2ae66-8e5d-4d91-bc4a-6b2e50496eb9.png)

4. 指令輸入完畢，就可以看到資料開始從gd01抄寫到gd02了。因為一切的操作都在Google雲端平台上的虛擬機器執行，所以不會耗用本機的空間及頻寬，資料不用先下載再上傳。接著就等它執行完畢即可，執行期間終端機瀏覽器可以關閉，電腦也可以關閉，等一段時間再連回來查看即可。下面說明連回來的方式。<br>

5. 透過GCP->控制台->Comuute Engine，可以依之前所述方式重新開啟瀏覽器終端機進入VM，輸入指令１，可以列出目前存在的視窗，下圖可以看到之前建立的視窗gd01togd02。輸入指令２可以連回gd01togd02視窗，就可以看到之前的執行結果，確認是否執行完畢。所以上述Rclone的指令一定要在gd01togd02視窗中執行，這樣可以確保之後可以連回來查看執行結果。<br>
輸入指令1：screen -ls<br>
輸入指令2：screen -x gd01togd02<br>
![2022-8-8 上午 10-01-38](https://user-images.githubusercontent.com/106213982/183323924-a833a193-af09-47d8-b3a4-3310c06c7c64.png)

6. 如果想先搬重要的資料夾而且資料夾有空格與非英文字的話，要用 "" 包起來。例如：<br>
   rclone -vv copy gd01:Music/"我愛聽的歌 第一集" gd02:Music/"我愛聽的歌 第一集"
7. 如果想設定Team Drive(小組雲端硬碟)，可以參考原文。基本上就是在設定Rclone第20點，選擇y，即可建立Team Drive遠端設定。













