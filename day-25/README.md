# 前言

在開發前端應用時，不良的錯誤處理可能導致使用者困惑、挫折，甚至選擇放棄使用你的應用。所以在專案中，我們透過 `try-catch` 語句捕捉潛在錯誤，並使用 `toast.error` 提供即時的使用者回饋。對於更嚴重的，可能導致整個頁面崩潰的錯誤，我們則採用 Next.js 的 `error.js` 及 `not-found.js` 進行集中管理，這樣不僅使頁面看起來更專業，也確保了使用者不會看到不必要的技術細節，提升使用者體驗。

今天的主題分為三個部分：

1. API 請求的錯誤處理
2. 404 錯誤處理
3. 未知的頁面錯誤處理

# API 請求的錯誤處理

是否還記得我們在後端錯誤處理實作時，遇到沒權限或驗證未通過的狀態都會回傳錯誤訊息以及狀態碼：

```jsx
new NextResponse('請輸入要修改的內容', { status: 400 })
new NextResponse('Unauthorized', { status: 401 })
new NextResponse('Server Error', { status: 500 })
```

這是後端回傳的訊息，那麼前端如何接收及呈現給使用者呢？

其實在前面的前端實作部分已經有出現在範例中，只是沒有詳細的說明。我們在 **`onSubmit`** 這類的請求功能中，通常都會用 **`try-catch`** 來抓取可能會出現的錯誤。這樣的設計是為了讓程式即便遇到問題也能繼續運作，並給予使用者明確的錯誤提示。

但很多時候會偷懶的以「更新失敗」四個字通知使用者，但這個情況變成使用者要通靈才知道哪個欄位出錯了，即使在有表單驗證的前提下，所以更準確的回傳錯誤訊息，會讓使用者更簡單的做修正。

```jsx
const onSubmit = async (values: FormValues) => {
    try {
			<---------請求邏輯---------->
      toast.success('更新成功')
    } catch (error) {
      toast.error('更新失敗')
    }
  }
```

首先我們必須判斷 catch 取得的 error 是什麼類型，因為它會先被定義為 any 的型別，而我們主要處理的有三種狀況：

1. 以 Axios 回傳的錯誤訊息，同時也包含我們在後端定義的回傳訊息
    - 以 `axios.isAxiosError` 進行判斷
2. 可以以 error.message 取得的 Error 訊息
3. 無法判斷的錯誤資訊

而上述的情況可能會需要比較複雜的判斷式，所以這邊做法是另外封裝成一個 utils，在所有請求處理中可以使用：

```jsx
// notifyError.tsx
const notifyError= (error) => {
  let errorMessage;

  // 判斷error是否為axios產生的錯誤
  if (axios.isAxiosError(error)) {
    errorMessage = error.response?.data || '請求錯誤';
  } else {
    errorMessage = error.message || '未知的錯誤';
  }

  toast.error(errorMessage);
};

export default notifyError;
```

建立完後即可在 API 請求時調用

```jsx
const onSubmit = async (values: FormValues) => {
    try {
			<---------請求邏輯---------->
      toast.success('更新成功')
    } catch (error) {
      notifyError(error)
    }
  }
```

# 404 錯誤處理

在前端頁面中，不免會發生使用者發出一個不存在路徑的請求，這時候預設都會出現一個 404 頁面：

![https://www.netlify.com/v3/img/blog/the404.png](https://www.netlify.com/v3/img/blog/the404.png)

在本專案中，尤其是前台頁面，是依據 username 來定義路徑產生頁面，如果使用者輸入一個不存在的頁面，一定會回傳 404。雖然預設的 404 頁面已經很簡約美，但第一，不符合我們專案的主題，第二，使用者無法進行轉跳或其他動作。

## not-found.js

所以這個錯誤處理我們可以使用 Next.js 提供的 `notFound()` 功能，只要在程式碼中執行這個函數，就會自動渲染 `not-found.js` ********這個檔案中的內容，接下來我們就先建立檔案的內容吧：

> `not-found.js` 本身預設是 Server Component，所以可以用 async await 取得所需的資料。
> 

而我希望在這個頁面，使用者可以跳回到首頁再進行其他操作，所以調用了 Link Component。

```jsx
// app/not-found.tsx
export default function NotFound() {
  return (
    <div className="...">
      <div className="...">ERROR 404</div>
      <h2 className=" text-5xl font-bold">Page not found</h2>
      <p className="...">Something went wrong, so this page is broken.</p>
      <div className="flex gap-4">
        <Link href="/" className="...">
          回到首頁
        </Link>
      </div>
    </div>
  );
}
```

## notFound()

建立好內容，接著我們可以在 page 的層級判斷是否可以取得輸入網址的資料，如果回傳 falsy 值則調用 `notFound()` ：

```jsx
const Page = async ({ params }: { params: { id: string } }) => {
	// 以 getUserByUsername 取的頁面資訊
  const user = await getUserByUsername(params.id)
	// 若無回傳資料直接執行 notFound()
  if (!user) {
    notFound()
  }

  return <PublicClient user={user} />
}

export default Page
```

## 畫面結果

最後當 `notFound()` 被調用時就會顯示以下內容：

![https://ithelp.ithome.com.tw/upload/images/20231003/20152073qzTvTVa0pt.png](https://ithelp.ithome.com.tw/upload/images/20231003/20152073qzTvTVa0pt.png)

# 未知的頁面錯誤處理

除了上面兩種錯誤，還有頁面可能會出現一些無法預知的錯誤，這時頁面會跳出 500 的系統錯誤：

![https://user-images.githubusercontent.com/42711013/86484940-f95cd000-bd89-11ea-820e-af5f6190b156.png](https://user-images.githubusercontent.com/42711013/86484940-f95cd000-bd89-11ea-820e-af5f6190b156.png)

但這個預設頁面一樣沒有太多操作功能或資訊，所以 Next.js 提供了一個 error.js 檔案，可以捕捉在 Server Component 或 Client Component 中出現的錯誤。當應用程式捕捉到錯誤時，會自動為 **`page.js`** 元件建立一個 React Error Boundary。若該範圍內出現錯誤，系統會使用從 **`error.js`** 導出的元件作為預備顯示。即使錯誤發生，上層的版面設計仍保持原狀並可互動。

![https://nextjs.org/_next/image?url=/docs/light/error-overview.png&w=1920&q=75&dpl=dpl_HEXnvq77Q9ge6cYtoLwfu56JRsVz](https://nextjs.org/_next/image?url=/docs/light/error-overview.png&w=1920&q=75&dpl=dpl_HEXnvq77Q9ge6cYtoLwfu56JRsVz)

## 注意事項

- `error.js` ****必須為 Client Component
- `app/error.js` boundary 不會捕捉 `app/layout.js` 或 `app/template.js` 中發生的錯誤，必須定義 `app/global-error.js` 檔案，這個 boundary 會包覆整個應用程式，所以也必須在其中定義 `<html>` 和 `<body>` 標籤。
- `error.js` ****包含兩個預設 props - error 及 reset
    - error ：有 message 和 digest 兩個屬性，`message` 可以取得錯誤訊息，`digest` 可以幫助開發者在伺服器的日誌中更容易地找到和該錯誤相關的詳細資訊
    - reset：為一個 function，調用後可以使出錯的頁面重新渲染

## 實作 error.js

在這個檔案中，首先必須定義為 Client Component，接著除了可以設定導向按鈕，也可以調用預設提供的 reset 進行重新渲染。

```jsx
'use client' 

import Link from 'next/link'

export default function Error({
  error,
  reset
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {

  return (
    <div className="...">
      <div className="..."> ERROR 505 </div>
      <h2 className="...">系統發生錯誤，請稍後再試</h2>
      <p className="..."> Something went wrong, so this page is broken. </p>
      <div className="flex gap-4">
        <Link href="/" className="..." >
          回到首頁
        </Link>
        <button onClick={() => reset()} className="..." >
          重新整理
        </button>
      </div>
    </div>
  )
}
```

## 畫面結果

![https://ithelp.ithome.com.tw/upload/images/20231003/20152073SBGkgz3CZB.png](https://ithelp.ithome.com.tw/upload/images/20231003/20152073SBGkgz3CZB.png)

# 結語

前端錯誤處理的核心還是以自訂義的樣式頁面呈現，實質上是為了讓使用者在遇到網頁錯誤時不會手足無措，或是胡亂操作。而當請求時適當的錯誤訊息也可以讓使用者更快速地修正操作。這樣的錯誤處理方式不僅提升了使用者的體驗，減少了他們的困惑和挫折，還有助於維護網站的專業形象，避免用戶對網站失去信心。本章的介紹為基本的設置，也可以考慮為不同的錯誤顯示更詳細的訊息頁面。

## 參考資料

https://nextjs.org/docs/app/building-your-application/routing/error-handling#how-errorjs-works

![https://ithelp.ithome.com.tw/upload/images/20231004/20152073NDj4jOYnaR.png](https://ithelp.ithome.com.tw/upload/images/20231004/20152073NDj4jOYnaR.png)