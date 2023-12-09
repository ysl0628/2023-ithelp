# 營養師不開菜單的第十一天 - NextAuth 開箱即用的身分驗證套件

![https://ithelp.ithome.com.tw/upload/images/20230926/20152073B7fBvtmCrj.png](https://ithelp.ithome.com.tw/upload/images/20230926/20152073B7fBvtmCrj.png)

NextAuth.js 是專為 Next.js 設計的身份驗證套件。提供了一種簡單的方法來添加 OAuth、電子郵件和其他多種身份驗證方式，我們就不需要編寫繁瑣的程式碼。除此之外，NextAuth.js 也提供 session 管理功能，使開發者可以輕鬆處理用戶 session 和授權。今天的文章除了會說明為什麼要使用 NextAuth.js，也會實作基本的設置以及各屬性的介紹，並且與大家分享自己在開發時遇到需要自訂義的狀況。

# 為什麼要用 NextAuth.js?

1. 我使用 Next.js 開發，而 NextAuth.js 是專為 Next.js 設計的套件
2. **支援多元認證方式**：支援多種第三方認證提供者（例如 Google、Facebook、GitHub 等），以及標準的 Username 和 Password 認證。
3. **安全性**: 提供內建的 CSRF、XSS 和 SQL injection 防護。
4. **JWT 支援**: 使用 JSON Web Tokens (JWT) 可以實現無狀態的身份驗證，這使得應用程式能夠在不訪問資料庫的情況下進行驗證用戶。
5. **Session 管理**：在 client side 可以藉由 hook，同時也支援 server side，可使用 get action 取得用戶的 session。
6. **高度可自定義**：可以自定義認證流程，包括自定義 callback、events 等。在這項專案中有自訂義的需求，下章節也會詳細說明及示範。
7. **資料庫兼容性**: NextAuth 支持多種資料庫，包括 MongoDB、MySQL、PostgreSQL 等等，這方便我們可以藉由資料庫進行用戶的資料或權限存取。

![chart](https://mermaid.ink/img/pako:eNqtVFtPE0EU_iuTedKklrbLbt19INGQ-OQlqU9mX4butN3Y7tbdqYhNk5ZwKRUCRloEKwgBUzUEgRgJAv4ZZi9P_gWnnW2pscHrPmx2Z873nct3zinCpKlhqEAbPypgI4lHdZS2UE41AHvumAQDS09nCDBT4KZljtvYUsD56Vd3ueWVp_3yAZ3ecVe_sLf_puK_3vp2Mu9XjpwXs0CFt3SSKYzxWxVyxoDj2sjIjXweJLD1uE14727iPgOgvD6ECiQzZOtpQzeG0h2CLvQC0EYzs-APXOGOrirA-_DM2Z_kv977BW93LoAOsGYkvYT8zT13-7g_XpbNTwEPdOmXK3T2mIO8w1mnsXqpy_6s6at1OnkIUDKJbRu0dfjtRLkC_VBAFxv0bKV7RMyH2PjLSPqxP4TSh3CXN5zqErCZvW4aHALOj97SqQO6-7J7PoCkV3NeMkZCaxt-ec3d2FGNS7vOqTYuGq_bY87Mc6byP_RY0sIaNoiOsrYKQbHngS4tuK39EKB7M-7WSWlAJqOIoDFkY9YD71ZYEADnkJ5lOtRAHtn2uGlpHNU1HFh0jnU_LTnrTW6OsiRg5MXhp_9NjT9RhE2zdzrnfT6kZ_N08SMfcU6DszYOwqTb-0595VfsfMbcqU2nXnXXprhwbGK8Vo25oZWmW1-lzVb_Mgk8GayOMARz2GIF1tiuKrYvVEgyOIdVqLBPDadQIUvaHVBipkxgMzFhJKFCrAIOwUJeQ6S72qCSYmKz0zwyoFKET6ASFSJhQRTFaEyKSZIUiwghOAGVWDw8LMaHo9fFuCDKMUmOl0LwqWkyikhYFiRRkkVZiImCLEc7dA86dwE91nRiWrf5eu1s2dJ3Kv9Eng?type=png)
# 設置方法

---

NextAuth.js 的設置全部都在 API route 的 `[...nextauth].js` 檔案中，設置的方法很簡單，以下我們從安裝開始，一步一步來操作吧!

## 安裝

```
npm install next-auth
```

## 設置 API route

> 因應 Next.js 升級後 app route 以 Route Handler 作為 API 設置的方法，目前 NextAuth.js 也支援 page route 及 app route 兩種設置方法。兩個設置內容都相同，只是最後 export 的方法有些不一樣~
> 

### Page Route

在 `pages/api/auth` 中建立 `[...nextauth].js` 檔案

- 定義一個 `authOptions` 的物件，型別為 `AuthOptions`
- 物件中會有些基本設置，例如 providers、pages、session、secret 等等
- 最後調用 `NextAuth` ，並 export `NextAuth(authOptions)` 即完成

```
import NextAuth, { AuthOptions } from "next-auth"
import GithubProvider from "next-auth/providers/github"

export const authOptions: AuthOptions = {
	<-------- NextAuth 主要設定 -------->
}

export default NextAuth(authOptions)
```

### App Route

在 `app/api/auth/[...nextauth]` 中建立 `route.js` 檔案

- 一樣設置型別為 `AuthOptions` 的 `authOptions` 的物件
- 開始設置內容
- 設置完成調用 `NextAuth` ，並設置 `NextAuth(authOptions)` 為 handler 或其他變數
- 最後 export GET 及 POST：`export { handler as GET, handler as POST }`

```
import NextAuth, { AuthOptions } from "next-auth"
import GithubProvider from "next-auth/providers/github"

export const authOptions: AuthOptions = {
 <-------- NextAuth 主要設定 -------->
// adapter:PrismaAdapter(prisma),
// providers:[].
// pages:{},
// secret:process.env.NEXTAUTH_SECRET,
// callback:{}
}
const handler = NextAuth(authOptions)

export { handler as GET, handler as POST }
```

## 設置 `authOptions`

### adapter

可作為與資料庫連接的橋樑，負責建立和保存用戶的身份驗證數據，除了 MongoDB、MySQL、PostgreSQL 也支援 Prisma ! 由於我的專案是以 Prisma 建置，以下就以 Prisma 為例

1. 安裝 `@auth/prisma-adapter`

```
npm install @auth/prisma-adapter
```

2. 在 options 中設定 adapter，引入 prisma client 後，將其設置於 PrismaAdapter 中

```
import { PrismaAdapter } from '@next-auth/prisma-adapter'
import prisma from '@/app/libs/prismadb' // 封裝好的 prisma client 實例

export const authOptions = {
	adapter: PrismaAdapter(prisma),
}
```

### provider

必填欄位，設定登入驗證的 providers，可設定多組如：Google, Facebook, Twitter, GitHub, Email 等等 provider，主要分為 OAuth、Email 及 Credentials 三種驗證方式

**OAuth**

第三方驗證，主要設置該種類的 clientId 和 clientSecret

先引入欲設定的 OAuth Provider 後，於該服務取得 ID 及金鑰後再於環境變數中設定並引用

```
    ****Provider({
      clientId: process.env.****_CLIENT_ID as string,
      clientSecret: process.env.****_SECRET as string
    })
```

> 如何取得 OAuth 的 ID 及金鑰？我會在下一篇文章中一一介紹設定！
> 

---

**Credentials**

使用自定義的 Username / Email 和 Password 等憑證進行身份驗證，可連接至 db 設定權限

- `name`：用來定義在登錄表單上如何顯示這種認證方法
    
    > 如果將 **`name`** 設置為 "Credentials"，那麼在登錄表單上，用戶會看到一個按鈕，上面寫著 "Sign in with Credentials" 或類似的文字。
    > 
- `credentials`：定義和描述應該在登錄表單上出現的欄位，當用戶提交這些欄位的值時的驗證型別。
- `authorize`：檢查提交的 Username / Email 和 Password 是否與存儲在資料庫中的數據相符，當用戶提交憑證嘗試登錄時，**`authorize`** 會被調用。
    
    ```
    import CredentialsProvider from "next-auth/providers/credentials"
    ...
    providers: [
      CredentialsProvider({
        name: 'Credentials',
    		// 指定你預期提交的任何欄位
    		credentials: {
            email: { label: 'email', type: 'text' },
            password: { label: 'password', type: 'password' }
        },
    		async authorize(credentials) {
    	      if (!credentials?.email || !credentials?.password)
              throw new Error('Invalid credentials')
    				// 與 prisma中的 user 進行比對
            const user = await prisma.user.findUnique({
              where: {
                email: credentials.email
              }
            })
    
            if (!user || !user.hashedPassword)
              throw new Error('Invalid credentials')
    				// 確認密碼正確
            const isValid = await bcrypt.compare(
              credentials.password,
              user.hashedPassword
            )
    
            if (!isValid) throw new Error('Invalid credentials')
    
            return user
          }
      })
    ]
    ...
    ```
    

### pages

代表 NextAuth.js 提供各種驗證方法的路徑位置，自定義身份驗證和授權過程中不同頁面的路徑

```

  pages: {
    signIn: '/auth/signin',
    signOut: '/auth/signout',
		// 當出現身份驗證錯誤時，例如驗證失敗或訪問被拒絕，
		// NextAuth.js 會將用戶重定向到這個路徑，以顯示相應的錯誤信息。
    error: '/auth/error', // Error code passed in query string as ?error=
		// 當需要用戶進行電子郵件驗證時，NextAuth.js 會將用戶重定向到這個路徑，
		// 以顯示相關的驗證請求。
    verifyRequest: '/auth/verify-request', // (used for check email message)
    // 當有新用戶進行註冊操作並需要額外資料時，NextAuth.js 會將用戶重定向到這個路徑，
		// 以收集新用戶的相關資料。
		newUser: '/auth/new-user' 
		// 當用戶忘記密碼並需要進行密碼重置時，NextAuth.js 會將用戶重定向到這個路徑，
		// 以執行密碼重置相關的操作。
		forgotPassword: '/auth/fogot-password'
		resetPassword: '/auth/reset-password'
  }

```

### secret

對 tokens 及 cookies 進行加密的密碼，

> ⚠若在 production 中沒有設置會跳出錯誤警告
> 

```
secret: process.env.NEXTAUTH_SECRET
```

### debug

當設定為 `true` 時，`NextAuth` 會在 terminal 輸出 log 及資訊，建議僅在開發環境中開啟。

```
debug: process.env.NODE_ENV === 'development',
```

### session

用於儲存和追踪用戶身份驗證狀態的機制。它可用於確定用戶是否已經通過身份驗證以及驗證相關資料（例如用戶 ID、角色等）。session 通常以安全的方式儲存在瀏覽器 cookie 中或 server 端。

- `strategy`：預設是 "jwt"，但如果 options 中設置了 `adapter`，則預設選擇會改為 `"database"`。在 `"database"` 模式下，session cookie 只會包含一個 `sessionToken`，這個 token 用於在 DB 中查找 session。
- `maxAge`：session 的最大有效期，超過此期限後，session 將不再有效
- `updateAge`：控制多久更新一次 session 的期限到 DB，如果使用 JWT 作為 session 存取，這個選項會被忽略。
- `generateSessionToken`：可以自定義生成 session token

```
session: {
    strategy: 'jwt', // 使用 JSON Web Token（JWT）於 session
    maxAge: 30 * 24 * 60 * 60, // session 的最大有效期（以秒為單位）
},
```

實測登入後可以在瀏覽器 cookies 中看到 token 的設置

![https://ithelp.ithome.com.tw/upload/images/20230926/20152073E2UGwK64fs.png](https://ithelp.ithome.com.tw/upload/images/20230926/20152073E2UGwK64fs.png)

- CSRF token 是一種 XSS 防護措施，確保提交給 Server 的請求是合法的，並且真的來自用戶。
- callback URL cookie 儲存了原始請求的位置（或其他特定位置），所以用戶可以在認證後返回到正確的頁面。

### jwt

主要用於 JWT 本身設定

- **`maxAge`**: 這是 JWT 的最大有效期。這意味著，當 JWT 過期後，它將不再有效，並且用戶需要重新獲取新的 JWT。
- **`encode`** 和 **`decode`**: 自定義函數，可以對 JWT 的內容進行加密和解密。

> ⚠如果 JWT 及 session 皆有設定 `maxAge` ，兩者值應該要相同，才能確保一致性！
> 

```
jwt: {
  maxAge: 60 * 60 * 24 * 30,
  async encode() {},
  async decode() {},
}
```

### events

可以讓開發者監聽並對各種認證相關的事件做自訂義設定。

以下當任何事件被觸發，相對應的 callback function 就會被執行。可以在這些 callback function 中執行需要的自定義邏輯。

```
events: {
    signOut: async (message) => {
      // 當用戶登出時會觸發
      console.log('用戶已登出:', message.session?.user);
    },
    signIn: async (message) => {
      // 當用戶登錄時會觸發
      console.log('用戶已登錄:', message.user);
    },
    createUser: async (message) => {
      // 當新用戶被建立時觸發
      console.log('新用戶已建立:', message.user);
    },
    linkAccount: async (message) => {
      // 當一個帳號被連接到另一個帳號時觸發
      console.log('帳號已連接:', message.user);
    },
    // ... 其他事件
  },
```

### callbacks

callbacks 設定選項可以讓開發者在進行特定操作時控制所發生的事情。以下是我在這次專案中實際遇到的狀況：

> 情境描述：
我將資料庫中 email 及 username 欄位設為唯一值，因為我需要以 username 作為前台頁面的 params，但是 OAuth 的預設不會將有 username 的欄位，這個情況會導致我的資料不符合，於是我必須在註冊的同時，自己手動設置 default username 進資料庫；另外必須注意的是 OAuth 的註冊及登入都是調用 `NextAuth` 的 `signIn()` 方法！
> 

```
callbacks: {
  async signIn({ user, account }) {
    // 檢查此 email 是否已存在於資料庫
    const exists = await prisma.user.findUnique({
      where: { email: user.email as string }
    });

    // 如果是 OAuth 登入且用戶不存在 -> 代表是註冊
    if (account?.type === 'oauth' && !exists) {
      // 使用 email 的前綴作為預設 username
      const defaultUsername = user?.email?.split('@')[0];

      try {
        const newUser = await prisma.user.create({
          data: {
            username: defaultUsername,
            // ... 其他用戶資料
          }
        });

        await prisma.account.create({
          data: {
            // ... 帳號資料
          }
        });
      } catch (error) {
        console.error("Error during user creation:", error);
        return false; // 返回 false 以表示登入失敗
      }
    }
    return true; // 登入成功
  }
}
```

# 取得 session 的方法

取得 session 的方法可以分為 Client Side 及 Server Side 兩個部分，其中 Client Side 又有 useSession 及 getSession 兩種方法，而 Server Component 則可以使用 getServerSession 實現功能，以下直接進入實作範例：

## Client Side

### 使用 useSession()

在 Client Side 中 NextAuth 提供了 hook 方式取得 session 資訊，前提是必須設定 `SessionProvider` 才能使用，所以先讓我們從 `SessionProvider` 的建立開始。

- 在第七天的文章中，有提到在 Next.js 中可以把所有 Povider 集中為一個 Providers component，再設置於 Root Layout 中，所以首先將 `SessionProvider` 設置在 Providers component 中：

```
"use client"

import { SessionProvider } from "next-auth/react"

const Providers = ({ children, ...props }) => {
  return (
    <SessionProvider >
      <ToasterProvider />
      {children}
    <SessionProvider />
  )
}

export default Providers
```

- 接著就可以在 Client Side 中調用 `useSession()`
    - data：當 session 還沒 fetch 時，data 的值會是 `undefined`。如果試圖取得 session 但失敗時，data 的值會是 `null`。成功取得 session 時，data 的值將是 `Session`。
    - status：有 loading、authenticated 及 unauthenticated 三種狀態

```
const { data, status } = useSession()
```

### 使用 getSession()

在 Client Side 調用正在登入中的 session，因為不需要 `SessionProvider` 輔助，所以如果在 provider 之外需要被調用 session 狀態則可以使用此方法。

```
async function myFunction() {
  const session = await getSession()
  /* ... */
}
```

## Server Side

### 使用 getServerSession()

雖然在 Server Side 一樣可以使 getSession 取得 session，但為了避免額外的 fetch 調用，官方建議還是使用 getServerSession()。而專案中使用的場景是在 getCurrentUser 這個 action 中，取得 session 的同時也取得用戶的基本資料：

```
// actions/getCurrentUser.ts
import { authOptions } from '@/app/api/auth/[...nextauth]/route'

export async function getSession() {
  return await getServerSession(authOptions)
}

export async function getCurrentUser() {
    const session = await getSession()
	<------ 其他操作邏輯 -------->
}
```

# 結語

從設置到 session 的取得就如上面一連串的介紹，NextAuth.js 所提供的功能可說是整個專案的腎臟，進行過濾及保護的作用，所以一篇文章當然是無法完全介紹完他的重要性，而且光依靠這個套件還無法完成整個身分驗證的實作，除了與資料庫連接以進行用戶資料的讀取和寫入，我們還可以使用第三方驗證方法。因此，明天將依照不同的第三方認證提供商，介紹他們各自的申請流程。

## 參考資料

NextAuth.js Documentation
https://next-auth.js.org/getting-started/client#getsession

![https://ithelp.ithome.com.tw/upload/images/20230926/20152073O7KX2FX2DR.png](https://ithelp.ithome.com.tw/upload/images/20230926/20152073O7KX2FX2DR.png)