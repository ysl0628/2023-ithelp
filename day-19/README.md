在前15篇章節中，已經詳細介紹了專案中使用的套件及在專案中的具體實作方式。接下來的文章將深入探討專案的細節和商業邏輯。考慮到這是一個從零開始建立的產品，我會按照設計到後端的實作順序逐步展開。本章節將從專案的起始介紹開始，包含互動設計以及用戶體驗(UX)的考量。這可以比喻為餐點製作過程中的「菜單設計」和「原料採購」階段。

# 設計工具及方法

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073qSDCGQMLZL.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073qSDCGQMLZL.png)

首先有了專案的構想後必須建立設計圖以方便日後的開發，由於以前的開發習慣再加上應用程式的大小，我選擇了 Figma 作為設計視覺的工具。

## 設計方法

從元件設計再到頁面布局構思，這些沒有經過時間的淬鍊以及專業的學習真的沒辦法在短時間內完成一個有系統的設計圖。那麼沒有太多設計知識背景的我們該如何自立自強呢？

> 使用 Figma 提供的社群力量！https://www.figma.com/community
> 

社群中有非常多模板、元件或是主題色票可以參考，而我的專案是以 Tailwind CSS 主題進行開發，所以搜尋關鍵字後，找到符合專案需求且功能俱全的模板。

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073zEKlyUMOJf.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073zEKlyUMOJf.png)

這些模板提供的元件及頁面真的非常豐富，接下來可以選擇喜歡的框架後，再取他們提供的元件來創建屬於自己的專案視覺。

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073DBTeSllFCJ.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073DBTeSllFCJ.png)

# 設計原則

在本專案中，除了採用模板所提供的元件外，也定義了獨特的主題色。我設定了灰階色、primary 和 secondary 色調作為專案的主要色彩。這些設計選擇的靈感來源於專案的名稱——「Link Orchard」，果園的形象~~( google 的結果)~~是滿滿的橘子，所以我選擇了綠色和橘色作為主題色。至於灰階色，則選擇了帶有一絲藍色的色調。除此之外，也定義了 danger (紅色調)、warning (黃色調)、info (藍色調) 三種類型的色調。

## 灰階色 (grey)

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073xhl4IES5R8.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073xhl4IES5R8.png)

## Primary (綠色調)

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073D9Xo7osbhr.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073D9Xo7osbhr.png)

## Secondary (橘色調)

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073P9b05mMemc.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073P9b05mMemc.png)

# 各頁面設計

在[第二天的專案介紹](https://ithelp.ithome.com.tw/articles/10320454)時，有稍微的介紹專案中所有的主要需求頁面，其中簡單說明了功能，這個段落將會再詳細的介紹各頁面的功能，以及操作流程。

> 回憶一下專案的頁面：前台、首頁、後台及登入註冊頁面
> 

## 導覽列

不論是首頁還是後台，我都設計了一個包含 Logo、登入按鈕（對於未登入用戶）、分享按鈕（對於已登入用戶）、 User Menu（對於已登入用戶）的NavBar。

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073CUuDJRGGix.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073CUuDJRGGix.png)
![https://ithelp.ithome.com.tw/upload/images/20231001/20152073o8nPKnzpob.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073o8nPKnzpob.png)

User Menu 在登入後顯示：

- username：辨識當前使用者用戶名
- 個人設定：可設定用戶名及查看用基本資料
- 開始我的 Link Orchard：轉跳至 Portal 頁面
- 聯絡我們
- Logout：登出後轉跳至首頁

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073RM1s74c1Kx.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073RM1s74c1Kx.png)

## 首頁

設計首頁時，我發現即使對於設定和顯示頁面充滿了靈感，首頁的設計仍然需要深思熟慮。畢竟，它是專案的第一印象。

關於首頁的初步概念，我希望它看起來既簡單又直觀，能快速傳達產品的核心價值。參考了蠻多商品的首頁設計，最常看到的是突出的產品視覺加上簡潔的描述，以及一些主要的操作按鈕。未來，當用戶滾動首頁時，我還計劃展示產品的詳細功能。

目前設計是進入首頁可以看到產品的宗旨及產品圖，接下來可以藉由「開始我的 Link Orchard」按鈕進入專案後台(如果未登入則需要先登入再自動轉跳到後台)，也可以點擊「聯絡我們」後轉跳到專案的 Link in Bio。

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073yNDW8n5OpX.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073yNDW8n5OpX.png)

## 登入註冊頁面

登入及註冊頁面則相對簡單，包含帳號密碼的方法及第三方登入的方法。並提供登入註冊頁面相互切換的連結。

### 註冊

- **註冊資訊**:
    - **username**: 必須是獨特的，不能與其他用戶重複。有輸入規則的提示 (tooltip)。
    - **email**: 必須是獨特的，不能與其他用戶重複。
    - **password**: 用戶需輸入密碼。輸入提示規則。
- **錯誤提示**:
    - 若 username 或 email 重複
    - 若 username 及 password 不符合規則
![https://ithelp.ithome.com.tw/upload/images/20231001/20152073QTIVm7sLBp.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073QTIVm7sLBp.png)

### 登入

- **登入資訊**:
    - **username**: 用戶需輸入其註冊時的 username。
    - **password**: 用戶需輸入對應的密碼。
- **錯誤提示**:
    - 若 username 或 password 資料不符
![https://ithelp.ithome.com.tw/upload/images/20231001/201520736b4TQ6bW5K.png](https://ithelp.ithome.com.tw/upload/images/20231001/201520736b4TQ6bW5K.png)

## 後台頁面

後台頁面包含 NavBar 、基本設定、連結設定及 User Menu 中的個人設定。基本設定及連結設定頁面也設置預覽畫面提供使用者在發布前檢視。

### 基本設定

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073JM0NtLcxEC.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073JM0NtLcxEC.png)

可以上傳前台的大頭貼、平台標題及敘述，以及目前提供三種主題色可以供選擇，此頁面可先預覽後再儲存設定。

- 大頭貼：上傳後會顯示在左方的人像中，在未儲存前都可以重置。
- 標題：為平台設定標題
- 描述：為平台設定描述，最大字數 300 字以內
- 主題色：目前提供基礎色、藍粉色及萊姆色選擇

    ![https://ithelp.ithome.com.tw/upload/images/20231001/201520734TAa2LA4ps.jpg](https://ithelp.ithome.com.tw/upload/images/20231001/201520734TAa2LA4ps.jpg)

### 連結設定

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073pEbs1hPMmL.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073pEbs1hPMmL.png)

可設定既有的社群連結，或是自定義連結，設定後可以修改也可以刪除，點選編輯順序，也可以使用拖曳調整順序，最多可以設定 8 個連結。連結若不符合 url 規則也會跳出錯誤訊息。

> 此頁面考慮到統一儲存連結不夠直觀，所以當新增後點下黃色勾勾就會更新前台資料。
> 
- 社群連結：下拉選單選擇社群網站，填寫網址 → 點選黃色勾勾
- 自訂義連結：填寫連結命名，輸入網址 → 點選黃色勾勾
    
   ![https://ithelp.ithome.com.tw/upload/images/20231001/20152073X238r4OnRy.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073X238r4OnRy.png)
    
- 編輯按鈕：點選後可進行編輯
- 編輯順序按鈕：點選後可調整連結順序
    
    ![https://ithelp.ithome.com.tw/upload/images/20231001/20152073PWmMT56wW5.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073PWmMT56wW5.png)
    

### 個人設定

個人設定頁面目前主要提供個人資訊，例如 email、名稱等個人資訊，以及修改 username 的功能。

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073pBO2bUOTXw.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073pBO2bUOTXw.png)

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073Ckl5b0wyk0.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073Ckl5b0wyk0.png)

## 前台頁面

前台頁面的呈現即為預覽頁面的內容，除此之外還有右上方「更多」的彈跳視窗。

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073ez0KEl76Ek.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073ez0KEl76Ek.png)

### 畫面

- 頭貼
- 標題
- 描述
- 建立的所有連結

### 更多

顯示該頁面的連結、複製連結按鈕以及登入按鈕(已登入則為進入後台的按鈕)

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073Qe7wqItsUz.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073Qe7wqItsUz.png)

# 結語

之前在工作上大部分都是藉由這些設計工具進行開發，這次 side project 則是自己從 0 甚至從 undefined 開始，設計的學問真的太多眉眉角角，更別說在真的實作後發現有很多地方太過於理想，所以回頭又修改及新增了許多需求，而且走的每一步都會影響下一步的決定。

或許寫完這篇的同時我又想到了可以增加 UX 的部分，設計圖又再次修改了?!

## 參考資料

https://www.tints.dev/blue/3B82F6
https://coolors.co/f0fdf4-dcfce7-bbf7d0-86efac-4ade80-22c55e-16a34a-15803d-166534-14532d

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073JOnU9cjW7m.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073JOnU9cjW7m.png)