![https://ithelp.ithome.com.tw/upload/images/20230917/20152073s66mUbHuRW.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073s66mUbHuRW.png)
今天營養師除了不開菜單，也不介紹保健食品或健康宣導，而是來專案介紹，實作作品是一個 Link in Bio Tool 的 0 到 1 開發，會依據專案背景、目的、功能、使用技術等幾個方向來做介紹。

# 專案背景

這個專案的啟發如同在引言中所說，本身有在經營社交平台，比較大眾的 Link in Bio 服務可能許多炫炮的功能都在花錢後才能解鎖，而小眾的服務可能時間一久了就不再提供服務或維護。正當苦惱的時候，發現自己轉職為網頁開發工程師，那怎麼不自己建置一個呢? 也趁這個機會驗收一下 Nextjs 的學習成果。

# 專案目的

前台讓訪客能夠瀏覽和點選用戶的個人連結，提供了直觀的用戶體驗。後台則讓用戶在登入後進行個人連結的客製化管理，方便地編排和更新連結。

# 主要功能

一開始規畫要參賽時，關於主要的功能需求思考了非常久

> 這個階段，我究竟要做到什麼程度的功能?
> 

我覺得這部分的拿捏必須考慮到兩個主要的重點: 時間與必要性

1. 時間 : 從規劃到成品上線只有兩個月的時間，一個人從設計到上線，我可以做的有多少?
2. 必要性 : 實作目的是做出成品寫出文章，所以第一階段，我需要注重的不是豐富性而是可用性，就不會是收集各連結的點擊分析，而是讓用戶可以建立連結並呈現在個人頁面。

經過以上兩點的思考，我規劃出以下前台及後台的初階段功能:

### 首頁

> 商品的入口
> 

![https://ithelp.ithome.com.tw/upload/images/20230917/20152073Nfgq1V02C2.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073Nfgq1V02C2.png)
1. 導覽列 - 依登入狀態顯示使用者圖像或登入按鈕
2. 標語
3. 產品圖示
4. 開始使用連結
5. 註冊連結

### 前台

> 用戶的個人頁面
> 

![https://ithelp.ithome.com.tw/upload/images/20230917/20152073GIP3YulvzV.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073GIP3YulvzV.png)
1. 頁面網址為該用戶的用戶名稱 ( username )
2. 顯示後臺設定的頭貼、用戶名、描述、所有設定的連結按鈕
3. 右上方需要一個 more 的按鈕
    1. 顯示連結
    2. 複製 bio link 的連結功能
    3. 登入或註冊即轉跳至後台

### 登入/註冊頁面

> 提供用戶登入註冊以及第三方登入功能
> 

登入及註冊頁面中有連結可以互相切換
![https://ithelp.ithome.com.tw/upload/images/20230917/20152073heWGWZCpmj.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073heWGWZCpmj.png)

### 後台

> 用戶管理所有設定的平台
> 
![https://ithelp.ithome.com.tw/upload/images/20230917/20152073RcvSyTNEJr.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073RcvSyTNEJr.png)
![https://ithelp.ithome.com.tw/upload/images/20230917/20152073BS6gfq7r4D.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073BS6gfq7r4D.png)
1. 基本設定：設定用戶頭貼、用戶名、頁面標題及個人描述
2. 連結設定：設定連結及網址，並提供拖曳改變順序的功能
3. 預覽功能：基本設定頁面及連結設定頁面共用一個預覽的元件
4. 分享按鈕：顯示連結並提供複製功能

# 使用套件技術

- Nextjs v13.4 - 實現全端應用框架
- TypeScript - 強型別語言，確保程式碼品質
- mongoDB - 非關係型資料庫，快速存儲和查詢資料
- Prisma - 資料庫ORM，方便地操作mongoDB
- TailwindCSS + Headless UI - 樣式框架，實現響應式和組件化UI
- NextAuth.js -用戶身份驗證，可實作 oAuth 設定
- Zustand - 局部或全局的狀態管理工具
- React-Beautiful-Dnd - 實現拖放互動套件
- React-Form-Hook - 表單管理，簡化表單建立
- Zod - 表單、請求資料驗證，以及確保輸入資料的正確性
- Cloudinary CDN - 圖片保存管理和優化服務
- React-Hot-Toast - 為 React 設計的通知套件
- React-Spinners - 用於資料載入時的等待動畫

# 結語

經過今天的專案介紹，希望有讓讀者稍微了解專案的目的以及視覺畫面。明天的文章會依據應用程式功能和使用的技術來介紹專案的架構以及操作流程圖，相信可以更深入了解這個 Side Project 樣貌~
![https://ithelp.ithome.com.tw/upload/images/20230917/20152073wOaLDnxACv.png](https://ithelp.ithome.com.tw/upload/images/20230917/20152073wOaLDnxACv.png)