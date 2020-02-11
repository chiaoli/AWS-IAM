# AWS IAM
### IAM（Identity and Access Management）
在申請完 AWS 帳號後，筆者認為第一要務是建立一位管理員權限帳戶，概念有點像是當你在管理伺服器時都會盡量避免直接使用 Root 用戶，取而代之的是建立一位擁有管理者權限的帳戶，可透過 IAM 配置適當的權限給不同的使用者，降低使用者因擁有過多的權限而作出需多有風險的操作。
### IAM 上的名詞基本介紹:
* User(終端使用者)
可以將這此視為使用者登入帳戶
* Groups(群組)
可以建立一個特定的群組並設定一組權限給一群特定的使用者（如管理員權限等）
* Roles(角色)
可以透過角色賦予給 AWS 的服務與其他資源的存取權限並這定至該角色上，如可將角色設定權限為能夠存取 ECR 並將此角色賦予給 ECS 使用，如此一來 ECS 就能擁有存取 ECR 上的 Image 權限
* Policies(策略方針)
可以透過此方式來制定一組設定檔來定義一至多個權限，角色可直接選用這些策略方針
![](https://i.imgur.com/4eLwrse.png)
User 以及 Group 的概念如同一般作業系統中類似，可針對 User 分配到不同的 Group 之中則有不同的權限，而 Role 的概念有點像是預先設定好的權限，可直接將 Role 套用在 Group 或是 User 身上，當有新 User 加入時，就可直接針對 User 需求套用適當的 Role，而 Role 也可將其看待是一個或多個 Policy 所組成的集合體，透過選用這些 Policy 可直接將相對應的權限反映至 Role 身上。 
### 本節目標：
* 於 Root account 下建立新使用者（管理員權限）
* 建立帳戶別名（alias）
* 建立使用者多重驗證（MFA, Multi-Factor Authentication）
* 建立使用者存取金鑰

## 1.建立管理員使用者

首先透過 [AWS Console](https://console.aws.amazon.com/console/home) 登入已申請好的 AWS root accout

登入後選擇 **服務->IAM** 選項
![](https://i.imgur.com/JtHRX3N.png)

可新增一個或多個使用者名稱，在此以新增一位使用者為範例，而下方 AWS 存取類型有兩種類型可進行勾選 **使用程式設計方式存取** 以及 **AWS Management Console 存取**。
* **程式設計方式存取:** 系統會自動隨機產生出一組 Access Key ID 以及 Secret Access Keys 供使用在 AWS CLI 或是使用 AWS SDK 登入時做為登入密碼使用，須注意此 Key pair 只會在系統隨機產生當下顯示，之後無法透過查詢得知該 Secret Access Keys，若忘記 Secret Access Keys 時的處置方式只能特過刪除舊有 Key pair 並重新產生一組新的 Key pair。
* **AWS Management Console 存取:** 如同 Root account 一般，透過限縮並配置適當的權限，使用者可以登入 AWS Management Console 並進行操作。
![](https://i.imgur.com/hNlXYiI.png)
點選下一步後會進行使用者群組的設定，這邊我們選擇建立一個新的使用者群組。
![](https://i.imgur.com/SNi3jgU.png)
點選後可直接選擇 AWS 所提供的 AdministratorAccess 預設範本並為此群組命名為 Admin（可依個人需求喜好修改）作為管理者群組，當然也可以事後對這些使用者群組進行更進一步的修改。
![](https://i.imgur.com/9AIqnuP.png)
成功新增後需記下密碼欄位，請注意此頁面密碼皆為事後不可查詢，在此先暫時忽略存取金鑰的部分，後續會進一步說明。
![](https://i.imgur.com/saIHTcI.png)
在此要先稍微記住一下 IAM 首頁此處的帳戶 ID 或是複製該處網址。
![](https://i.imgur.com/V9ifx3I.png)
此時我們登出 Root account 並重新登入並於空格處輸入剛才的帳戶 ID。
![](https://i.imgur.com/cisI26J.png)
接著並輸入剛才所建立的使用者名稱及密碼，可以發現之後登入都需要輸入三種資訊:**帳戶ID**、**IAM 使用者**、**密碼**
![](https://i.imgur.com/LHj2fcl.png)
成功登入後系統會出現如下要求進行密碼修改，成功後就會以剛才所建立的使用者身份進入 console 畫面。
![](https://i.imgur.com/3Lvzq4H.png)
## 2.建立使用者別名（alias）
會發現若用 Root account 以外的使用者登入都會需要輸入帳戶 ID，並不方便於人類記憶，因此可自行建立一個帳戶別名，以後登入時可以改由輸入此別名進行。

回到 IAM 的主畫面，並點選畫面中 **自訂** 按鈕。
![](https://i.imgur.com/BOfhgoO.png)
可依照個人喜好輸入帳戶別名並點選建立，此後都入都可以此別名進行帳戶登入。
![](https://i.imgur.com/ae0K8hj.png)
## 3.建立使用者多重驗證（MFA, Multi-Factor Authentication）
回到 IAM 主畫面，看到下方安全狀態欄位的部分顯示驚嘆號示警，AWS 建議啟用 MFA 機制以更近一步保護帳號的安全。
點選 **管理MFA** 按鈕。
![](https://i.imgur.com/aT5FNGQ.png)
畫面會轉換到 AWS 設定頁面，在多重驗證區域點選 **指派 MFA 裝置** 按鈕。
![](https://i.imgur.com/Hs8JCUB.png)
在此處筆者是以虛擬 MFA 裝置為範例，相關的 Authenticator 應用程式可於行動裝置或電腦上安裝，如 Google authenticator、Microsoft authenticator 等，可以個人喜好選擇。
![](https://i.imgur.com/ck0ThZX.png)
可透過輸入密鑰或是掃描 QR code 將密鑰輸入至 Authennticator 之中，將 MFA 代碼輸入至下方，即可將雙方進行同步。
![](https://i.imgur.com/NIzioIM.png)
## 4.建立使用者存取金鑰
回到剛才 AWS 設定頁面，在存取金鑰的部分因建立帳號時我們先略過，因此並不知道此金鑰的密碼，而金鑰存取密碼是不可查詢的，只能夠重新設定。
如下圖，在操作的部分有兩個選項 **設為非作用中** 以及 **刪除**。
**設為非作用中**為暫時凍結這組金鑰，可在無操作需求時設為非作用中，即使知道此金鑰 ID 及密碼仍無法登入。
在這邊我們進行 **刪除** 操作並重新建立一組金鑰。只需點選 **建立存取金鑰** 按鈕即可。
![](https://i.imgur.com/df7Qta0.png)
畫面會顯示出存取金鑰的 ID 以及密碼。
![](https://i.imgur.com/EeCAOR1.png)
接下來我們將透過 AWS CLI 進行此金鑰的存取驗證。（需安裝 [AWS CLI](https://docs.aws.amazon.com/zh_tw/cli/latest/userguide/cli-chap-install.html)）
在此筆者以 MacOS 系統做範例。於家目錄下建立一個`.aws`資料夾
```sh
mkdir .aws 
cd .aws
```
進入後可使用指令或自行編輯 profile 檔案
以指令為例：
```sh
$ aws configure
AWS Access Key ID [None]: AKIAI44QH8DHBEXAMPLE
AWS Secret Access Key [None]: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: ap-northeast-1
Default output format [None]:
```
若加入 `--profile` 選項則可將資訊儲存於指定的 profile 之中，不輸入則為 default
```sh
$ aws configure --profile myUser
AWS Access Key ID [None]: AKIAI44QH8DHBEXAMPLE
AWS Secret Access Key [None]: je7MtGbClwBF/2Zp9Utk/h3yCo8nvbEXAMPLEKEY
Default region name [None]: ap-northeast-1
Default output format [None]: 
```
自行編輯：
針對檔案 `~/.aws/config` 進行編輯
```
[default]
region = ap-northeast-1
```
針對檔案 `~/.aws/credentials` 進行編輯
```
[default]
aws_access_key_id = <YOUR ACCESS KEY ID HERE>
aws_secret_access_key = <YOUR ACCESS KEY PASSWORD HERE>
```
可利用指令確認目前的設定組態```aws configure list```
```
$ aws configure list
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************YZOL shared-credentials-file
secret_key     ****************lO8u shared-credentials-file
    region           ap-northeast-1      config-file    ~/.aws/config
```
## Reference
[AWS Documentation](https://docs.aws.amazon.com/)
## License
MIT license

