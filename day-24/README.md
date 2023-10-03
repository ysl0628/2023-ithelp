# 前言

接續著上一篇的前端實作分享，我們介紹完 NavBar 的核心功能邏輯以及響應式設計，這篇文章中我們的焦點將轉向「登入/註冊頁面」，深入挖掘使用者認證的細節與技巧。接著逐一解析「基本設定」與「連結設定」的實作流程，包括資料取得後的格式轉換以及請求的處理方法。最後是「前台頁面」的延伸功能。實作中所有的表單處理為先前介紹的 React-Form-Hook 以及 Zod 做驗證，可參考系列文的介紹。讓我們繼續這趟前端實作之旅吧！

# 登入/註冊頁面

首先以「登入/註冊頁面」做為開端，回顧一下系列文中，我們利用 NextAuth.js 實現身分驗證，並介紹了 API 設置和 OAuth 的步驟。這邊我們將探討如何發送請求以及接收和呈現請求結果給使用者。實作方法會分為一般的帳號密碼以及 OAuth 註冊登入。

## 帳號密碼方式

在專案中有提供以使用者自行以帳號密碼的方式進行註冊登入，在 `NextAuth.js` 中，我們使用了 Credentials Provider 來設定這個登入機制。但當涉及到「註冊」時，需要在後端實作部分建立一個 register 的 api 讓用戶的資料直接寫入資料庫中。功能主要以 React-Form-Hook 進行資料的收集與驗證後，在 `onSubmit` 的處理函式中加入以 `axios` 發送請求的相關邏輯。

- POST 請求是向自己的 API 請求，所以 endpoint 為 `/api/register`
- 註冊成功後轉跳至後台，以 `router.push` 實現
- 如果請求失敗則跳出通知顯示

```jsx
const onSubmit: SubmitHandler<FieldValues> = async (data) => {
  try {
    await axios.post('/api/register', data);
    router.push('/portal/basic');
  } catch (err) {
    toast.error('註冊失敗');
  }
}
```

當使用者試圖登入時，我們會使用 `NextAuth.js` 提供的 `signIn` 方法並帶入參數，等於進行 `POST "api/auth/signin/credentials"` 的請求，當請求發送時，它會觸發 Credentials Provider 的登入機制。

- 第二個參數是一個物件，包含:
    - 使用者輸入的 email 和 password。
    - `redirect` 設置為 `false` ，若登入失敗，頁面不會重新導向。
    - `callbackUrl` 是登入成功後導向的 URL。

> 如果用戶在登入前試圖訪問受保護的頁面，登入成功後，他們會被重定向到之前嘗試訪問的那個頁面。但是，如果用戶是直接點擊登入按鈕來進行登入，他們將被導向到我們在 `signIn` 方法中設定的 `callbackUrl`。若未設定 `callbackUrl`，系統將預設將用戶導回主頁 “/” 路徑。
> 

以下是登入的實作範例：

- 以 `signIn` 發送請求登入，等待請求結果
- 如果 `callback?.ok` 回應為 true，表示登入成功，依據 callbackUrl 的設置，將導向指定的頁面並跳出成功的通知。
- 如果 `callback?.error` 存在，表示登入失敗。由於設定了 `redirect: false`，所以用戶不會被重新導向到其他頁面，而是會在當前頁面收到一個表示登入失敗的通知。

```jsx
const onSubmit: SubmitHandler<FieldValues> = (values) => {
    signIn('credentials', {
      email: values.email,
      password: values.password,
      redirect: false,
      callbackUrl: CALLBACK_URL
    })
      .then((callback) => {
        callback?.error
          ? toast.error('登入失敗')
          : toast.success('登入成功')
      })
  }
```

## OAuth 方式

第三方驗證的方式相對簡潔非常多，因為登入及註冊都是調用 `signIn` 的方法請求，輸入參數及後續邏輯也都相同，由於各個 OAuth 只有輸入參數不同，因此也封裝為 handleSocialLogIn 的 function：

- 以 `signIn` 請求登入，並於第一個參數輸入使用的 OAuth 類型 (參數型別為 String)
    - 等於進行 `POST "api/auth/signin/<OAuth 類型>"`
- 第二個參數一樣可設定 redirect 及 callbackUrl
- 等待請求結果，再依據成功或失敗顯示通知

```jsx
const handleSocialLogIn = async (socialType: string) => {
    try {
      await signIn(socialType, {
        callbackUrl: CALLBACK_URL
      })
      toast.success('登入成功')
    } catch (error) {
      toast.error('登入失敗')
    } 
  }
```

```jsx
// 調用 handler 並傳入驗證類型
<Button onClick={() => handleSocialLogIn("google")} />;
```

# 基本設定

在基本設定頁面中，我們將著重於三大功能：首先是對圖片的上傳與資料收集；其次是能夠重置已上傳的圖片；最後，應用程式提供了一個預覽功能，讓用戶可以即時查看所做的更改。

## 圖片上傳與重置: 以 Next-Cloudinary 和 React-Form-Hook 為基礎

通過組合 Next-Cloudinary 的元件和 React-Form-Hook，我們實現了圖片的上傳和重置。Cloudinary 元件上傳圖片後，會回傳一個 CDN 的 image URL，這個 URL 會暫時存放在 React-Form-Hook 的 `useForm` 狀態中：

```jsx
const handleUpload = (url: string) => {
	// setCustomValue 為以 setValues 封裝的 function
  setCustomValue("customImage", url);
};
```

而操作介面中圖片會顯示出來，此圖片的狀態由 `useForm` 控制，初載入以全域的 user state 的資料呈現，上傳後會以新的值顯示：

當使用者上傳圖片後，新的網址只會暫時儲存在 `useForm` 的狀態中，並不會發送任何請求或修改 Zustand 的全域狀態。因此，如果使用者後悔並希望回到原先的照片，可以直接使用 `useForm` 中的 `resetField` 方法來重置圖片欄位回到其預設值：

```jsx
const handleResetImg = () => {
  resetField("customImage");
};
```

## 預覽功能

在應用程式中的所有預覽功能資料來源都是取自全域狀態，而預覽時也希望在更新狀態前做一遍資料驗證，所以會使用到 useForm 的 `trigger` 功能：

- 先調用 `trigger` 進行資料驗證。如果回傳值是 `false`，表示驗證未通過，不再進行後續操作
- 若資料通過驗證，則使用 `getValues` 取得 `useForm` 中的表單資料
- 最後，更新 Zustand 的 user state

```jsx
const handlePreview = async () => {
  const result = await trigger();
  if (!result) return;

  const basicValues = getValues();
  update({ user: basicValues });
};
```

![https://imgur.com/hENt0Ec](https://imgur.com/hENt0Ec.gif)

# 連結設定

連結設定頁面主要著重於使用者的連結管理，如新增或修改。同時，我們也提供更新連結顯示順序的功能，並允許使用者在調整後選擇發送更新請求或取消排序。

## 新增修改連結

[範例連結](https://github.com/ysl0628/next13-omni-links/blob/develop/components/page/portal/LinksSetup/EditLinkItem.tsx)

兩個操作的邏輯都是將 `useForm` 收集的資料進行 API 請求，請求後將回傳的資料更新於全域的 links state，不同的是 endpoint，以及參數的傳遞

- 新增：以 POST 請求，並調用 `addLink` 更新狀態
    
    ```jsx
    const { data: res } = await axios.post<AxiosResponse<Link>>(
    	`/api/user/${user?.id}/links`, 
    	result
    );
    addLink(res.data);
    ```
    
- 修改：以 PUT 請求，傳遞需修改連結的 id 以及修改的內容，並調用 `updateLink` 更新狀態
    
    ```jsx
    const { data: res } = await axios.put<AxiosResponse<Link>>(
    	`/api/user/${user?.id}/links/${values.id}`, 
    	result
    );
    updateLink(res.data.id, res.data);
    ```
    

## 更新順序

連結設定頁面加入了編輯順序模式，讓使用者透過拖放方式調整連結的順序。當進入此模式時，有幾個重要的實作細節：

1. 結合 React-Beautiful-Dnd 的 `onDragEnd` 功能，必須同時更新 user state，實現拖放後的即時預覽。
    
    ```jsx
    const onDragEnd = (result: DropResult) => {
    	// 經過拖曳後的重新排序
      const items = reorder(links, result.source.index, result.destination.index);
    	// 將排序後的結果更新於全域狀態達到即時預覽效果
      update({ links: items });
    };
    ```
    
2. 如果 React-Beautiful-Dnd 的元件中使用 order 作為拖放時的 index ，可能會造成操作的判斷錯誤，但 API 的回傳又要依據 order 判斷，所以在 API 請求前，需要格式化連結的 order 值。
    
    ```jsx
    const updateLinks = links?.map((item, index) => {
      return {
        ...item,
        order: links.length - index, // 以降冪排序
      };
    });
    const { data: res } = await axios.patch(`/api/user/${user?.id}/links`, {
      links: updateLinks,
    });
    ```
    
3. 若使用者決定取消調整，`revertLinks` 功能能夠快速恢復到原始的全域狀態。
    
    ```jsx
    // 全域狀態後先以 useState 保存原始狀態
    const links = useSetup((state) => state.links)
    const [originalOrder] = useState<LinkSetupType[] | null>(links)
    
    // 以原始的狀態恢復 links state
    const handleCancelUpdate = () => {
      revertLinks(originalOrder);
    };
    ```
    

### 補充說明連結排序邏輯

連結的排序可以分為資料庫以及全域狀態儲存的

- 資料庫：每組連結物件中都有 order 屬性，該屬性為資料庫回傳的排序依據，所以 API 請求的動作，如果有修改連結排序都必須額外處理 order 的格式，所以上面的實作中，會以 `links.length - index` 實現降冪排序。
- 全域狀態：在新增連結的 action 中，已經手動將新物件放在 state 的最前方做回傳，所以從 hook 取得的 state 不需要再 format。

# 前台頁面延伸功能

前台頁面不僅呈現後台設定的資訊，為了提高使用者的操作體驗，我們也加入了一些附加功能。其中，我們設計了一個動態的『更多』按鈕。點擊後，它會以動畫效果展開一個小視窗，讓使用者可以方便地複製連結。

## 實現方法

實現複製的功能使用的是 **Clipboard API**，可以向瀏覽器中的系統剪貼簿進行訪問操作，是以非同步的方式讀取或寫入剪貼簿，不僅可以處理純文字也可以處理 HTML 內容：

- 先以 username 定義要被複製的 URL
- 調用 Clipboard API 的 `writeText` 操作複製行為

```jsx
const copyUrl = `<my-domain>.com/${username}`;
const handleCopy = async () => {
  try {
    await navigator.clipboard.writeText(linkUrl);
    toast.success("已複製連結");
  } catch (err) {
    toast.error("連結複製失敗");
  }
};
```

![https://imgur.com/VPOSQi2](https://imgur.com/VPOSQi2.gif)

# 結語

連續兩天的前端實作在此告一段落，比起冗長的逐行程式碼，希望是以一個重點操作來做介紹，在寫這兩篇文章的同時也發現當初開寫的時候有很多層面沒考慮到，等於也再重新 code review 了一次。實作結束但只是功能都正常可以運作，不過當我們遇到錯誤時都會跳出驚悚的預設畫面，所以下章節我們將進入前端錯誤處理的部分，讓使用者獲得更友善的用戶體驗，同時也提供開發者有效的錯誤資訊。

## 參考資料

https://ithelp.ithome.com.tw/articles/10271977

![https://ithelp.ithome.com.tw/upload/images/20231003/20152073Kme4AwiLyx.png](https://ithelp.ithome.com.tw/upload/images/20231003/20152073Kme4AwiLyx.png)