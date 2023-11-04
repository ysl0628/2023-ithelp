![https://ithelp.ithome.com.tw/upload/images/20230918/20152073QyNWkSsOI7.png](https://ithelp.ithome.com.tw/upload/images/20230918/20152073QyNWkSsOI7.png)
我們人體的骨架是支撐我們人體重要的器官，它不僅提供結構，使我們能夠站立和行動還有保護內臟，我覺得專案架構就如同我們的骨架，如果沒有好好的建立，讓所有功能和模組都有其適當的位置，往後要擴充功能或維護都會有困難度。正如一個強健的骨架支持健康的身體，一個穩健的專案架構也是確保技術專案成功的關鍵。在這篇文章中，我會向各位深入探索我如何設計 Link in Bio Tool 專案的“骨架”，從資料夾結構、資料庫設計到整體的操作流程，讓我們開始這場解剖之旅。

# 資料夾結構

在資料夾結構設計的時候主要思考的依據是 

> Nextjs 的 filed base route 再去延伸我需要什麼頁面?
> 

在 Nextjs 官方文件的[專案組織策略](https://nextjs.org/docs/app/building-your-application/routing/colocation#project-organization-strategies)中其實有建議三種結構：

1. 將專案的資料夾設定於 app 之外
    
    ```jsx
    ├── my-project
    │   ├── actions
    │   ├── app
    │   ├── components
    │   ├── constants
    │   ├── hooks
    │   ├── libs
    ```
    
2. 將專案的資料夾設定於 app 第一層
    
    ```jsx
    ├── my-project
    │   ├── app
    │       ├── components
    │       ├── actions
    │       ├── components
    │       ├── constants
    │       ├── hooks
    │       ├── libs
    ```
    
3. 在各個頁面設定各頁面所需的資料夾
    
    ```jsx
    ├── my-project
    │   ├── app
    │       ├── portal
    │       │   ├── page.tsx
    │       │   ├── components
    │       │   └── actions
    │       └── home
    │           ├── page.tsx
    │           ├── components
    │           └── actions
    ```
    

> 但由於 app 資料夾中會越生越多路徑檔案及其他功能性檔案，我自己在查找的時候有時候會覺得眼花撩亂，所以最後選擇了第一種方式，
> 

以下的資料夾樹狀圖中，將資料夾分為**核心資料夾**及**功能資料夾**

## 核心資料夾

> `app`、`components`、`actions`
> 
- `app` - 專心於設定所有與路徑相關的資料夾及檔案
- `components`  - 共用元件，如果是頁面自己的元件則建立在 `component/page/[頁面名稱]` 中
- `actions` - 各個調用資料庫 data 的 function

## 功能資料夾

> `constants`、`hooks`、`libs`、`prisma`、`providers`、`types`
> 
- `constants` - 常數管理的資料夾
- `hooks` - 包含 custom hook 及 zustand 建立的資料夾
- `libs` `prisma` - prisma 相關設定的資料夾
- `providers` - 狀態管理或第三方套件所設定的 provider
- `types` - 型別管理資料夾

```bash
├── actions
├── app
│   ├── (portal)
│   │   ├── default.tsx
│   │   └── portal
│   │       ├── basic
│   │       │   └── page.tsx
│   │       ├── layout.tsx
│   │       └── links
│   │           └── page.tsx
│   ├── (public)
│   │   ├── (routes)
│   │   │   └── [id]
│   │   │       └── page.tsx
│   │   └── layout.tsx
│   ├── (root)
│   │   └── page.tsx
│   ├── @auth
│   │   ├── (.)login
│   │   │   └── page.tsx
│   │   ├── (.)signup
│   │   │   └── page.tsx
│   │   └── default.tsx
│   ├── api
│   │   ├── auth
│   │   │   └── [...nextauth]
│   │   │       └── route.ts
│   │   ├── register
│   │   │   └── route.ts
│   │   └── user
│   │       └── [id]
│   │           ├── links
│   │           │   ├── [linkId]
│   │           │   │   └── route.ts
│   │           │   └── route.ts
│   │           └── route.ts
│   ├── default.tsx
│   ├── favicon.ico
│   ├── layout.tsx
│   ├── login
│   │   └── page.tsx
│   └── signup
│       └── page.tsx
├── components
│   ├── dialog
│   ├── input
│   ├── navbar
│   └── page
│       ├── auth
│       ├── home
│       ├── portal
│       │   ├── BasicSetup.tsx
│       │   ├── LinksSetup
│       └── public
├── constants
├── hooks
├── libs
├── prisma
├── providers
├── public
│   ├── images
├── styles
├──	types
├── middleware.ts
├── next-env.d.ts
├── next.config.js
├── package-lock.json
├── package.json
├── postcss.config.js
├── tailwind.config.js
└── tsconfig.json
```

# 資料庫結構

Bio Link 的專案所規劃的資料庫結構並不複雜，主要的資料存取放置於 User、Account 及 Link Model

之後的文章會再詳細介紹各欄位的說明，下方先簡要的提供資料表之間的關聯，詳細欄位敬請期待!

- `User` 主要儲存使用者的相關資訊，包含客製化設定
- `Account` 是存儲與第三方服務提供商或授權系統相關的資訊
- `Link` 是存儲與用戶相關的網頁連結
- `Account` 和 `Link` 與 `User` 是一對多的關係，並且以 userId 作為關聯的依據

![https://ithelp.ithome.com.tw/upload/images/20230918/201520734Nuo9bOW3b.png](https://ithelp.ithome.com.tw/upload/images/20230918/201520734Nuo9bOW3b.png)

# 流程圖(Roadmap)

接著想要介紹關於 Link in Bio Tool 這個應用程式的操作流程，請各位讀者搭配著圖來一趟操作旅程！

應用程式主要分為兩個入口: 首頁、網站前台

首頁是應用程式的介紹，網站前台則是所有訪客皆可參閱的介面，兩者皆可以經由登入驗證後進入後台管理，進入後台後可經由基本設定及連結設定為網站前台的內容做設定，這個頁面中有預覽可以檢視，並且在設定儲存後同步於網站前台～

![https://ithelp.ithome.com.tw/upload/images/20230918/20152073lPPw9N8uLL.png](https://ithelp.ithome.com.tw/upload/images/20230918/20152073lPPw9N8uLL.png)
# 結語

透過這篇分享，希望讀者們能夠了解到這個專案是如何組織的，以及功能操做概念。自己在設計資料夾結構時，也沒辦法一次完成或是完全不更改結構細節，隨著需求的增加會有更多不可預期的更動，但設定好大方向後，未來沿著設定的模式擴展思緒也會清晰許多。

前三篇介紹完專案的概要後，接下來幾天將會為各位介紹我在專案中所使用的工具，開發生態各式各樣套件都有，除了說明為什麼要使用這些工具外，也會依據我在專案中使用的場景向各位展示，明天就從那個支撐我整個作品以及這篇系列文的 Next.js 開始吧！

![https://ithelp.ithome.com.tw/upload/images/20230918/20152073DuOmEq8AxR.png](https://ithelp.ithome.com.tw/upload/images/20230918/20152073DuOmEq8AxR.png)