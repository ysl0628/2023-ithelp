![https://ithelp.ithome.com.tw/upload/images/20230919/201520733Ze7Z94ZVP.png](https://ithelp.ithome.com.tw/upload/images/20230919/201520733Ze7Z94ZVP.png)

說到為什麼要用 Nextjs 來做主要的開發框架，我覺得可以透過我的產品目的及我的開發模式來決定:

> 1. 我的專案是個公開性的網頁，並且希望用戶可以透過我的產品提升個人連結的點閱
> 2. 我需要進行全端開發，而且我是 React 使用者
> 

基於上面這兩點我希望可以有 Server side render 來加速我的網頁首次載入時間以提升 SEO，也希望可以有前後端整合並且以 React 友善的功能，更不用說擁有非常值觀的 file base route 以及新版提供的 server component 功能，對於一條龍開發的使用者來說，Nextjs 是一個毫無疑問的選擇，就像綜合維生素，不用一堆瓶瓶罐罐，一顆就可以解決我基本的需求。

另外還有很重要的：官方文件比小說還好看

今天的內容不會過多地探討 Next.js 的原理，而是基於 Next.js 的基礎特性，分享我在專案中的實作經驗。我也會提到在開發新功能時所遇到的問題和挑戰。讓我們直接開始！

# Pages and Layouts
Nextjs 13 的路徑改為 `app` 目錄，依舊可以跟 `pages` 共用，在習慣新的路徑形式時也可以在舊的目錄中使用，但要注意 app route 和 pages route 避免使用相同的 URL path，以防在 build 時出現錯誤，並且一個路徑片段必須由一個 `page.tsx` 檔案建立。

> ⚠️ app 目錄下的檔案預設為 `Server Components` ，使用者也可以將檔案設定為 `Client Components`
> 

## 路徑怎麼設置？

> 起手式：`app === ‘/’` ，URL path 就是資料夾路徑去除 app 的 path
> 

```
http://my-domain.com/ → 顯示 app/page.tsx 中的內容
```

### 範例

下方依據本專案需要的路徑：

- /login  → 顯示 `app/login/page.tsx` 中的內容
- /portal/basic → 顯示 `app/portal/basic/page.tsx` 中的內容
- /portal/links → 顯示 `app/portal/links/page.tsx` 中的內容

> ⚠️ 注意：可以留意路徑中 `portal` 若沒有頁面可以不用在資料夾中設置 `page.tsx` 


### 檔案格式

除了 page.js 之外，Nextjs 還有提供許多檔案功能可以顯示不同狀態內容或設置共用的 component UI

| Folder | File | Means | 是否可在各路徑資料夾中設置 |
| --- | --- | --- | --- |
| app | page.js | `http://<my-domain>.com/` 頁面的內容 | 可 |
|  | layout.js | 建立區段路徑或是他的 child component 共用的 UI  | 可，但 head 只可以在 Root Layout 設置 |
|  | template.js | 類似 layout.js |  |
|  | loading.js | loading UI，在 React Suspense Boundary 中包裝 | 可 |
|  | error.js | error UI，在 React Error Boundary 中包裝 | 可 |
|  | not-found.js | 當 notFound() 被調用時所顯示的 UI | 可 |
|  | favicon.ico | 網站代表 icon 圖示 | 否 |
| app/api | route.js | 建立該路徑的 api 端點 |  |

## Root Layout
在 app 目錄的最高層級中被定義，且目錄中至少含有一個 root layout，這個 layout 也會被所有的路由 `page.js` 引用

### 功能

- 設定 SEO 相關屬性，如： `<html>` 和 `<body>` 標籤，以及 `metadata` 資訊
- 設定共用元件，例如 NavBar、SideBar、Provider 等等

> ⚠️ 注意
> 1.  root layout 中必須定義 `<html>` 和 `<body>` 標籤，也只有 root layout 可以設定這兩個標籤
> 2. 預設且**必須**為 Server Components，不可設定為 Client Component
> 

### 範例

```
// app/layout.tsx

export const metadata: Metadata = {
  title: 'Link Orchard',
  description: 'Branch Out with LinkOrchard'
}

export default function RootLayout({ children }: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en">
      <body>
		<Providers>
          <NavBar/>
          <main className="h-screen flex justify-center ">
            {children}
          </main>
        </Providers>
      </body>
    </html>
  );
}
```

## Layouts
資料夾下的頁面中共用的 UI 組件，頁面除了顯示此 Layout 內容，也會顯示 Root Layout 設置的共用元件。如果各路徑資料夾下沒有特定的 layout，則預設使用 Root Layout。

![https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fnested-layouts-ui.png&w=1920&q=75&dpl=dpl_2PHSwT98ft8Jg2vp5hMS9ojinrcz](https://nextjs.org/_next/image?url=%2Fdocs%2Flight%2Fnested-layouts-ui.png&w=1920&q=75&dpl=dpl_2PHSwT98ft8Jg2vp5hMS9ojinrcz)

> ⚠️ 注意
> 1. 預設為 Server Components，但也可以設定為 Client Component
> 2. 在 `layout.js` 檔案中即可 fetch data
> 3. 可額外設置專屬的 `metadata`
> 

### 範例

```
// app/portal/layout.tsx
export const metadata: Metadata = {
  title: 'Portal page',
  description: 'Branch Out with LinkOrchard'
}

export default function PortalLayout({
  children, 
}: {
  children: React.ReactNode,
}) {
  return (
    <section>
      {children}
    </section>
  );
}
```

# Route Group - `()`
在開發的時候可能會因為網頁的分布目的，或是有不同團隊進行開發，當所有路徑都攤在 app 資料夾的時候，當路徑一多會眼花撩亂很難管理，想要分類卻不能直接歸類資料夾，於是 Nextjs 提供了一個不會影響 URL path 的資料夾檔名格式，讓開發者更好的管理結構。

> 可以將資料夾名稱以括號包覆來建立 route group：`(folderName)`，在括號中的資料夾名稱會在 URL 中被省略
> 

### 範例

在專案中，我將功能頁面分為首頁、前台、後台，於是我也將路徑分類為三個檔案，這樣的模式除了讓我方便管理，也可以讓我在各自的 Group 設定該功能專屬的共用元件。

除此之外 group 內也可以再包覆 group 資料夾，一樣也不會影響 URL 結構!

![https://ithelp.ithome.com.tw/upload/images/20230919/20152073MBLJ5KL9hr.png](https://ithelp.ithome.com.tw/upload/images/20230919/20152073MBLJ5KL9hr.png)
![https://ithelp.ithome.com.tw/upload/images/20230919/201520735K1FRrTMhP.png](https://ithelp.ithome.com.tw/upload/images/20230919/201520735K1FRrTMhP.png)
## Layout 設置

在 Route Group 的結構下也可以依據使用請況來設置各 Group 的 Layout

### 1. 在不影響 URL 結構的情況下組織 route

> 共用 `RootLayout`，下圖中 `(marketing)` 和 `(shop)` 被省略
> 

![https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568349/nextjs-docs/darkmode/route-group-organisation.png](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568349/nextjs-docs/darkmode/route-group-organisation.png)

> 共用 app 底下 `layout.js` 也顯示各自的 `layout.js`
> 

 `(marketing)` 和 `(shop)` ，共用了 app 中的 `layout.js`，也可以在各個 Group 中設定各自的 layout

![https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568349/nextjs-docs/darkmode/route-group-multiple-layouts.png](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568349/nextjs-docs/darkmode/route-group-multiple-layouts.png)

### 2. 選擇將特定 route 區段加入 layout

可以將相同群組的 routes 獨自建立自己的 layout，在群組外的資料夾或檔案就不會使用到那個群組中的 layout

> 例如下圖中 `(shop)` 自成一個群組，使用自己群組中的 layout，而群組外的 `checkout` 資料夾就不會顯示 > > `(shop)` 中的 layout
> 

![https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568350/nextjs-docs/darkmode/route-group-opt-in-layouts.png](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568350/nextjs-docs/darkmode/route-group-opt-in-layouts.png)

### 3. 建立多重 Root layouts 來切分應用程式

在 RootLayout 的段落中有介紹到每個 app 目錄下都必須設置一個 Root Layout ，但是使用 Route Group 的模式下，可以將 app 中最高層的 `layout.js` 刪除，並且必須在各自的群組中建立自己的 `layout.js` ，而 `<html>` 和 `<body>` 也必須在各自的 Root Layout  中設定

![https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568348/nextjs-docs/darkmode/route-group-multiple-root-layouts.png](https://assets.vercel.com/image/upload/f_auto,q_100,w_1600/v1666568348/nextjs-docs/darkmode/route-group-multiple-root-layouts.png)

> ⚠️ 注意
> 1. route groups 裡的檔案路徑名稱不可以重複
> `(marketing)/about/page.js` 和 `(shop)/about/page.js` 皆會指向 `/about` 並且發生錯誤
> 2. 使用多重 root layouts 時，轉跳至由不同 layout 包覆的路徑時，皆會引發整個頁面的載入
> 

# 結語

在 Next.js 的第一篇中，我按照官網文件的順序介紹了基礎的路由設定、共用的 Layout 設計，以及我認為非常實用的 Route Group 功能。這也是我首次在 Next.js 專案中使用此功能，確實使得資料夾結構更加清晰整齊。明天我將繼續深入探討 Route 設定，進一步了解 Next.js 這個強大框架的其他特點和功能！

![https://ithelp.ithome.com.tw/upload/images/20230919/20152073Uw80vpZZ4B.png](https://ithelp.ithome.com.tw/upload/images/20230919/20152073Uw80vpZZ4B.png)

## 參考資料
https://nextjs.org/docs/app/building-your-application/routing