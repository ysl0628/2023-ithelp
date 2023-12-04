# 營養師不開菜單的第七天 - Next.js 13 Server Component 初體驗

![https://ithelp.ithome.com.tw/upload/images/20230922/20152073BQOmYJmKp8.png](https://ithelp.ithome.com.tw/upload/images/20230922/20152073BQOmYJmKp8.png)
在營養教育的時候常會介紹整個餐點製作的過程，稱為「農場到餐桌」，我們的 code 就像食材，確定了網站的基礎結構和用戶體驗；而 Rendering 則像烹飪技巧，決定了頁面的呈現方式和效能。

在 13 版更新後使用以 React Server Component 與 Client Component 切分作為宣傳招牌，以下來簡單的了解 Server Component 與 Client Component 在專案中的使用模式，以及所謂的靜態渲染和動態渲染與 SSR 那些渲染有什麼關係?

# Server Component 的特徵

![https://ithelp.ithome.com.tw/upload/images/20230922/20152073iUSGcL4JBV.png](https://ithelp.ithome.com.tw/upload/images/20230922/20152073iUSGcL4JBV.png)

1. 在 Server component 中的程式碼只會在 Server 端執行：可以直接在這些組件中存取資料庫或進行其他 Server 專屬的操作。
2. 當 Client 需要渲染 Server component 時，會發出一個請求，並傳遞必要的 props 到 Server 端。
3. React 在運行時會從伺服器收到分段的 JSON 回應，即所謂的「Chunked Response」
4. React 則會根據這個回應內容來 render 對應的 Server component 跟 Client components
5. 如果是 Server component 就將 react element commit 給 DOM
6. 大幅降低 bundle size 並提升效能。

![https://ithelp.ithome.com.tw/upload/images/20230922/20152073HLXzftQVHu.png](https://ithelp.ithome.com.tw/upload/images/20230922/20152073HLXzftQVHu.png)

# Client Component 的特徵

> 在檔案第一行，所有 import 之前標註 `"use client"`
> 

```
'use client';
import useRouter from 'next/navigation'

export default function ClientComponent() {
	const router = useRouter()

  return (
    <div>
        <button onClick={() => router.push('/login')}>Login</button>
    </div>
  );
}
```

# Server Component 與 Client Component

在 app 目錄中，所有 component 預設都是 Server Component，而 Server 及 Client Component 可以共存，讓開發者選擇這個 component 想要渲染的環境。

> 在 Server Component 中的 component 可以保證**只在 server side render**
在 Client Component 中的 component 雖然主要在 client render，但 Next.js 中可以在 server side 預渲染，再於 client hydrate，主要使用 `useSWR` 或直接將 server component `fetch` 到的 data 傳給 client component
> 

## 如何分辨誰是 Server 誰是 Client

在程式碼中我們可以用 `“use client”` 來分辨是否為 Client Component，那麼在網頁上我們如何分辨呢？最重要的方法也是 Server Component 最大的優點：Server Component 不會包含在 JavaScript bundle 中，因此也減少了 bundle 的大小，減少前端網頁的載入量，從而提升效能。

![https://i.imgur.com/vru2LcM.gif](https://i.imgur.com/vru2LcM.gif)

## 兩者如何共存？

- Server Component 中可以引入 Client Component
    
    ```
    // app/layout.tsx
    
    import ServerComponent from './ServerComponent'
    import ClientComponent from './ClientComponent'
     
    // Layout 預設就是 Server Component
    export default function Layout({ children }: { children: React.ReactNode }) {
      return (
        <>
          <nav>
            <ClientComponent />
            <ServerComponent />
          </nav>
          <main>{children}</main>
        </>
      )
    }
    ```
    
- 從 Server Component 傳遞**序列化**的 props 給 Client Component
    
    JavaScript 中，指的是將資料結構轉換為 JSON 格式的 string (JSON.stringify)，以下幾個可能不小心就用到的非序列化格式：
    
    1. **Function**：例如 `function() { return 1; }` 不能被序列化為 JSON。
    2. **`undefined`**：JSON不支持 `undefined`。
    3. **特殊的物件**：如 `RegExp`, `Map`, `Set`, `Date`，這些在轉為 JSON 時不會保留其原始結構和方法。
    4. 引用於自身 reference 的物件或陣列。
    5. **原型鏈上的屬性**：僅物件自身的屬性會被序列化，原型鏈上的不會。
    6. **`Error`物件**：雖然能夠序列化，但它的某些屬性（如 `stack`）可能不會完整地被序列化。
- Client Component 中不可以直接引用 Server Component
    
    ```
    // ❌ 會出現錯誤
    'use client';
    
    import useRouter from 'next/navigation'
    import ServerComponent from './ServerComponent'
    
    export default function Counter() {
    	const router = useRouter()
    
      return (
        <div>
    		<button onClick={() => router.push('/login')}>Login</button>
    		<ServerComponent/>
        </div>
      );
    }
    ```
    
- Server Component 可以作為 Client Component 的 children
    
    ```
    // app/ClientComponent.js
    
    'use client';
    import useRouter from 'next/navigation'
    
    export default function ClientComponent({children}) {
      const router = useRouter()
    
      return (
        <div>
          <button onClick={() => router.push('/login')}>Login</button>
          {children}
        </div>
      );
    }
    ```
    
    ```
    // app/page.js
    
    import ClientComponent from "./ClientComponent";
    import ServerComponent from "./ServerComponent";
    
    // Pages 預設是 Server Component
    export default function Page() {
      return (
        <ClientComponent>
          <ServerComponent />
        </ClientComponent>
      );
    }
    ```
    
- 第三方套件若有預設 client-only 的功能，可先封裝為 Client Component 後再引進 Server
    
    ```
    'use client'
 
    import { SessionProvider } from "next-auth/react"
 
    export default SessionProvider
    ```
    
- Provider 的實作方法：將所有 provider 封裝為一個 Provider component，再於 Root Layout 調用
    
    ```
    // providers/Providers.tsx
    
    'use client'
    
    import React, { FC } from 'react'
    import { ThemeProvider } from 'next-themes'
    
    import { type ThemeProviderProps } from 'next-themes/dist/types'
    import ToasterProvider from './ToastProvider'
    
    const Providers: FC<ThemeProviderProps> = ({ children, ...props }) => {
      return (
        <ThemeProvider {...props}>
          <ToasterProvider />
          {children}
        </ThemeProvider>
      )
    }
    
    export default Providers
    ```
    
    ```
    // app/layout.ts
    export default async function RootLayout({ children, auth }: RootLayoutProps) {
      return (
        <html lang="en" suppressHydrationWarning>
          <body className={inter.className}>
            <Providers attribute="class" enableSystem>
              {children}
            </Providers>
          </body>
        </html>
      )
    }
    ```
    

## 如何判斷什麼時候要加上 'use client'

基本上有用戶與網頁的交互作用就會視為 Client Component

| 情境 | Server Component | Client Component |
| --- | --- | --- |
| 獲取數據 | ✅ | ❌ |
| 直接訪問後端資源 | ✅ | ❌ |
| 將敏感信息 (access tokens, API keys, etc) 保存在 server 上 | ✅ | ❌ |
| 將大型依賴項保留在服務器上 / 減少 client 端 JavaScript | ✅ | ❌ |
| 使用互動性及監聽事件(onClick(), onChange(), etc) | ❌ | ✅ |
| 使用 State 和生命週期 Effects (useState(), useReducer(), useEffect(), etc) | ❌ | ✅ |
| 使用 browser-only APIs ( localStorage, XMLHttpRequest, etc) | ❌ | ✅ |
| 使用依賴於 state、effects 或 browser-only APIs 的自定義 hooks | ❌ | ✅ |
| 使用 React Class components | ❌ | ✅ |

# Static rendering 與 dynamic rendering

在 13 版本中可以發現各處都看不見 SSR / SSG / ISR 這些渲染的介紹，取而代之的是 **Static rendering 與 dynamic rendering** ，他們之間有什麼關聯，各自又是什麼？下方帶各位一探究竟。

## 靜態渲染

> 當資料請求有被暫存且未 **Dynamic Functions** 才會實現靜態渲染，Nextjs 預設為靜態渲染
> 

Server 及 Client Component 會在 **build time** 做預渲染，預渲染的結果會被暫存並在後續的請求中重用，而暫存的結果可以被重新生效(revalidated)

## 動態渲染

> 有使用 **Dynamic Functions 或 Dynamic Fetches** 時實現動態渲染
> 

Server 及 Client Component 會在 **request time** 被渲染，渲染的內容不會被暫存

## 什麼是 Dynamic Functions ?

1. 在 Server Component 中使用 `cookies()` 或 `headers()` 
2. 在 Client Component 中使用 `useSearchParams()` 
    
    > [官方建議](https://nextjs.org/docs/app/api-reference/functions/use-search-params#static-rendering)在靜態渲染的 route 中，如果有子組件使用了 `useSearchParams()` 會影響其他相近的所有 client component 渲染模式，建議以 `<Suspense/>` 包覆，可以隔絕該 component，使其他 components 依舊為靜態渲染
    > 
3. page.js 頁面有取得  `searchParams` - Server Component 及 Client Component 皆可以取得
    
    ```
    export default function Page({
      params,
      searchParams,
    }: {
      params: { slug: string }
      searchParams: { [key: string]: string | string[] | undefined }
    }
    ) {
      return (
        <div>
          // 內容
        </div>
      );
    } 
    ```
    

## 什麼是 Dynamic Fetches ?

執行 `fetch()` 請求時，請求的 Data 沒有 cache 的行為會被視為 **Dynamic Fetches** ，在 13 版不再使用 `get…Props()` 的方式進行資料的請求，取而代之的是 `fetch()` api 並且在所有的 component 都可以調用，不再只限定於 Page component，下方條列出的情況 data 不會被 cache：

1. `fetch('https://...', { cache: 'no-store' })`
2. 單獨調用 `fetch(https://..., { next: { revalidate: 0 } })` 請求時
3. 當 fetch 請求位於使用 POST 方法的 Route handler 中時：因為 Route handler 不在 React component tree 之中
    
    ```
    import { NextResponse } from 'next/server'
     
    export async function POST(request: Request) {
      const data = await fetch('https://...')
      const res = await request.json()
      return NextResponse.json({ res })
    }
    ```
    
4. 當使用 headers 或 cookies 之後進行 fetch 請求時
    
    ```
    import { headers } from 'next/headers'
     
    async function getUser() {
      const headersInstance = headers()
      const authorization = headersInstance.get('authorization')
      const res = await fetch('...', {
        headers: { authorization },
      })
      return res.json()
    }
    ```
    
5. 當使用 `const dynamic = 'force-dynamic'` 時
6. `export const fetchCache = 'default-no-store’`
7. 當 fetch 請求使用 Authorization 或 Cookie headers，且在組件樹中它上面有一個未被緩存的請求時。

## 跟 SSR / SSG / ISR 有什麼關係?

如果以 page route 的概念來看靜態渲染及動態渲染，就會是下面三種組合

- Static fetch + static render = SSG (`getStaticProps`)
- Revalidate fetch + static render = ISR (`getStaticProps with revalidate props`)
- Dynamic fetch + dynamic render = SSR (`getServerSideProps`)

> 阿怎麼又多了兩種 fetch ?
> 

別擔心官網有告訴你：

```
// `force-cache` 是預設可以省略
const staticData = await fetch(`https://...`, { cache: 'force-cache' })

// 'no-store' 是不做緩存
const dynamicData = await fetch(`https://...`, { cache: 'no-store' })

// 設置 revalidate 時間
const revalidatedData = await fetch(`https://...`, {
  next: { revalidate: 10 },
})
```

# 結尾

對於新版的 Nextjs 感覺需要拋去許多固有的用法轉而新的概念，對新的使用者也許學習曲度不會這麼高，不過相信如果是公司舊有的專案，許多也尚未更新到最新的版本，雖然可以兩種形式並存但轉移也需要時間成本，必較兩種不同的模式也是很有趣的，就像多學了很多新知識一樣。而 Vercel 很貼心的做了一些新功能範例供開發者參考，有空可以玩玩並看一下程式碼的實踐方法：
https://app-router.vercel.app/

## 參考資料
Static and dynamic rendering in Next 13
https://dev.to/peterlidee/static-and-dynamic-rendering-in-next-13-52gb
【react】初探server component
https://juejin.cn/post/6918602124804915208#heading-18
Understanding React Server Components
https://vercel.com/blog/understanding-react-server-components
React Server vs Client components in Next.js 13
https://kulkarniankita.com/react/react-server-client-components

![https://ithelp.ithome.com.tw/upload/images/20230922/20152073DUJsq8ENbu.png](https://ithelp.ithome.com.tw/upload/images/20230922/20152073DUJsq8ENbu.png)