# 前言

系列文到二十五天已經大致開發完成，但完成後會開始各種修改及優化，其中也包括效能的優化。效能對於前台網站是非常關鍵的。當網站載入速度緩慢時，可能會導致用戶流失，進而影響到網站的整體轉換率。為了避免這樣的情況，我們在這系列文章中將著重於三個主要的優化方向：Image、Font 以及 Loading。

# Image

在當前的網頁設計趨勢中，高解析度的圖片常常被用於背景或者作為視覺焦點，但這也增加了網頁的載入時間。因此，圖片的優化尤為重要。一般使用 HTML img 標籤時，為了優化會使用 「 srcset 」為不同的裝置或解析度做圖片大小的優化，為了優化需要生成多種格式的圖檔以及轉檔，又如果圖片是動態來源，我們也無法自行生成，所幸 Next.js 提供了 Image Component 解決了這些麻煩。

## Next.js 提供了什麼優化？

1. 首先根據不同裝置及解析度自動提供正確尺寸的圖片，並且將圖片格式轉為體積小但維持解析度的 WebP 和 AVIF。
2. 圖片只會在進入 viewpoint 時加載，利用瀏覽器原生的 Lazy load 功能，並且提供 blur 的 placeholder 效果來加速頁面載入。
3. 不論圖片來源是本地或遠端，都可以按照需求進行尺寸調整。

## 如何設置

設置照片我將分為本地來源以及遠端(含有 URL)來源，會依據來源而對於圖片大小、Lazy load 設置等等有不一樣的設置：

### 本地來源

本地的定義為使用 import 引入的靜態資源，例如 `.jpg` 、`.png` 及 `.webp` 圖檔，並且會自動的判斷圖片的長寬，所以不需要設定  width 與 height 屬性。

- src：圖片的來源，必須以 import 引入
- alt：當圖片無法顯示時的輔助提示文字，為必填選項
- priority：當設為 true，該圖片資源會被視為高優先載入，並且停用 Lazy loading 的設定。建議在網頁中面積較大的圖片都加上此屬性，可以有效減少 LCP 的時間。
- quality：為被優化圖片的品質，1-100 中數字越大品質越好，檔案大小也越大，預設為 75
- placeholder：當圖片處於載入中的狀態，設置為 `blur` 時可以顯示模糊效果。
    
    ![https://cdn-media-1.freecodecamp.org/images/hbNTQ6U2ikSx2bBCETSrj9QsjVlJReX-Hlqx](https://cdn-media-1.freecodecamp.org/images/hbNTQ6U2ikSx2bBCETSrj9QsjVlJReX-Hlqx)
    

```jsx
import logo from 'public/images/logo.png'

<Image
  src={logo}
  alt="logo"
  className="block object-cover cursor-pointer"
  onClick={() => router.push("/")}
  priority
	quality={60}
	// placeholder="blur"
/>;
```

### 遠端(含有 URL)來源

任何包含 URL 的字串皆會被視為遠端來源，包括本地資源以字串顯示時：`src="/images/logo.png"` 。由於無法預先取得資源的長寬，所以必須設置 width 與 height，或是 fill 屬性 (兩者擇一)，否則會出現以下錯誤訊息：

```jsx
Unhandled Runtime Error

Error: Image with src "/images/logo.png" must use "width" and "height" properties or "layout='fill'" property
```

如果設定 width 與 height 的大小不是原始圖片大小，也會出現錯誤，所以必需在 style 中加上 w-auto h-auto：

```jsx
<Image
  src="/images/logo.svg"
  alt="logo"
  width={200}
  height={32}
  className="hidden md:block object-cover w-auto h-auto"
/>;
```

但每次都要如果都要找到圖片原始大小，幾乎是不可能的任務，尤其當網站有上百上千張時，所以如果要修改圖片大小，請在 `<Image/>` 外包覆一層 `div` ，並且以這個 div 操控長寬：

- `<Image/>` 移除 width 與 height 屬性，並加上 `fill` 屬性。
- 在外層 `div` 設定 width 與 height，由於 `<Image/>` 預設包含 absolute 屬性，所以 `div` 需要加上 relative 或 fix 的 position 屬性。
- 可以在 `<Image/>` 加上 `object-fit` 的樣式來控制圖片的縮放。
- 當 `<Image/>` 設有 `fill` 屬性，需加上 sizes 設定不同視窗大小的響應式尺寸

```jsx
<div className="w-[40px] h-[40px] rounded-full relative">
  <Image
    src="/images/logo.svg"
    alt="logo"
    fill
    className="object-cover"
    sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  />
</div>;
```

再來是 placeholder blur 的設定，如果使用遠端來源必須加上 **`blurDataURL`** 的設定作為臨時圖片。`blurDataURL` 必須是 base64-encoded 的圖片，可以參考官網的[範例](https://github.com/vercel/next.js/blob/canary/examples/image-component/pages/color.tsx)，或是以現成服務生成[色彩圖像檔](https://png-pixel.com/)，而下方則使用 svg 示範：

> 建立一個 svg 字串，並轉換為 base64 ，最後使用 Data URL 格式回傳
> 

```jsx
export function staticBlurDataUrl() {
  const blurSvg = `
    <svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 8 5'>
        <filter id='b' color-interpolation-filters='srgb'> 
            <feGaussianBlur stdDeviation='1'/>
        </filter>

        <rect preserveAspectRatio='none' filter='url(#b)' x='0' y='0' height='100%' width='100%' stroke-width="3" stroke="#BBF7D0"  fill="#15803D" />
    </svg>
`;

  const toBase64 = (str: string) =>
    typeof window === "undefined"
      ? Buffer.from(str).toString("base64")
      : window.btoa(str);

  return `data:image/svg+xml;base64,${toBase64(blurSvg)}`;
}
```

建立好 utils 後，在 Image 中調用：

```jsx
  <Image placeholder="blur" blurDataURL={staticBlurDataUrl()} />
```

# Font

字型的部分，一樣使用 **next/font** 進行優化。首先除了字體的優化，內建使用了 self-hosting 的方法，這些字體檔案將被儲存在自己的 server 或網站，再加上使用底層 CSS `size-adjust` 屬性，不會讓頁面載入時因為字體的轉換而使畫面跑版：

![https://imgur.com/MIYFELS](https://imgur.com/MIYFELS.gif)

而由於是 self-hosting，在應用程式 build 時，CSS 和字型檔案會被下載，並與其他靜態資源一起 self-hosting，所以儘管是 Google 等外部字體，也不會向 Google 發送任何請求。不僅提高了效能也保護隱私。

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073Ld6GYDoFRP.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073Ld6GYDoFRP.png)

## 設定方法

基本設定範圍會是在 Layout 或是 Page 比較高層級的地方定義。在以下的範例中，會分別以預設用法及搭配 TailwindCSS 的方法做介紹：

### 預設用法

1. 在 Layout 中引入 next/font，並選擇字體樣式，可參考：https://fonts.google.com/
2. 建立字體物件，其中可以設定字體的粗細、樣式、子集等等。
3. 設置於 body 的 className 中

```jsx
import { Inter } from "next/font/google";

const inter = Inter({
  style: ["normal", "italic"],
  subsets: ["latin"],
  weight: "100", // 多個粗細則以 array 呈現：['100','200','300']
});

export default async function RootLayout({ children, auth }: RootLayoutProps) {
  return (
    <html lang="en">
      <body className={inter.classname}>{children}</body>
    </html>
  );
}
```

### 使用 TailwindCSS

如果要以 TailwindCSS 來設定字體的變數，除了在字體物件必須加上粗細設定，還要自訂義 variable 屬性，並且在 `tailwind.config.js` 中延伸字體樣式，才能在程式碼中使用：

1. 字體物件中設定 variable 屬性
    
    ```jsx
    const inter = Inter({
      subsets: ["latin"],
      variable: "--font-inter",
      weight: "100",
    });
    const oswald = Oswald({
      subsets: ["latin"],
      variable: "--font-oswald",
      weight: "400",
    });
    ```
    
2. 將 `inter.classname` 改為 `inter.variable` 
    
    ```jsx
    export default async function RootLayout({ children, auth }: RootLayoutProps) {
      return (
        <html lang="en">
          <body className={`${inter.variable} ${oswald.variable}`}>{children}</body>
        </html>
      );
    }
    ```
    
3. `tailwind.config.js` 中以字體物件設定的 variable 為擴充的值
    
    ```jsx
    theme: {
      extend: {
        fontFamily: {
          inter: ['var(--font-inter)'], // 步驟一設定的變數
          oswald: ['var(--font-oswald)'] // 步驟一設定的變數
        },
    	},
    }
    ```
    
4. 於程式碼中使用：`font-inter` 以及 `font-oswald`
    
    ```jsx
    <p className="font-inter">連結設定 Link Setup</p>
    <p className="text-sm font-light text-grey-400 font-oswald">
      最多可新增 8 筆連結 Limit 8 links
    </p>
    ```
    
5. 結果驗證：兩個英文字體不同
    
      ![https://ithelp.ithome.com.tw/upload/images/20231005/20152073R3UPIVF3Es.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073R3UPIVF3Es.png)

# Loading

Loading 的優化主要注重於三個方向 - loading.js、資源的 Lazy loading 以及請求時的 loading，三者會藉由 react-loading-skeleton 以及 react-spinners 兩個套件進行等待時的動畫效果，以提升使用者體驗。

## loading.js

loading.js 是基於 React Suspense 的作用機制，會被自動在 **layout.js** 中調用，並包覆在 children 外部，當內容還在 loading 的時候，會顯示 loading.js 中的內容，而當網頁完全渲染完畢時，則會顯示 children 內容。相同的可以在各個 route 段作各自的 loading 設定，但如果 root 也有 loading.js 會先載完 app/loading.js 再載 app/portal/loading.js，等於會有兩個效果。

![loading](https://nextjs.org/_next/image?url=/docs/light/loading-overview.png&w=1920&q=75&dpl=dpl_AudJamTAq369VDxL1LXxCyUr4HQ4)

在專案中因應某些頁面有 Navbar 的設置，所以會在各自的 route 段中建立 loading.js，root 就不會有設置了：

```jsx
// app/(root)/loading.tsx

import Loader from '@/components/Loader'

const Loading = () => {
  return <Loader />
}

export default Loading
```

## Lazy loading

在瀏覽器中，當初載入頁面時，通常會需要下載一個名為 `index.js` 或 `page.js` 的檔案。這個檔案中包含了頁面所需的所有 JavaScript 程式碼，以確保頁面可以正常運作。然而，隨著應用程式的增長，`page.js` 的大小可能會變得非常大，進而影響到頁面的初載入速度。

而 Lazy loading 的概念是確保在初載入頁面時，只載入當前頁面所需的最小量的 JavaScript 程式碼，而不是加載整個應用程式的所有程式碼。當使用者導覽到其他部分或與頁面進行互動時，需要的其他程式碼才會被按需加載。

在 Next.js 中可以有兩個方法實現 Lazy loading：

- 使用 next/dynamic
- 使用 React 的 `lazy()` 以及 `Suspense`

### 使用時機

- next/dynamic：當有交互作用後才希望可以顯示某個元件時，使用 next/dynamic import 該元件
- `lazy()` + `Suspense`：Client component 中以 Lazy loading 方式引入元件

### 使用範例

- `lazy()` + `Suspense`：NavBar 中的元件，載入時以 react-loading-skeleton 動畫效果呈現
    
    ```jsx
    "use client";
    
    import { Suspense, lazy } from "react";
    import Skeleton from "react-loading-skeleton";
    const UserButtons = lazy(() => import("./UserButtons"));
    
    const NavBar: React.FC<NavBarProps> = ({ currentUser }) => {
      return (
        <Suspense fallback={<Skeleton width={125} height={40} />}>
          <UserButtons />
        </Suspense>
      );
    };
    ```
    
- next/dynamic
    
    ```jsx
    const EditLinkItem = dynamic(() => import('./EditLinkItem'))
    
    {linkType !== '' && (
      <EditLinkItem
        isWebsite={linkType === 'website'}
        onClose={() => setLinkType('')}
        lastItemOrder={links ? links.length - 1 : -1}
      />
    )}
    ```
    
    也可以使用 dynamic 的方式替代 lazy() + Suspense，可以直接將 loading component 寫在 dynamic 的第二個參數中
    
    ```jsx
    const UserButtons = dynamic(() => import('./UserButtons'), {
      loading: () => <Skeleton width={125} height={40} />
    })
    ```
    

### 使用前後檔案大小比對

使用前：`page.js` 大小為 4.4 MB

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073PVbD7jJoZW.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073PVbD7jJoZW.png)

使用後：將 `UserButtons` 做 code splitting 後，檔案大小大幅下降為 1.7 MB

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073PStG9EHKxg.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073PStG9EHKxg.png)

### Server Component 使用 next/dynamic

當使用`dynamic import` 引入一個 Server Component 時，因為 Server Component 已經透過 server 進行了 code splitting，所以它本身不會因為動態導入而有 lazy loading。只有在 Server Component 內部有 Client Components 作為其子組件時，這些 Client Components 是可以透過 `dynamic import` 來實現 lazy loading。

## 請求時的 loading

在進行 API 請求時，從請求到回傳結果中間會有一段等待時間，此時可能會導致使用者疑惑請求是成功或失敗，或是認為頁面無反應，所以在 loading 的時間希望可以加上轉圈效果來表示正在請求中

### 使用範例

- 設定一個 `isLoading` 的 state，預設為 false
- 在請求 (try 之前)前設為 true，當狀態為 true 調用 loading 動畫，並且 disable Button 的行為
- 請求結束後 ( finally )設回 false，停止動畫並顯示結果提示

```jsx
const [isUpdating, setIsUpdating] = useState(false)

// 請求時的狀態操控
const handleUpdateLinks = async () => {
    setIsUpdating(true)
    try {
     // 請求邏輯
      toast.success('更新成功')
    } catch (error: any) {
      notifyError(error, '更新失敗')
    } finally {
      setIsUpdating(false)
    }
  }

// Button 的動畫顯示
<Button
  label={isUpdating ? <ButtonLoader /> : '更新'}
  onClick={handleUpdateLinks}
  disabled={isUpdating}
/>
```

### 使用 react-form-hook

如果使用的是 react-form-hook，其中 `useForm` hook 有提供一個 `isSubmitting` 的狀態，可以取代上述的 `isLoading` 狀態作為等待的判斷：

```jsx
import { useForm } from 'react-hook-form'

const {
    handleSubmit,
    formState: { isSubmitting },
  } = useForm<FormValues>()

<Button
  label={isSubmitting ? <ButtonLoader /> : '儲存'}
  type="submit"
  disabled={isSubmitting}
/>
```

# 結語

隨著網頁設計和開發技術的快速演進，提供出色的使用者體驗已經不再是選擇，而是必要。網頁的效能是用戶體驗的核心部分，載入速度的每秒鐘都可能決定著是否能夠留住訪客或將其轉化為客戶。透過 Next.js 和動畫套件的優化方法，不僅可以優化這些要素，還能為使用者提供更平滑的體驗。經過這篇文章，也親身實作後，發現優化網站的效能不僅僅是技術的問題，更是一門藝術，需要我們不斷學習和探索。

## 參考資料

https://www.freecodecamp.org/news/using-svg-as-placeholders-more-image-loading-techniques-bed1b810ab2c/

https://twitter.com/alex_barashkov/status/1590329974409469952

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073LMZwQ8GP9T.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073LMZwQ8GP9T.png)