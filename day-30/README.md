# 前言

營養師不開菜單系列來到第三十天，首先謝謝有觀看到最後一天的讀者，也恭喜堅持到最後一天的你和我。在部署之後，應用程式上線後，我們就這樣結束了嗎？還是我們可以做什麼呢？最後一篇文章想藉由幾個方向規劃這個專案的未來之路，雖然沒有百分之一百完美的網站，但盡可能的做到可維護性以及功能上的擴充。

# 未來規劃

## 規劃之一：提高客製化程度

在第二天的專案介紹中，有提到：這次的專案計畫是在有限的時間內做出一個上線產品，所以在第一階段中，我們做出了基礎的功能，但看看市面上成熟的產品，有高度的客製化功能，例如背景更多樣化設定、頁面可以分區塊設定等等的功能，可以在不大規模更改專案結構的前提下，進行設定功能的擴增。

## 規劃之二：提高專案的安全性

![https://developers.google.com/static/safe-browsing/images/SafeBrowsing_Logo_Vert_1920.png?hl=zh-tw](https://developers.google.com/static/safe-browsing/images/SafeBrowsing_Logo_Vert_1920.png?hl=zh-tw)

在本專案中雖然宗旨是讓使用者可以建立個人的連結頁面，但實際上卻有潛在的安全疑慮，如果有惡意人士申請帳號並設定惡意連結，這變相成為了惡意網站的收集站，使得所有使用者陷入資安疑慮中。所以下個階段將首先進行連結建立時的安全性檢查，預計是在送出請求前，除了做格式驗證，也會進行該網址的驗證。

預計會使用 [Google Safe Browsing](https://safebrowsing.google.com/)、OpenPhish 以及 PhishTank 等提供 API 的反釣魚檢測服務進行驗證。

## 規劃之三：新增訪問者行為分析功能

![https://ithelp.ithome.com.tw/upload/images/20231008/20152073xTHF9T2qN6.png](https://ithelp.ithome.com.tw/upload/images/20231008/20152073xTHF9T2qN6.png)

隨著大家越來越重視數據來做決策，尤其在社群網站上，了解粉絲點擊個人頁面的哪些連結，就變得特別重要了。未來這個 Link in bio tool 不只是給用戶一個可以放多個連結的地方，也可以讓每位用戶都有自己的後台統計頁面，可以看到哪些連結被點得最多，哪些可能需要調整。這樣更精準地知道粉絲的喜好，再調整自己的內容或是推廣策略！

1. **實時點擊統計**：用戶可以查看每一個連結的點擊次數，了解哪些連結最受歡迎。
2. **地理位置分析**：根據點擊者的 IP 位置，分析點擊者來自哪些地區，幫助用戶了解他們的受眾群。
3. **時間分析**：提供點擊的時間統計，例如最受歡迎的連結在什麼時段最常被點擊。
4. **設備和瀏覽器分析**：分析使用者使用的設備和瀏覽器，例如手機、桌面、Chrome、Safari 等。

## 規劃之四：建立專案監控

![https://ithelp.ithome.com.tw/upload/images/20231008/20152073PxKfdhRRVt.png](https://ithelp.ithome.com.tw/upload/images/20231008/20152073PxKfdhRRVt.png)

在現代數位服務的世界中，穩定性和高效的問題解決能力是極為重要的。任何的延遲或中斷都可能導致使用者流失或是對品牌形象造成損害。因此，即時的監控和問題回報成為每個專案的必要環節。

之後也預計引入 【Sentry】的監控服務，Sentry 主要提供錯誤追蹤和效能監控，能夠即時捕捉應用程式中的問題，並給予詳盡的分析報告。除此之外，它還對應用程式的反應時間進行分析，並對其他重要的網路指標進行監控，以協助開發者找出需要改善之處。當系統出現異常，Sentry 亦會透過即時通知功能，迅速告知開發團隊。所有的資料都清楚展示在一個精緻的儀表板中，方便團隊進行管理和監控。

## 規劃之五：專案測試

![https://ithelp.ithome.com.tw/upload/images/20231008/201520733pbNTP0dBu.png](https://ithelp.ithome.com.tw/upload/images/20231008/201520733pbNTP0dBu.png)

在未來的規劃中，當功能越來越複雜龐大，為了產品專案的穩定性及可靠性，除了使用監控服務，當也預計加入專案測試。預計會以 react-testing-library 和 jest 來進行單元測試。react-testing-library 提供了豐富的API，讓我們能夠著重於應用程式的使用方式，以及如何在 UI 上展現。另外，藉由 "Jest" 提供的豐富匹配器及模擬函數，我們可以靈活地測試組件和 API。不只驗證各功能模塊的預期輸出，我們也將模擬使用者行為，以達到更接近實際使用情境的測試。

# 完賽心得

在這三十天中，不僅要做出作品，也要寫出文章，而產出的內容也不能是『 理論上 』的行為，必須在專案中可以執行起來，就算有提前準備還是有些微吃力，但還是非常欣慰的可以走到最後一天，而且作品也真的成功上線。這三十天的挑戰就如同成為工程師一年以來的心路驗證，雖然不是一路暢通，但從中學習到的新知是無價的，透過自己摸索拼湊出的結果實在是難能可貴。

第一次從構想、設計、前端、後端、資料庫、部署，甚至到優化及 SEO 的實作，這樣一條龍的完全開發，其實就跟當營養師一樣，從開菜單、成本控制、採購、驗收、庫房管理、人員管理、衛生管理到廚餘管理，還有營養教育甚至有些學校營養師還要學會修鍋爐，完全十八般武藝。基本上在這次 Side Project 中都是使用 Vercel 系列的產品，挖得越深越深陷他們家的魅力，Vercel 已經是一種信仰了呢！當然，也是因為有這個機會才能藉由 Vercel 完成一個全端作品。

最後想要說說轉職的心得，這是兩個完全不同類型的職業，作為前端工程師，資源隨手可得，自學一點都不是問題，並且有非常多熱情分享的前輩及同路人，覺得每天都可以有收穫有成長。寫完這三十天系列文，也發現以前是不知道自己的不足，或是不知道自己有不足，現在知道自己的不足後，可以更有目標的前進。

而本系列文的每一天都有附上營養小知識，希望讀者們也可以同時吸收非常日常的營養知識，自己了解後也可以再傳給親朋好友。

寫到這篇也剛好是成為工程師的一週年，營養師關上 Excel 開完菜單了，現在已經打開 VScode 寫全端！感謝一起閱讀到最後一天的讀者們，也感謝一起奮鬥的隊友們，我們都非常棒！

## 所以隊名要叫什麼

感謝隊友們一起撐到第 30 天，以下也介紹隊友們的文章
如果對確保系統穩定性和用戶體驗的可觀測性有興趣，可以參考以下圖文並茂簡單易學的：
[你以為你在學 Grafana 其實你建立了 Kubernetes 可觀測性宇宙](https://ithelp.ithome.com.tw/articles/10319012)系列
如果對 Laravel 從架構規劃到準企業級開發，並搭配雲端資源使用的專案有興趣可以參考：
[Laravel 擴展宇宙：從 1 到 100 十倍速打造產品獨角獸](https://ithelp.ithome.com.tw/articles/10316385)系列

# 工商時間

Side project 網址：[https://link-orchard.vercel.app/](https://link-orchard.vercel.app/ysl0628)

個人網址：https://link-orchard.vercel.app/ysl0628

作品 Github：https://github.com/ysl0628/next13-omni-links

個人經營專業：https://www.instagram.com/whatsulook/