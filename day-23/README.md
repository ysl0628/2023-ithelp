# 前言

經過從設計到後端的細緻實作後，我們現在進入了至關重要的前端實作階段。在這個階段，我們不僅要將畫面與資料結合，更要為我們的應用程式注入靈魂。前端不僅僅是視覺的呈現，它是資料和用戶之間的橋樑，賦予用戶操作和互動的能力。在前面的章節，我們已深入探討了如何使用各種套件工具在專案中實現特定功能。而在接下來的兩個章節中，我們將從前端實作的流程入手，以及狀態管理邏輯，隨後深入探討每個頁面的核心功能邏輯，讓整個專案更完整的呈現。

# 前端實作流程

在前端實作的步驟中，主要會先執行大量的畫面以及基本的用戶操作。接著將狀態管理邏輯完成後，待有初步的資料時呈現於畫面中。再來進行後台最重要的表單資料收集及請求，確保基本功能都完成後，再開始進階功能的實作：

1. **設計與 Layout**：設計及建立 Layout 共用元件，例如 NavBar

2. **頁面組織**：建立頁面，並以資料夾進行群組區分；以及整體切版

3. **共用元件建立與整合**：建立共用元件，如 Input、Button，並結合 React-Form-Hook

4. **使用者認證功能**：實作登入註冊功能

5. **狀態管理**：建立 Zustand 功能

6. **頁面內容呈現**：依據 Zustand  存放的資料於各頁面呈現

7. **圖片管理**：以 Next-Cloudinary 實作圖片上傳

8. **表單管理與驗證**：

- 以 React-Form-Hook 收集表單資料
- 結合 Zod 設定表單欄位驗證
- 實作表單請求

9. **進階功能實作**：實作 Dnd（拖曳）功能及前台頁面附加功能

# 狀態管理邏輯

在專案中使用 Zustand 作為狀態管理的工具，同時也搭配 immer 在不修改原始狀態的情況下賦予 state 新值。
參考程式碼：https://github.com/ysl0628/next13-omni-links/blob/main/hooks/useSetup.ts

## State

在 Zustand 中，我們設定的狀態是與資料庫的 User 及 Link Collection 同步的。因此 **`initial state`** 會這樣定義：

```jsx
user: null,
links: null,
```

## Actions

而操作這些狀態的主要功能包括：同步、`user` 的更新或重置，以及 `links` 的更新或重置等動作。儘管許多功能有合併的可能性，但考慮到維護的便利性，仍然選擇將不同功能進行分開設計：

- initialSync：在專案載入時，以資料庫回傳的資料同步更新 state
- update：在 API 請求後更新整個 user 或 links state
- addLink：新增一個連結
- removeLink：移除特定的連結
- updateLink：更新特定的連結
- revertLinks：取消預覽或不儲存後，恢復原本 links
- revertUser：取消預覽或不儲存後，恢復原本 user 資料

### initialSync

在後台操作時，所有 state 皆託付於 Zustand 管理，所以為了取得資料庫資料，在進入後台後必須同步更新個狀態的初始值，在 Server Component 中使用的是 `StoreInitailizer` 的方式進行同步，同步的邏輯為：

```jsx
initialSync: (partial) =>
  set(
    produce((state) => {
      if (partial.user) {
				state.user = partial.user
      }
      if (partial.links) {
        state.links = partial.links
      }
    })
  ),
```

### update

設計為當基本設定有所變更時，可以按需對 `user state` 進行更新；而在連結設定中，當編輯順序有所調整後，會更新整個 `links` 陣列。需要注意的是，因為 `user` 的接收欄位會根據不同頁面而異，所有的欄位在需求上都被定義為 optional。因此在進行更新時，必須檢查每個值是否已經更新。如果沒有接收到新的值，我們會使用原本的資料，否則該值會被設定為 `undefined`。

> 特別注意，user state 中也有 links 欄位，當 links 更新時也要一同更新。
> 

```jsx
update: (partial) =>
  set(
    produce((state) => {
      if (partial.user) {
        state.user.username =
          partial.user.username ?? state.user.username
        state.user.customImage =
          partial.user.customImage ?? state.user.customImage
        state.user.title = partial.user.title ?? state.user.title
        state.user.description =
          partial.user.description ?? state.user.description
        state.user.themeColor =
          partial.user.themeColor ?? state.user.themeColor
      }
      if (partial.links) {
        state.links = partial.links
				state.user.links = partial.links
      }
    })
  ),
```

### addLink

參數為 `newLink` ，當在連結設定中新增一個連結後，將在請求 response 時更新回傳的資料到 `links` 及 `user` 狀態。在排序設定上，為了確保新加入的連結可以立刻呈現在最前方，我們會將它放置為陣列的第一筆資料。

```jsx
addLink: (newLink) =>
  set(
    produce((state) => {
      state.links = [newLink, ...(state.links || [])]
      state.user.links = [newLink, ...(state.user.links || [])]    
		})
  ),
```

### removeLink

參數為 `linkId` ，同樣的刪除一條連結後，也將請求後回傳的資料於狀態中更新

```jsx
removeLink: (linkId) =>
  set(
    produce((state) => {
      const newState = state.links?.filter(
        (l: LinkSetupType) => l.id !== linkId
      )
      state.links = newState
			state.user.links = newState
    })
  ),
```

### updateLink

參數為 `linkId` 及 `newLink` ，修改以 `linkId` 指定的 link 後，也一併將 `newLink` 更新於 user 及 links

```jsx
updateLink: (linkId, updatedLink) =>
  set(
    produce((state) => {
      const link = state.links?.find((l: LinkSetupType) => l.id === linkId);
      const userLink = state.user?.links?.find(
        (l: LinkSetupType) => l.id === linkId
      );
      if (!link || !updatedLink) return;

      // 更新指定的屬性
      const propertiesToUpdate = ["title", "type", "url", "order"];
      propertiesToUpdate.forEach((property) => {
        link[property] = updatedLink[property];
        userLink[property] = updatedLink[property];
      });
    })
  )
```

### revertLinks

編輯連結順序的功能中，若修改後沒有儲存，則恢復修改前的排序

```jsx
revertLinks: (oldLinks) =>
  set(
    produce((state) => {
      state.links = oldLinks
			state.user.links = oldLinks
    })
  ),
```

### revertUser

同 revertLinks 的邏輯

```jsx
revertUser: (oldUser) =>
  set(
    produce((state) => {
      state.user = oldUser
    })
  )
```

# 頁面核心功能

了解完大致的實作流程及狀態管理後，緊接著進入核心功能邏輯的介紹。首先，我們從「NavBar」開始，繼而探索「登入/註冊頁面」的使用者認證。之後，會詳述「基本設定」與「連結設定」的實作。最後，深入「前台頁面」的延伸功能。

## NavBar

專案中 NavBar 顯示的範圍在首頁及後台 (portal) 頁面，並且會依據身分驗證與否顯示不同的元件，主要元件有 Logo、NavLink、UserButtons：

- Logo：所有頁面、所有狀態皆顯示
- NavLink：設定頁面導覽連結，僅後台 (portal) 頁面顯示
- UserButtons：
    - 首頁：未登入顯示登入按鈕，登入後顯示使用者頭像
    - 後台：顯示發佈按鈕及使用者頭像

在確定上述邏輯之後，我們就可以根據路徑和狀態判斷要顯示哪些元件。接著，我們要考慮 NavBar 主元件的放置位置。最初，我考慮將其放在 Root Layout 中，再根據路徑進行判斷。但考量到只有兩個路徑會用到這個元件，且專案中有「Intercepting Routes」的行為，可能會造成路徑判斷出現問題，因此決定將其放置在首頁和後台頁的 Layout 中，依照圖解切分：

### 首頁

![https://ithelp.ithome.com.tw/upload/images/20231003/20152073frvJ8jrqRJ.png](https://ithelp.ithome.com.tw/upload/images/20231003/20152073frvJ8jrqRJ.png)

### 後台頁

![https://ithelp.ithome.com.tw/upload/images/20231003/20152073qLaWIQsUoG.png](https://ithelp.ithome.com.tw/upload/images/20231003/20152073qLaWIQsUoG.png)

### NavBar 中元件顯示判斷

而 NavBar 中的判斷也會簡潔許多，只需要以後台路徑及身分狀態來判斷 NavLink 是否顯示：

```jsx
const NavBar: React.FC<NavBarProps> = ({ currentUser }) => {
  const path = usePathname();
  const isAdmin = path.includes("/portal");

  return (
    <div className="...">
      <Logo />
      {currentUser && isAdmin && <NavLinks />}
      <UserButtons currentUser={currentUser} isAdmin={isAdmin} />
    </div>
  );
};

export default NavBar;
```

後台路徑及身分狀態傳進 UserButtons 中後，再判斷發佈按鈕、user menu 以及登入按鈕顯示：

```jsx
const UserButtons: React.FC<UserButtonsProps> = ({ currentUser, isAdmin }) => {
  return (
    <div className="...">
      {currentUser ? (
        <>
          {isAdmin ? <Button label="發佈" /> : null}
          <div className="...">
              <UserMenu/>
          </div>
        </>
      ) : (
        <>
          <Button label="Log In" />
          <Button label="Sign Up" />
        </>
      )}
    </div>
  )
}

export default UserButtons
```

### 響應式設計

![https://ithelp.ithome.com.tw/upload/images/20231003/20152073Y5rwlItjaq.png](https://ithelp.ithome.com.tw/upload/images/20231003/20152073Y5rwlItjaq.png)

"Link in Bio Tool" 的整體設計考量到前後台的使用場景。前台主要是針對手機使用者進行呈現，而因為使用者可以透過前台連結訪問後台，後台則採取響應式設計。這樣不僅確保了手機用戶的流暢體驗，還讓使用各種設備的用戶都能夠方便地訪問和設定後台。

在響應式的執行上，評估最有挑戰性的實作是 NavBar，我們雖然可以透過基本的縮放、排版或隱藏特定區塊來達到頁面層級的響應性，但在 NavBar 這塊區域集結了 Logo、NavLink 以及 UserButtons/登入註冊三個元件，在手機板有限的寬度空間中，勢必要重新布置規劃。

因此，我的解決方案是：在手機版面中，只顯示 Logo，而將 NavLink 和 UserButtons/登入註冊整合到一個下拉式選單裡，為了方便用戶操作，則以漢堡式選單作為開關操作按鈕。而這個下拉式選單的具體實現方式，我選擇使用 HeadlessUI 的 Disclosure 元件來完成。

1. 在原有的 `NavBar` 元件外，使用 `Disclosure` 進行包覆。
2. 新增一個由 `Disclosure.Button` 建立的 `MenuButton` 元件，這即為漢堡式選單按鈕。
3. 使用 `Disclosure.Panel` 建立下拉選單的具體內容部分。
4. 而要控制這個下拉選單的開關狀態，可以透過 `Disclosure` 的 `open` 參數來進行操作判斷。
5. 最後，在 `MenuButton` 和 `UserButtons` 中進行響應式的樣式調整。

參考程式碼：

https://github.com/ysl0628/next13-omni-links/blob/develop/components/navbar/NavBar.tsx

```jsx
const NavBar: React.FC<NavBarProps> = ({ currentUser }) => {

  return (
    <Disclosure as="nav" className="...">
      {({ open }) => (
        <div className="mx-auto">
          <div className="relative flex h-16 items-center justify-between ">
            <------- 原本的 NavBar 元件 ------->
						<div className="...">
				      <Logo />
				      {currentUser && isAdmin && <NavLinks />}
							<------- 以 Disclosure.Button 建立的漢堡按鈕 ------->
							<MenuButton open={open} />
				      <UserButtons currentUser={currentUser} isAdmin={isAdmin} />
				    </div>
          </div>

          <Disclosure.Panel className="sm:hidden pt-2">
              <------- 下拉選單內容 ------->
          </Disclosure.Panel>
        </div>
      )}
    </Disclosure>
  )
}

export default NavBar
```

# 結語

在這篇上集中，我們已經探討了前端實作的基礎流程、狀態管理的關鍵概念，以及深入分析了 `NavBar` 的核心功能邏輯。希望透過這些資訊，先為讀者們奠定了一個清晰的基礎。然而，這趟旅程還沒有結束。在下集中，我們將繼續深入探討其他頁面的核心功能邏輯，讓整個專案更完整。

## 參考資料
https://tailwindui.com/components/application-ui/navigation/navbars#component-70a9bdf83ef2c8568c5cddf6c39c2331
https://www.figma.com/community/file/1088418504991825797
[https://g801109g51.medium.com/漢堡-hamburger-選單-是福亦是禍-cb61cf491830](https://g801109g51.medium.com/%E6%BC%A2%E5%A0%A1-hamburger-%E9%81%B8%E5%96%AE-%E6%98%AF%E7%A6%8F%E4%BA%A6%E6%98%AF%E7%A6%8D-cb61cf491830)

![https://ithelp.ithome.com.tw/upload/images/20231003/201520733sHNCCxzsY.png](https://ithelp.ithome.com.tw/upload/images/20231003/201520733sHNCCxzsY.png)