![https://ithelp.ithome.com.tw/upload/images/20230915/20152073jZDgzxfbOR.png](https://ithelp.ithome.com.tw/upload/images/20230915/20152073jZDgzxfbOR.png)
# 前言

一年半前，我還是個每天在廚房走跳的營養師，除了跟廚師或阿姨打交道，就是跟廠商吵架，再接著對老師家長畢恭畢敬，職業生涯中完全沒想過這輩子會當前端工程師。在因緣際會以及不斷被慫恿洗腦下，我覺得牙一咬好像真的可以成功。所以從自學到轉職，從中獲得很多樂趣以及成就感，對於博大精深的知識感到渴望與興奮。

# 動機

這次除了有幸跟兩位大神組隊參賽，也鼓起勇氣想為自己挑戰一次，主要是去年 Next.js 13 版本發佈之後在社群上造成大轟動，對於 React 使用者想自己完成一個從 0 到 1 的全端 Side Project 是非常友善的，從開始接觸 Next.js 之後也跟著 YouTube 的實戰做了幾個作品，這次，想從頭到尾自己完成！

不過這次 Side Project 的主題發想其實跟營養師沒有太大的相關，本身有因為興趣經營 Instagram 以及臉書專頁，在社群發達的時代，Link in Bio 這種對於行銷或宣傳性質高的服務已經非常盛行了，但有時候看著那些付費的功能，身為免費仔那種看得到吃不到的感覺，真是巴不得自己直接建一個！

所以這三十天的系列文會將初階需求、畫面設計、資料庫欄位設計、前端實作及工具使用、後端功能實作到優化及部署，一天一天的完成 Link in Bio 專案！
# Roadmap
![https://ithelp.ithome.com.tw/upload/images/20230916/201520734TNjfAlyts.png](https://ithelp.ithome.com.tw/upload/images/20230916/201520734TNjfAlyts.png)
# 目錄

> 前三天以前言及專案介紹以及整體架構作為開章
> 
- [營養師不開菜單的第一天 - 為什麼要寫全端 Project？](https://ithelp.ithome.com.tw/articles/10318966)
- [營養師不開菜單的第二天 - 專案介紹](https://ithelp.ithome.com.tw/articles/10320454)
- [營養師不開菜單的第三天 - 專案架構](https://ithelp.ithome.com.tw/articles/10321267)

> 第四到七天，以 Next.js 作為專案框架，進行大方向功能的介紹。
> 
- [營養師不開菜單的第四天 - 為什麼要用 Next.js](https://ithelp.ithome.com.tw/articles/10321399)
- [營養師不開菜單的第五天 - 為什麼要用 Next.js 的三種 Route](https://ithelp.ithome.com.tw/articles/10323133)
- [營養師不開菜單的第六天 - 使用 Next.js 的連結導覽 & Data Fetching](https://ithelp.ithome.com.tw/articles/10323970)
- [營養師不開菜單的第七天 - 使用 Next.js 的 Server Component](https://ithelp.ithome.com.tw/articles/10324804)

> 第八天開始，將根據專案中所使用的套件，逐一解釋選用它們的原因，以及在專案中的具體實作方式。
> 
- [營養師不開菜單的第八天 - 為什麼要用 MongoDB](https://ithelp.ithome.com.tw/articles/10325586)
- [營養師不開菜單的第九天 - 為什麼要用 Prisma 新世代 ORM 工具](https://ithelp.ithome.com.tw/articles/10326491)
- [營養師不開菜單的第十天 - 為什麼要用 Tailwind CSS + Headless UI](https://ithelp.ithome.com.tw/articles/10327231)
- [營養師不開菜單的第十一天 - 為什麼要用 NextAuth 做身分驗證](https://ithelp.ithome.com.tw/articles/10328508)
- [營養師不開菜單的第十二天 - 要如何申請 OAuth 權限](https://ithelp.ithome.com.tw/articles/10328911)
- [營養師不開菜單的第十三天 - 為什麼要用 Zustand 做狀態管理](https://ithelp.ithome.com.tw/articles/10329157)
- [營養師不開菜單的第十四天 - 為什麼要用 React-Beautiful-Dnd 做拖曳效果](https://ithelp.ithome.com.tw/articles/10330367)
- 營養師不開菜單的第十五天 - 為什麼要用 React-Form-Hook
- 營養師不開菜單的第十六天 - 為什麼要用 Zod 驗證
- 營養師不開菜單的第十七天 - 為什麼要用 Cloudinary CDN
- 營養師不開菜單的第十八天 - 為什麼要用 react-hot-toast

> 完成套件的實作後，我們將深入探討畫面設計、前端以及後端的關鍵實作細節
> 
- 營養師不開菜單的第十九天 - 畫面設計與整體規劃
- 營養師不開菜單的第二十天 - 後端實作 - Model 與 API
- 營養師不開菜單的第二十一天 - 後端實作 - 錯誤處理及 security headers
- 營養師不開菜單的第二十二天 - middleware
- 營養師不開菜單的第二十三天 - 前端實作(上)
- 營養師不開菜單的第二十四天 - 前端實作(下)
- 營養師不開菜單的第二十五天 - 前端錯誤處理

> 專案大致功能完成，緊接著是優化以及部署，最後也會針對專案的發展性及未來功能擴展做相關計畫
> 
- 營養師不開菜單的第二十六天 - 專案效能優化
- 營養師不開菜單的第二十七天 - SEO 優化
- 營養師不開菜單的第二十八天 - Docker 建置
- 營養師不開菜單的第二十九天 - 部屬網站
- 營養師不開菜單的第三十天 - 蛻變成為網頁工程師

# 結尾

這次的實作挑戰及套件工具介紹，皆是基於我廣泛搜集的資訊，再加上個人在專案中或是工作上遇到的坑後，加入一點小小心得分享，內文中大該有 95 % 會以實際的專案 code 作為範例，而剩下的 5 % 則是沒有實作到，但也希望分享給讀者們的內容，如果有任何需改進或錯誤訊息，歡迎留言指教！也希望給菜逼八轉職仔一個鼓勵會更有前進的動力！

![https://ithelp.ithome.com.tw/upload/images/20230916/2015207310xY1Hn5pK.png](https://ithelp.ithome.com.tw/upload/images/20230916/2015207310xY1Hn5pK.png)