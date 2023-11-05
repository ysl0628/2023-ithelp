![https://ithelp.ithome.com.tw/upload/images/20230920/201520731w6llEIY6z.png](https://ithelp.ithome.com.tw/upload/images/20230920/201520731w6llEIY6z.png)
超商有三種起司三明治，Next.js 則有三種 Route，Dynamic Routes、Parallel Routes、Intercepting Routes (如果有其他 Route 出現在來改名字吧)，Dynamic Routes 是舊有的功能，Parallel Routes、Intercepting Routes 則是 13 版本推出的新功能，看了官方文件後覺得非常有趣，自己實作時也採了很多坑，以下開始分享我的使用!
# Dynamic Routes - `[]`


Dynamic Routes 是可以藉由於 URL 上輸入變數 parameter 來動態地生成或訪問頁面，例如：一個部落格平台可以動態根據每篇文章的ID生成獨特的URL ，而不需要手動為每篇文章定義一個固定的路由。

如同上面的舉例，在我的專案中前台的功能是以使用者的 username 來判斷要顯示哪位使用者設定的內容，這時候我就可以使用 Dynamic Routes ，並以頁面取得的 params 請求 API 取得該頁面的資料 `http://my-domain.com/<用戶的 username>` ，以下 Nextjs 提供了三個層級的 Dynamic Routes：

### 如果資料夾命名為 `[folderName]`

以下 URL 路徑皆會顯示 `app/shop/[slug]/page.js` 中的內容 

1. /shop/clothes
2. /shop/shoes
3. /shop/bottom
4. 以及更多不同的組合

> `page.js` 回傳的 params 格式 `{ params : { slug:string } }` ，`[slug]` 中命名什麼在 params 就以該命名取值，如果是 [id]，則取 `{ params : { id:string } }`
> 

以上面的範例，`slug` 可以是 `clothes || shoes || bottom` 等等

```
// app/shop/[slug]/page.js

const Page async({ params : { slug:string } }) => {
	const data = await getData(slug) // 以 parameter 請求 API，取得各頁面的資訊

    return <div>...</div>
}
```

### 如果資料夾命名為 `[...folderName]`

在這種特定的命名規範下，多個 URL 路徑都會指向同一個檔案：**`app/shop/[...slug]/page.js`**，並顯示其內容。

這包括了以下的 URL：

1. /shop/clothes
2. /shop/clothes/tops
3. /shop/clothes/tops/t-shirts
4. 以及更多不同的組合

> `page.js` 回傳的 `params：{ params : { slug:string[] } }`
> 

以上述的路徑為例，**`slug`** 的值分別為：

1. **`[ "clothes" ]`**
2. **`[ "clothes", "tops" ]`**
3. **`[ "clothes", "tops", "t-shirts" ]`**

### 如果資料夾命名為 `[[...folderName]]`

在這種命名方式下，除了前面提及的 1 ~ 4 URL 路徑，`/shop` 也將被包括在內，但這是在沒有設定 `/shop/page.js` 的情況下。

現在來看 `app/shop/[[...slug]]/page.js` 中的資料結構。根據上述的例子，`slug` 的值將會是：

1. `[ "clothes" ]`
2. `[ "clothes", "tops" ]`
3. `[ "clothes", "tops", "t-shirts" ]`
4. `/shop` 因為沒有接續的 params ，則不會有 `slug` 

# Parallel Routes - `@`


Parallel 中文是平行，我把它理解成在同個路徑底下平行運作，簡單的說法就是在一個 route file 下可以設置多個 page.js，咦？！前面不是都說一個資料夾底下只能一個 page.js 嗎？怎麼辦到的！！

答案就是這個神奇的小老鼠 - `@`
下圖中三個 page.js 的路徑都是 `http://my-domain.com/`
![https://ithelp.ithome.com.tw/upload/images/20230920/20152073zUjVJAZskC.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073zUjVJAZskC.png)
## 特點

- 與 Route Group 一樣，不會影響 URL 的結構
- 像模組區塊化一樣，設置後必須在 layout.js 中引入，＠後面接的名稱就會是 layout 除了 children 的其他 props
    
    ```
    // app/layout.js
    
    export default async function Layout(props: {
      children: React.ReactNode
      setup: React.ReactNode
    	preview: React.ReactNode
    }) {
      return (
        <>
          {props.children}
          {props.preview}
          {props.setup}
        </>
      )
    }
    ```
    
- 如果 Parallel Routes 底下的路徑不相同，表示有某個或某些資料夾下的 page.js 不是活躍的狀態，這時候可以在 @filename 下設置一個 default.js ，並回傳 null 以表示不渲染
    
    ```
    // app/@preview/default.js
    
    export default function Default() {
      return null
    }
    ```
    
- 可以設置各自的 loading.js 以及 error.js

## 使用場景

1. 同個頁面同個路徑下，頁面中有多個區塊，希望可以渲染多個 page.js
![https://ithelp.ithome.com.tw/upload/images/20230920/201520737ZOdwzXteJ.png](https://ithelp.ithome.com.tw/upload/images/20230920/201520737ZOdwzXteJ.png)
    > 這時候會疑問：為什麼不放兩個 component 就好了呢？
    > 
    如同上面特點所說，如果今天 component 的內容比較複雜，並且希望可以各自處理 loading 狀態以及 error 顯示，就可以實現這種模式。結構如下：
    
    ![https://ithelp.ithome.com.tw/upload/images/20230920/20152073xsc1fj5LeO.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073xsc1fj5LeO.png)

2. 依據不同狀態在 layout 中渲染不同模組
    
    最簡單的例子就是，相同路徑下，判斷登入與否來顯示不同頁面
    
    ```
    // app/layout.js
    
    import { getCurrentUser } from '@/actions/getCurrentUser'
     
    export default function Layout({
      portal,
      login,
    }: {
      portal: React.ReactNode
      login: React.ReactNode
    }) {
      const currentUser = getCurrentUser()
      return currentUser ? portal : login
    }
    ```
    

### 持續探討

我自己在實作的時候，想試著將 Route group 與 Parallel Routes 結合，類似以下的結構：
![https://ithelp.ithome.com.tw/upload/images/20230920/20152073kHrOdJTOCy.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073kHrOdJTOCy.png)
卻發現兩者會牴觸，導致 notFound() 的錯誤出現，在官方的 [issue](https://github.com/vercel/next.js/issues/53170) 中也有幾則討論，目前可以多方嘗試來找出可運行的排列組合，如果有讀者發現解決方法也歡迎提供！！
# Intercepting Routes - `(.)`

---

最後一種 route 是攔截 route，如果希望展示某一 route 的內容，但不希望使用者轉跳到另一個環境或畫面上時，就可以使用這個模式

## 特點

- 該路徑會影響 URL 的結構
- 在路徑前以 (.) 或 (..) 或 (..)(..) 或 (…) 的符號作為要攔截後顯示的資料夾
- 攔截資料夾必須以被攔截的 route 命名相同，例如： `(.)login` 攔截 `login` 資料夾
- 可搭配 Parallel Routes 一起服用
- 命名條件是基於 route，而不是實際的 file system

## 攔截符號

- `(.)`: 用來匹配同一層級的 route。
    - 下方範例中，login 的路徑是 : `http://my-domain.com/login`
    - @auth 不會影響 URL 路徑，所以攔截的 login 資料夾路徑也是 `http://my-domain.com/login`
    - 結論：攔截符號為同一層級的 `(.)`
    ![https://ithelp.ithome.com.tw/upload/images/20230920/20152073k5xILjgWd4.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073k5xILjgWd4.png)
- `(..)`: 這個符號用來匹配上一層級的 route。
    - 下方範例中，photo 的路徑是 : `http://my-domain.com/photo`
    - (route) 不會影響 URL 路徑，所以攔截的 photo 資料夾路徑是 `http://my-domain.com/feed/photo`
    - 結論：攔截路徑與原路徑相差一層級，攔截符號為 `(..)`
    ![https://ithelp.ithome.com.tw/upload/images/20230920/20152073GAFPrJhZp6.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073GAFPrJhZp6.png)
- `(..)(..)`: 這個符號用來匹配上兩層的 route。
    ![https://ithelp.ithome.com.tw/upload/images/20230920/20152073vNnDFAWEng.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073vNnDFAWEng.png)
- `(...)`: 這個符號用來匹配從 app 的根目錄開始的所有 route。

## 使用場景

官網中唯一的範例就是 Modal 的顯示，在我的專案中目前是以登入畫面實作，可以依據需求在攔截路徑與原路徑中顯示不同的樣式設計！
![https://ithelp.ithome.com.tw/upload/images/20230920/20152073pBNrHa41U6.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073pBNrHa41U6.png)
![https://ithelp.ithome.com.tw/upload/images/20230920/20152073G3qf7ftVRB.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073G3qf7ftVRB.png)

### 持續探討

在專案中實作時，也是發現 Route Group 結合 Intercepting Routes 的模式算是半成功攔截，成功的顯示 `(.)login` 的內容，但背景卻不是轉跳前的畫面~ 所以可能要等到之後的更新後有沒有修正或其他更多範例來驗證了！

另外想要提醒：如果實作時，多次的修改這些 route 功能，可能有暫存或是使 compile 出錯，在絕望之前可以將 .next remove 在進行 run dev

```
rm -rf .next
npm run dev
```

# 結語

包括 Route Group 和這三種 route 功能，在開發上是很實用的，不過 Route Group、Parallel Routes 和 Intercepting Routes 三者在結合上面可能還需要多研究以及持續注意社群上的討論。在這次專案中，四種方法都有實作，尤其 Parallel Routes 和 Intercepting Routes 這兩個功能雖然網路上可以查詢到的資訊量不大，但自己親身挑戰後，覺得非常有趣也可以打造不一樣的使用者體驗。

## 參考資料

https://github.com/vercel/next.js/issues/53170

https://nextjs.org/docs/app/building-your-application/routing/dynamic-routes

https://nextjs.org/docs/app/building-your-application/routing/parallel-routes

https://nextjs.org/docs/app/building-your-application/routing/intercepting-routes

![https://ithelp.ithome.com.tw/upload/images/20230920/20152073pAsJLFevNy.png](https://ithelp.ithome.com.tw/upload/images/20230920/20152073pAsJLFevNy.png)