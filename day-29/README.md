# 前言

專案建置的最後一哩路，就是讓整個應用程式上線，而在我們的專案小作品中，需要一個平台讓我們部署，並公開網站。Next.js 的母公司 Vercel 提供了一個非常方便的免費部署服務。使用 Vercel，我們可以透過幾個簡單的步驟，將網站順利地部署到網路上。Vercel 提供了友善的操作介面，能完美整合 Next.js 應用，並自動化大部分部署過程，確保網站上的內容總是最新的。每當我們更新或推送新代碼，它都會即時重新部署。此外，它還為網站提供了免費的 SSL 證書，確保網站安全性。

# 前置作業

在部署之前，我們必須先檢查有沒有頁面是已經建立路由，但可能基於階段性目標所以尚未完成內容，這時候可以統一以【網頁建置中】等樣式代替，這部分類似於先前前端錯誤處理中的範例，但我們不想讓用戶出現 404 頁面，可以做個預告性的頁面提示：

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073E3emevkD3T.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073E3emevkD3T.png)

再來就是先前 NextAuth.js 中申請 Facebook OAuth 時需要建立的【隱私權政策】、【服務政策】以及【資料刪除請求】頁面，也需要在部署前建立：

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073x9Yms7B1Na.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073x9Yms7B1Na.png)

# 註冊登入

在使用 Vercel 的前提是預設已經將專案建立在 git repositories 中，所以只需要以不同的代管平台登入，Vercel 會讀取 git repositories 中的專案，之後不論是用指令或平台進行部署，都會連結到你登入的帳戶中。

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073akdAZdGUQt.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073akdAZdGUQt.png)

# 開始部署

註冊登入完成後，非常快速的可以進入部署環節，目前有兩種方法提供部署

1. 使用 Vercel platform 部署
2. 在本機終端機通過指令部署

## 使用 Vercel platform 部署

在登入 [Vercel platform](https://vercel.com/new) 後，可以看到頁面上羅列了所有 git repositories 的專案，接下來點擊需要的專案，並點選 Import 後，進入下個介面

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073JCcKuyFXf3.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073JCcKuyFXf3.png)

這個介面中，可以設定專案名稱、執行指令以及環境變數等等，設定完成後點選 【Deploy】就會開始部署。

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073J4pqeAKxn0.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073J4pqeAKxn0.png)

此時頁面下方會顯示部署時的 Log 供我們查看，接著就靜靜等待部署結果。

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073hzJLdD1p91.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073hzJLdD1p91.png)

## 在本機終端機通過指令部署

在本機部署需要回到專案中，並於終端機執行以下指令：

### 安裝 Vercel CLI

```jsx
npm i -g vercel
```

### 部署至 Vercel

安裝完 Vercel CLI 後，可在終端機中執行 vercel 指令並開始部署步驟：

```jsx
vercel
```

執行後會進入一連串的設定，請依序完成

```jsx
? Set up and deploy "<專案路徑>"? Y
? Which scope do you want to deploy to? <你的 vercel 帳號>
? Link to exsiting project? N
? What’s the name of your existing project? <專案名稱>
? In which directory is your code located? ./
Auto-detected project settings (Next.js):
- Build Command: `next build` or `build` from `package.json`
- Output Directory: Next.js default
- Development Command: next dev --port $PORT
? Want to override the settings? [y/N] N
```

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073yb8ZhH2kgF.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073yb8ZhH2kgF.png)

接下來會開始進行部署，終端機中也會顯示一些訊息，也可以點選連結去監控部署的進度：

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073BOQggg8kam.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073BOQggg8kam.png)

如果部署成功則會出現這個狀態

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073A3WMtZDbCr.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073A3WMtZDbCr.png)

而 Vercel 的 platform 也會出現 Ready 的狀態

![https://ithelp.ithome.com.tw/upload/images/20231007/2015207302QJP4bUXV.png](https://ithelp.ithome.com.tw/upload/images/20231007/2015207302QJP4bUXV.png)

### 設定環境變數

以上是非常單純的狀態進行部署，而如果專案中有設定環境變數，則需要將環境變數設置好，Vercel 提供了 `vercel env add` 的指令讓開發者可以在終端機中直接執行：

```jsx
vercel env add <環境變數名稱>
? What's the value of <環境變數名稱>? <環境變數>
```

接著我們就可以一一的把在專案中 `.env` 的內容執行新增：

> 注意：當時申請 NextAuth.js OAuth 時，有設定 NEXTAUTH_URL 及 NEXTAUTH_SECRET ，在部署時不需要設定 NEXTAUTH_URL 變數，只需要設定 NEXTAUTH_SECRET ，並可以參考[這裡](https://generate-secret.vercel.app/32)自動生成隨機密鑰。
> 

```
DATABASE_URL="..."

GOOGLE_CLIENT_ID="..."
GOOGLE_SECRET="..."

FACEBOOK_CLIENT_ID="..."
FACEBOOK_CLIENT_SECRET="..."

GITHUB_ID='...'
GITHUB_SECRET='...'

TWITTER_ID="..."
TWITTER_SECRET="..."

NEXTAUTH_SECRET="secret"
NEXTAUTH_URL='http://127.0.0.1:3000'

NEXT_PUBLIC_CLOUDINARY_CLOUD_NAME="..."
```

如果是在部署後才加入環境變數，需要在新增後重新執行部署指令

```jsx
vercel
```

# 錯誤處理

上述的步驟如果順利的完成，非常恭喜的已經成功將產品上線，但！專案部署也有可能會失敗，例如本專案中使用了 Prisma 作為操作資料庫的工具，結果重新部署時就發生了 error：

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073faZ3memm7c.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073faZ3memm7c.png)

這個錯誤的出現是因為 Vercel 快取了你的專案依賴，直到其中一個依賴項發生變更。Prisma 通常在安裝依賴時，透過 postinstall 來生成 Prisma Client。但由於 Vercel 使用快取的模式，所以在初次部署後的後續部署中，這個 postinstall 不會被執行。導致 Prisma Client 與你的資料庫模式不再同步。

但莫寂寞慌張，我們只需要在 Build 時設定 generate 指令就可以解決：

以下步驟：

進入 [dashboard](https://vercel.com/dashboard) → 點選發生錯誤的專案 → 點選【settings】頁籤 → 點選【 General 】→ 在【 Build & Development Settings 】區塊 → 修改指令如下 → 儲存後重新部署

![https://ithelp.ithome.com.tw/upload/images/20231007/201520734tFryXRne5.png](https://ithelp.ithome.com.tw/upload/images/20231007/201520734tFryXRne5.png)

# 部署成功後

完成部署，出現 Ready 則部署成功，在頁面上方點選第二個 Bread Crumb 可以回到專案的主介面

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073uevc6MTEEX.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073uevc6MTEEX.png)

同時也可以在 Open Graph 中查看自己網頁的 meta tag 成效

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073BUx6gaMk95.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073BUx6gaMk95.png)

取得 Domain https://link-orchard.vercel.app/ ，可以將網站啟用了！

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073LlLNjGoSrX.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073LlLNjGoSrX.png)

# 結語

部署是每一個專案開發的終結，也是新的開始。透過 Vercel 我們可以輕鬆地將 Next.js 專案上線。儘管過程中可能會遇到一些挑戰，但只要耐心透過錯誤訊息進行修正，每一次的解決都是成長的契機。而 Vercel 大平台提供如此佛心的服務也讓開發者可以有更多發揮空間，雖然免費版有額度限制，但做為個人網站綽綽有餘阿！也期望每位開發者都能順利上線，分享創意，並在過程中繼續學習與進步。

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073RlvZnJgDgv.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073RlvZnJgDgv.png)

## 參考資料

https://next-auth.js.org/deployment

https://vercel.com/docs/cli/env

https://www.youtube.com/watch?v=W3jKJ3V_4V4&t=73s

https://ithelp.ithome.com.tw/articles/10276545

https://www.prisma.io/docs/guides/other/troubleshooting-orm/help-articles/vercel-caching-issue

https://github.com/vercel/next.js/issues/51026

![https://ithelp.ithome.com.tw/upload/images/20231007/20152073Ul5464wJPK.png](https://ithelp.ithome.com.tw/upload/images/20231007/20152073Ul5464wJPK.png)