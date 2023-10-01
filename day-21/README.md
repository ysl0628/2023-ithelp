在上一章節，我們對請求數據進行了深入的驗證。透過多層篩選和檢查，對於不同的驗證結果，我們已經設定了相對應的錯誤回應。但是，我們還沒有詳細探討在哪些特定情境下應該回傳哪種錯誤狀態。更重要的是，我們需要反思：在程式碼中進行的驗證真的百分之百安全嗎？任何安全措施都可能有破解的方式。儘管只是一個小型的 side project，但我們仍可進一步強化安全措施，以提高網站的安全性。

# 錯誤回傳


回憶一下前一篇文章中有進行各種驗證後回傳相對應的錯戶訊息及狀態，例如註冊的請求：

```jsx
if (!email || !username || !password)
      return new NextResponse('Missing Info', { status: 400 })
```

## 什麼是狀態碼?

狀態碼是 HTTP 協議中規範的三位數代碼，由 1XX ~ 5XX 表示各種回應狀態

- 1XX：請求已被接受，需要繼續處理
- 2XX：代表請求已成功被伺服器接收、理解、並接受
- 3XX：需要客戶端採取進一步的操作才能完成請求，通常需要重新導向
- 4XX：客戶端看起來可能發生了錯誤，妨礙了伺服器的處理
- 5XX：server 端在處理請求的過程中有錯誤或者異常狀態發生

如果是請求成功，一般常見的會是回傳 200 ok

![https://ithelp.ithome.com.tw/upload/images/20231001/201520732yCW4YGiSA.png](https://ithelp.ithome.com.tw/upload/images/20231001/201520732yCW4YGiSA.png)

## 錯誤狀態碼

錯誤的狀態回應就是本段落的重點，在專案中會使用到的有 4XX 及 5XX，但不同情況會有不同的代碼，而不是全部都回傳 400 ，接下來就依據各種情境來說明吧。

### 400 Bad Request

> 由於明顯的客戶端錯誤（例如，格式錯誤的請求語法，太大的大小，無效的請求訊息或欺騙性路由請求），伺服器不能或不會處理該請求。
> 

![https://www.keycdn.com/img/support/400-bad-request-lg.webp](https://www.keycdn.com/img/support/400-bad-request-lg.webp)

任何不符合 Server 端的資料請求會以 400 做處理，例如我上方第一個範例中，註冊請求中，有任何一個欄位為空值，都無法建立一個用戶，所以錯誤狀態碼會以 400 定義。簡單來說，如果請求的 body 中沒有通過驗證就會回傳此狀態。

### 401 Unauthorized

> 即「未認證」，表示當前請求需要使用者驗證。
> 

專案中進行身分驗證的方法為登入後取得 session，並透過 `getCurrentUser` 的方法取得登入的身分。所以除了註冊方法，其他的需要登入後執行的增刪改查動作，皆會進行身分判斷，如果沒得到值(表示登入失敗)，或是比對失敗都會回傳 401 狀態。

```jsx
// endpoint: api/user/{id}

if (!currentUser || currentUser.id !== params.id) {
  return new NextResponse("Unauthorized", { status: 401 });
}
```

### 403 Forbidden

> 伺服器已經理解請求，但是拒絕執行它。
> 

![https://www.keycdn.com/img/support/403-forbidden-error-lg.webp](https://www.keycdn.com/img/support/403-forbidden-error-lg.webp)

雖然專案中未使用到，但如果應用程式中有設定不同角色的權限判斷，則可以依判斷回傳 403 狀態碼。例如某個管理頁面只能由管理者權限訪問，當一般使用者請求該頁面網址，則回傳 403 錯誤。

### 404 Not Found

> 請求失敗，請求所希望得到的資源未被在伺服器上發現，但允許使用者的後續請求。
> 

在當使用者請求一個不存在的頁面時回傳 404 Not Found，例如：用戶想要進入前台頁面，想要訪問 username 為 `blackpink` 的頁面，但輸入成 `pinkblack` 而資料庫中也沒有名為 `pinkblack` 的用戶，這個情況將回傳 404 錯誤。

### 405 Method Not Allowed

> 請求行中指定的請求方法不能被用於請求相應的資源
> 

![https://www.keycdn.com/img/support/405-method-not-allowed-lg.webp](https://www.keycdn.com/img/support/405-method-not-allowed-lg.webp)
假如前端進行請求動作，想要以 PUT 方法修改某個資料，但 Route Handler 中並沒有定義該 endpoint 的 PUT 方法，則會發生請求失敗；或是使用錯 request method 都會回傳 405 錯誤。

### 500 Internal Server Error

> 一個通用錯誤訊息，表示某個未預期的情況阻止了伺服器完成該請求。
> 

在專案中除了 4XX 錯誤，其他 server 端的錯誤會以 **500 Internal Server Error** 作為錯誤訊息，並以 log 呈現方便錯誤盤查。

```jsx
try {
...
} catch (error) {
  return new NextResponse('Server Error', { status: 500 })
}
```

## 驗證錯誤處理

上一章節中有使用了 zod 進行請求來源的驗證，並且藉由 `ZodError` 拋出錯誤訊息

```jsx
const result = validate.safeParse(body);

if (!result.success) {
  throw new z.ZodError(result.error.issues);
}
```

在 try…catch 中使用 `throw error`，該如何取得錯誤訊息並回傳錯誤狀態 400 呢？

由於拋出的錯誤都將傳至 catch 的 error 參數中，所以可以先判斷 error 的型別：

1. error 是不是由 zod 提供的
2. 取得 error message：如果錯誤訊息有多個，會以 string[] 作為 errors 的值
3. 回傳錯誤訊息及狀態 400 

```jsx
try {
...
} catch (error) {
	if (error instanceof z.ZodError) {
	  // 如果希望只回傳第一個錯誤訊息可以使用
	  // const message = error.errors[0]
	  const message = error.errors.map((e) => e.message).join(", ");
	  return new NextResponse(message, { status: 400 });
	}
  return new NextResponse('Server Error', { status: 500 })
}
```

# Security headers

---

在網路世界，HTTP headers 控制瀏覽器與伺服器的交互，「Security Headers」就像是我們的防護罩。這些特別的 HTTP headers 保護網站不受外來威脅，例如「XSS 跨站腳本攻擊」或「點擊劫持」。正確設置這些 headers，能夠為網站和其用戶提供更強大、更可靠的安全性，確保隱私和資料都得到保護。

## 設置方法

在 Next.js 專案中要設置 HTTP headers 無法在 Route Handler 中設定，必須在 `next.config.js` 中統一設定，基本設定方法如下：

- source - 設定的請求路徑，`/(.*)` 表示所有路徑
- headers - response header 的陣列，每組包含 key 和 value
    - key - header 選項，例如 `X-DNS-Prefetch-Control` 或 `X-Frame-Options`
    - value - 選項的設定值

```jsx
module.exports = {
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          {
            key: 'x-custom-header',
            value: 'my custom header value',
          },
          {
            key: 'x-another-custom-header',
            value: 'my other custom header value',
          },
        ],
      },
    ]
  },
}
```

## 各防禦設定

緊接著設定的重要段落中，將介紹各種設定，這些設定旨在防禦不同類型的攻擊。同時，會簡單介紹這些攻擊的方法和原理。

### Content-Security-Policy

攻擊原理是攻擊者嘗試在目標網站上進行跨站腳本攻擊（XSS）和數據注入攻擊，從而危害網站用戶。CSP 通過設置安全政策，限制了哪些來源的資源可以載入，阻止不符合政策的資源執行。攻擊者可能嘗試繞過 CSP，但合理設置的 CSP 可以有效防止這種攻擊。

- 設定 source：控制不同類型的資源來源，有 `default-src` 、`script-src`、`style-src`、`img-src` 等等。各種類型以 `;` 區隔
- 設定來源限制：
    
    
    | Source Value | 範例 | 說明 |
    | --- | --- | --- |
    | * | img-src * | 萬用字元，不做限制 |
    | 'none' | object-src 'none' | 任何資源都不允許 |
    | 'self' | script-src 'self' | 只允許自己同域名 |
    | example.com | img-src example.com | 允許指定的域名 |
    | *.example.com | img-src *.example.com | 允許指定的域名底下的子域名 |
    | 'unsafe-inline' | script-src 'unsafe-inline' | 允許使用 inline 元素，例如樣式屬性，onclick或 javascript：URI |
    | 'unsafe-eval' | script-src 'unsafe-eval' | 允許進行不安全的動態程式碼，例如JavaScript eval（） |

```jsx
{
  key: 'Content-Security-Policy',
  value: "default-src 'self' ; style-src 'self' ; img-src 'self' cdn.example.com ; script-src 'self' ; font-src 'self'"
}
```

### X-Frame-Options

設定是否允許網站中嵌入 iframe 元素，主要目的是防止點擊劫持攻擊 (clickjacking)，這種攻擊方式中，攻擊者試圖將網站內容嵌入到攻擊者的網站中。

> 現代瀏覽器更傾向使用內容安全策略（CSP）的 `frame-ancestors` 選項
> 

可設置的值有兩種

1. DENY：瀏覽器將完全阻止在 frame 中加載網頁內容，無論是來自同一網站還是其他網站的請求。
2. SAMEORIGIN：僅允許在相同網站域名下(來自相同網站的請求)的 frame 中加載網頁內容。

```jsx
{
  key: 'X-Frame-Options',
  value: 'SAMEORIGIN'
}
```

### Permissions-Policy

```jsx
Permissions-Policy: <directive> <allowlist>
```

指定一個或多個權限 directive，達到限制瀏覽器中可使用的功能和API，例如：攝影機、麥克風、地理位置、Cookie、顯示通知等，以增強網站的安全性和隱私。

常用的 directive 有：

1. **geolocation**：地理位置訪問權限。
2. **camera**：攝影機訪問權限。
3. **microphone**：麥克風訪問權限。
4. **fullscreen**：全屏模式的訪問權限。
5. **display-capture**：截圖功能的權限。
6. **web-share**：訪問 `Navigator.share()` Web API 的權限。

```jsx
{
  key: 'Permissions-Policy',
  value: 'camera=(self), microphone=(self http://trusted-ad-network.com), geolocation=(), interest-cohort=()'
}

// * = 無限制； () = 禁止； (self) = 允許同域名網址 ；(<url>) = 白名單網址
```

### X-Content-Type-Options

當網站允許使用者上傳檔案，那**攻擊者就可以惡意上傳一些有 Javascript 特徵的 .jpg 檔**。接著想辦法讓這張圖片被載入到前端來，導致裡面的惡意腳本被執行，因為瀏覽器會認為檔案的 MIME類型 錯誤，並自行推測為 text/javascript。使用 **X-Content-Type-Options** 來防止瀏覽器進行MIME類型嗅探。

> 在沒有 MIME 類型的情況下，或瀏覽器認為它們不正確的某些情況下，瀏覽器可能會執行嗅探來猜測 MIME 類型。
> 

```jsx
{
  key: 'X-Content-Type-Options',
  value: 'nosniff'
}
```

### Referrer-Policy

用於控制網頁瀏覽器在導航到其他網頁時如何包含引用（referrer）資訊。referrer 資訊通常包含了用戶所在的前一個網頁的 URL，並提供有關用戶的網頁瀏覽行為的資訊。

> 預設為 `strict-origin-when-cross-origin`
> 

設定選項：

1. **no-referrer**：不發送引用（referrer）資訊。
2. **no-referrer-when-downgrade**：協議更安全時發送，否則不發送。
3. **origin**：僅發送來源資訊。
4. **origin-when-cross-origin**：同源發送完整，跨源僅發送來源。
5. **same-origin**：同源發送完整，跨源不發送。
6. **strict-origin**：保持協議時僅發送來源。
7. **strict-origin-when-cross-origin**：僅在協議安全等級保持不變時（例如 HTTPS→HTTPS）發送原始來源的資訊。
8. **unsafe-url**：發送完整URL，無論協議或安全性。

```jsx
{
  key: 'Referrer-Policy',
  value: 'origin-when-cross-origin'
}
```

### Strict-Transport-Security

```jsx
Strict-Transport-Security: max-age=<expire-time>; includeSubDomains; preload
```

此項設定是告訴瀏覽器只能通過 HTTPS 來訪問頁面，而不是 HTTP，並且可以使用 `max-age` 設定這項安全策略的執行期限、是否適用所有子域名，以及是否使用 `preload` 。

> 如果專案最後是部署到 Vercel 中，部署時會自動設定此策略，不需要手動設定
> 

⚠使用 `preload` 時，`max-age` 至少為一年，並且必須設定 `includeSubDomains`

```jsx
{
  key: 'Strict-Transport-Security',
  value: 'max-age=63072000; includeSubDomains; preload'
}
```

# 結語

本篇文章深入探討了 HTTP 錯誤狀態碼的種類與應用，希望透過這些狀態碼可以提高 API 穩定性和優化用戶體驗的重要性。同時，我們也介紹了如何透過實施 security headers（例如Content-Security-Policy 和 X-Frame-Options）來增強網站安全，保護應用程式避免受 XSS 和 Clickjacking 攻擊。但如果有跟著實作的讀者，會發現設置後會出現一些意料外的錯誤，所以明天，就藉由 middleware 來做進一步的處理。

## 參考資料

https://nextjs.org/docs/app/api-reference/next-config-js/headers#options

[https://zh.wikipedia.org/zh-tw/HTTP状态码](https://zh.wikipedia.org/zh-tw/HTTP%E7%8A%B6%E6%80%81%E7%A0%81)

https://www.keycdn.com/support/troubleshooting

https://www.geeksforgeeks.org/next-js-security-headers/

https://www.nextjs.cn/docs/advanced-features/security-headers

https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Permissions-Policy

![https://ithelp.ithome.com.tw/upload/images/20231001/20152073LpRcLGOSOp.png](https://ithelp.ithome.com.tw/upload/images/20231001/20152073LpRcLGOSOp.png)