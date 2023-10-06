# 前言

SEO，是搜尋引擎最佳化 (Search Engine Optimization) 的簡稱，是一套策略技術，幫助提升網站或網頁在搜尋引擎裡的曝光度。主要的目標是讓網站有更多流量，而這些流量大部分來自搜尋引擎的自然搜尋結果，不是付費或直接點擊連結的流量。SEO 不只是針對網站的文字內容進行優化，還要考慮圖片、影片和其他多媒體。網站的流量來源可以有很多種，例如來自圖片搜尋、影片搜尋、學術文章搜尋或是某個特定產業的搜尋引擎。

# Metadata 對 SEO 的優化

Meta tags 是網頁的後台資訊，提供給搜尋引擎對網頁內容的基本描述。設定適當的 Meta tags能幫助搜尋引擎更正確地解讀和索引你的網站，進而提高在搜尋結果中的排名。而 Next.js 的 Metadata API 提供了一個方便的方式來管理和生成 Meta tags。

# Metadata API 設置

Next.js 提供的 Metadata API 可以讓開發者更方便的設定各種 meta tags，只需要在 `layout.js` 或 `page.js` 檔案中，定義並 export 一個靜態的 metadata 物件或一個動態的 `generateMetadata` function，就會自動為頁面生成相關的 `<head>` element。

在本篇文章中，將會介紹一些對 SEO 具影響力的 Meta tags，並且提供相關的範例。

## Title Tag (`<title>`)

這是每個網頁裡重要的 meta tag 之一，顯示在搜尋結果的藍色標題連結上。這個標題要簡潔、獨特，且要能夠準確反映該頁面的主要內容。Next.js 中的 Title Tag，不僅可以在 root 上設定，也可以在各個路徑中自訂義，或是以 template 的方式呈現：

### 一般設定

建立 metadata 物件後，設定值為靜態 string 的 title 屬性

```jsx
export const metadata: Metadata = {
  title: 'Admin'
}
```

### 建立 title template 物件

在 Root layout 檔案 (`app/layout.tsx`) 中，你可以使用 `title.template`、`title.default` 和 `title.absolute` 來設定 title。

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073AYAjGTSh8l.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073AYAjGTSh8l.png)

```jsx
// app/layout.tsx

export const metadata: Metadata = {
  title: {
    default: 'Link Orchard',
    template: '%s | Link Orchard',
  }
}
```

- default：在 `app/layout.tsx` 中設定 default 作為有設定 template 時，但子路徑未定義 title 時的預設 title。
    
    ```jsx
    // app/(public)/[username]/layout.tsx
    
    export const metadata: Metadata = {}
    
    // 頁籤標題：Link Orchard
    ```
    
- template：可為子路徑定義的標題添加前綴或後綴，變數以 `%s` 定義，當子路徑設定 title 時會呈現：`<子路徑 title> | Link Orchard` 。
    
    ```jsx
    // app/(portal)/portal/layout.tsx
    
    export const metadata: Metadata = {
    	title: 'Portal'
    }
    
    // 頁籤標題：Portal | Link Orchard
    ```
    
- absolute：當 `app/layout.tsx` 中有設定 template，但不想在當前路徑繼承時，可以使用 absolute 忽略並顯示設定的值。
    
    ```jsx
    // app/contact-us/layout.tsx
    
    export const metadata: Metadata = {
    	title: {
    		absolute: 'Contact Us'
    	}
    }
    
    // 頁籤標題：Contact Us
    ```
    

## Meta Description (`<meta name="description" content="...">`)

這個標籤提供了頁面的簡短摘要，並在搜尋結果中顯示。雖然不會直接影響搜尋排名，但一個吸引人的描述可以提高點擊率。

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073OvYKkrX75G.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073OvYKkrX75G.png)

```jsx
export const metadata: Metadata = {
  title: "Link Orchard",
  description:
    "Branch Out with LinkOrchard：探索連結的果園，一次點擊就能找到所有你的精彩內容。將你的最佳連結栽種在這片網路果園中，並讓你的追蹤者在其中自由採摘。不只是 link in bio，更是你品牌故事的繽紛果園。",
};
```

## Canonical Tag (`<link rel="canonical" href="...">`)

當網站出現相似或重複的內容，可能讓搜尋引擎難以判定哪一頁是"標準"版本。Canonical Tag 的功能即是在此情境下指引搜尋引擎，透過在HTML的<link>標籤中加入rel="canonical"屬性，明確標註哪一個URL為首選，確保搜尋引擎能正確理解和索引網頁。

假設 **`LinkOrchard`** 的用戶 Renee 有以下連結：

1. **`https://linkorchard.com/renee123?ref=homepage`**
2. **`https://linkorchard.com/renee123?campaign=summer`**
3. **`https://linkorchard.com/renee123`**

儘管這些連結指向相同的頁面，但由於 URL 的 query string 不同，搜尋引擎可能會視它們為不同的頁面。為了確保搜尋引擎知道 Renee 的主要頁面是第三個URL，我們可以在此頁面的 `Page.js` 中加入的 meta tag，由於各用戶頁面的生成是動態，所以我們使用 `generateMetadata` 以及 `alternates` 屬性來實現：

```jsx
export function generateMetadata({
  params,
}: {
  params: { id: string },
}): Metadata {
  return {
    alternates: {
      canonical: `https://linkorchard.com/${id}`,
    },
  };
}

const Page = async ({ params }: { params: { id: string } }) => {};

export default Page;
```

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073WkdvC4FE1S.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073WkdvC4FE1S.png)

## Robots Meta Tag (`<meta name="robots" content="...">`)：

這個 tag 能告訴搜尋引擎是否要索引這個頁面或追蹤頁面上的連結。例如，使用 **`noindex`** 可以確保搜尋引擎不會索引此頁面，而 **`nofollow`** 則讓搜尋引擎知道不應追蹤頁面中的連結。

在我們的專案裡，後台頁面是受保護的，只有登入後才能存取，所以這些頁面一般不會被搜尋引擎索引。為了保障隱私和確保資訊安全，建議使用 **`noindex`** 進行設定，這樣可以避免搜尋引擎不小心索引到這些後台內容。

```jsx
// app/(portal)/portal/layout.tsx

export const metadata: Metadata = {
  title: "Portal",
  robots: {
    index: false, // 告訴搜尋引擎不要索引此頁面，頁面不會出現在搜尋結果中
    follow: false, // 告訴搜尋引擎不要追蹤此頁面上的連結，即不進一步查看這些連結
    nocache: true, // 告訴搜尋引擎不要將此頁面存儲在其暫存中
  },
};
```

## Sitemap 設置

sitemap 是一個能夠列出你網站上所有頁面的清單，主要目的是幫助搜尋引擎更方便地探索和索引你的網站。當你的網站很大或結構複雜時，有了 sitemap，搜尋引擎就能更輕鬆找到所有頁面，確保它們都被正確地索引。而 XML 格式的 sitemap  還可以提供頁面的其他資訊，像是多久更新一次或是哪些頁面比較重要。

當正在評估網站是否需要設置 Sitemap，可參考以下 Google 的建議：

### 建議設置 Sitemap

1. 網站規模龐大，難以確保所有頁面都被其他頁面連結。
2. 新網站缺乏外部連結，導致 Googlebot 難以找到。
3. 網站有大量多媒體或新聞內容。

### **可能不需要設置 Sitemap**

1. 網站規模小，網頁不超過500頁。
2. 網站內部連結完整，方便搜尋引擎進行索引。
3. 不打算在 Google 上展示多媒體或新聞內容。

在 Next.js 中可以在 app 根目錄上建立一個 sitemap.js，會自動轉為 XML 檔，其中只需要設定『 公開 』的網站，所以後台頁面不適合出現在設定中。每個連頁面有以下屬性：

- url：網頁的 URL。
- lastModified：網頁最後修改的日期，以 `new Date()` 賦值。
- changeFrequency：頁面可能會發生更改的頻率。值可以是：`always`、`hourly`、`daily`、`weekly`、`monthly`、`yearly` 或 `never`。
- priority：此頁面相對於其他頁面的優先級。範圍從 `0.0` 到 `1.0`。

```jsx
import { MetadataRoute } from "next";

export default async function sitemap(): MetadataRoute.Sitemap {
  const users = await getAllUsers();
  const userUrls = users.map((user) => ({
    url: `https://linkorchard.com/${user.username}`,
    lastModified: user.updatedAt,
    changeFrequency: "daily",
    priority: 0.7,
  }));

  return [
    {
      url: "https://linkorchard.com",
      lastModified: new Date(),
      changeFrequency: "yearly",
      priority: 1,
    },
    {
      url: "https://linkorchard.com/contact-us",
      lastModified: new Date(),
      changeFrequency: "yearly",
      priority: 0.4,
    },
    {
      url: "https://linkorchard.com/privacy-policy",
      lastModified: new Date(),
      changeFrequency: "never",
      priority: 0.2,
    },
    ...userUrls,
  ];
}
```

## Open Graph Tags and Twitter Cards

Open Graph Tags 和 Twitter Cards 是幫助我們優化社交媒體分享的 meta tags。Open Graph 是由 Facebook 推出的，當我們分享網站連結到臉書時，它會依照這些設定的標籤來顯示預設的標題、內容描述和圖片，確保分享的外觀既美觀又吸引點擊。而 Twitter Cards 則是 Twitter 的版本，讓我們分享的連結在 Twitter 上呈現為有質感的預覽卡片。透過這些小工具，不僅讓網站在社交平台上的展現更加專業，同時也增加了人們點擊的意願。

而 Next.js 提供開發者在各個路徑資料夾設定 `opengraph-image` (以下簡稱 og)以及 `twitter-image` tsx 檔案為專案個頁面設定這項 meta tag，並且藉由 next/server 的 ImageResponse API 以 JSX 及 CSS 生成圖片。

### metadataBase

```jsx
⚠ metadata.metadataBase is not set for resolving social open graph or twitter images, using "https://link-orchard-ngov4miow-ysl0628.vercel.app". See https://nextjs.org/docs/app/api-reference/functions/generate-metadata#metadatabase
```

### 基本設定屬性

- alt：`image:alt`，設定 og 及 twitter image 的替代文字
- size：`image:width` 及 `image:height`
- contentType：`image:type`
- Image：`image` 本身，使用 ImageResponse 回傳一個圖檔

### 範例

```jsx
// app/(public)/(routes)/[id]/

import { ImageResponse } from "next/server";
import { getUserByUsername } from "@/actions/getUserByUsername";

export const size = {
  width: 900,
  height: 450,
};

export const contentType = "image/png";

export default async function Image({ params }: { params: { id: string } }) {
  const user = await getUserByUsername(params.id);
  return new ImageResponse(
    (
      <div tw="relative flex items-center justify-center">
        <img
          src={user?.customImage || '/images/placeholder.jpg'}
          alt={user?.title || 'cover'}
          tw="object-cover"
        />
        <div tw="absolute flex bg-black opacity-50 inset-0 " />
        <div tw="absolute flex items-center top-2 w-full ">
          <p tw="text-white text-4xl flex font-bold m-5">{user?.title}</p>
          <p tw="text-indigo-200 text-xl flex font-bold m-5">
            {user?.username}
          </p>
          <p tw="text-purple-200 text-xl flex font-bold m-5">
            {user?.updatedAt?.toDateString()}
          </p>
        </div>
      </div>
    ),
    size
  );
}
```

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073VKK757hNgn.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073VKK757hNgn.png)

`http://localhost:3000/ysl0628/opengraph-image-2s6j21?b898c4e5c88b729f` 的圖片內容：

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073EDrPpaocnF.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073EDrPpaocnF.png)

分享的成果：

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073SqtfxXTSnz.jpg](https://ithelp.ithome.com.tw/upload/images/20231005/20152073SqtfxXTSnz.jpg)

![https://ithelp.ithome.com.tw/upload/images/20231005/201520735zUO7BrOnH.png](https://ithelp.ithome.com.tw/upload/images/20231005/201520735zUO7BrOnH.png)

# 結語

SEO 是一個持續優化的過程，並且需要多方面的技巧與策略。使用 Next.js 和相關的工具，我們可以更簡單地管理和優化網站的 SEO，讓我們的網站在搜尋引擎結果中取得更好的排名。適當的 meta tags、sitemap 和社交分享標籤都是提高網站流量的關鍵元素。透過不斷的學習和嘗試，我們可以使我們的網站更容易被搜尋引擎和用戶找到。

## 參考資料

[https://zh.wikipedia.org/zh-tw/搜尋引擎最佳化](https://zh.wikipedia.org/zh-tw/%E6%90%9C%E5%B0%8B%E5%BC%95%E6%93%8E%E6%9C%80%E4%BD%B3%E5%8C%96)

https://www.youtube.com/watch?v=L8_98i_bMMA

https://youtu.be/kFzXOuBFN9Q?si=zzm3nKqShpQLS0zb

https://welly.tw/serp-rank-optimization/what-is-canonical-url

https://github.com/vahid-nejad/nextjs13-seo/blob/main/src/app/post/%5Bslug%5D/opengraph-image.tsx

https://vercel.com/docs/functions/edge-functions/og-image-generation/og-image-examples#using-tailwind-css---experimental

![https://ithelp.ithome.com.tw/upload/images/20231005/20152073w1SYoG9PQe.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073w1SYoG9PQe.png)