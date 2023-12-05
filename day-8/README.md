# 營養師不開菜單的第八天 - 為什麼要用 MongoDB
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073qlbwTH4HZH.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073qlbwTH4HZH.png)

食物進入身體後，經過消化系統分解為能量儲存：葡萄糖給予快速能量，而脂肪提供長期儲存。考慮資料庫時，我們也會選擇適合的“能量”來源。像葡萄糖那樣，有些資料庫適合迅速存取；像脂肪那樣，其他資料庫則適合長期儲存。在思考為什麼要用 MongoDB 之前，我們需要先簡單了解資料庫的種類以及特點。

# 資料庫的種類及特點

依據資料儲存、查詢、結構、擴展性、使用情境等特性可以分為關聯式資料庫(SQL)及非關聯式資料庫(NoSQL)：

## 關聯式資料庫 (SQL)
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073W1c4w6IATY.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073W1c4w6IATY.png)

1. **產品**：MySQL, SQLite, PostgreSQL, Oracle… 等
2. **固定的資料結構**：定義好資料的 Table Schema ，有固定的欄位，當 Table 被建立後，若要變更結構需要進行資料庫 Migrate，但可以確保資料的完整性、一致性。
3. **資料間的高度關聯性**：輕鬆地聯接多個 Table，解決複雜的資料問題
4. **強大的查詢能力**：提供強大的查詢語言，方便地從多個關聯表中取得資料。
5. 提供 **ACID** 屬性：原子性（Atomicity）、一致性（Consistency）、隔離性（Isolation）和持久性（Durability）。確保資料的可靠性和完整性。
    1. **原子性 (Atomicity)：**
        - 一個交易（Transaction）的所有操作只有失敗或成功兩個結果。不會出現部分操作成功，部分操作失敗的情況。
            
            > 例如：假設 A 想要從銀行帳戶轉 100 元到 B 的帳戶，如果轉帳的任何部分失敗(A 扣款失敗或B 轉入失敗)，整個交易都會被取消，不會出現 A 扣款成功 B 卻沒有增加存款的情況
            > 
    2. **一致性 (Consistency)：**
        - 確保了資料庫的完整性，交易前後都必須維持資料的一致性
            
            > 例如：銀行規定每個帳戶的最低餘額必須是 500 元。A 的銀行帳戶裡有 600 元，他想從帳戶中提取 200 元。如果交易成功，則餘額將降至 400 元，違反了銀行的最低餘額政策，也違反了資料庫的一致性規則，整個交易都會被取消。
            > 
    3. **隔離性 (Isolation)：**
        - 每個交易都像是在其自己的私有環境中運行，不受其他交易的影響
            
            > 例如：假設同一個帳戶，A 想要提款 500 元，B 想要存款 300 元，確保這兩個交易彼此獨立。可能在提款交易開始，存款交易將被放在等待狀態，等到提款交易完全結束，存款交易才會開始。避免提款交易在重新計算餘額前就被存款交易中斷，而出現非期望的結果。
            > 
    4. **持久性 (Durability)：**
        - 一旦交易被確認，交易的結果就是永久性的，即使系統故障也不會不見。
            
            > 例如：當 A 在銀行存入 1000 元後收到確認，即使銀行系統突然故障，持久性保證當系統恢復時，她的 1000 元仍安全地存在帳戶中。
            > 
            

## 非關聯型資料庫 (NoSQL)

![https://ithelp.ithome.com.tw/upload/images/20230923/20152073AEV8Slgk81.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073AEV8Slgk81.png)

NoSQL，意為 "not only SQL"（不僅僅是 SQL）

1. **產品**：MongoDB, Cassandra, Redis
2. **彈性的資料結構**：可以儲存結構多變的資料，可以簡單的修改資料結構
3. **水平擴展性**：通過增加更多的服務器或節點來應對更大的數據負載
4. **快速讀寫操作**：高效地處理大量並發請求，滿足現代應用的高流量和即時反應需求。
5. **多種資料模型**：有不同種類的 NoSQL 資料庫，**文件型資料庫**、鍵值存儲、列存儲和圖形資料庫，各自適合不同的應用需求。
    1. **鍵-值資料庫（Key-Value）**:
        - **例子**: Redis, Oracle NoSQL Database
        - **描述**: 採用鍵值對方式儲存資料，其中每個鍵均為唯一，與其對應的值可以是任意型態的物件。具有出色的可分割性，並能支持大型的橫向擴展。
    2. **文件型資料庫（Document-oriented）**:
        - **例子**: MongoDB, CouchDB
        - **描述**: 以文檔( XML、YAML、JSON 和 BSON )格式儲存於資料庫。讓開發者能使用與應用程式中相同的文件模型來輕易地操作資料。適合目錄、用戶配置和內容管理系統情景。
    3. **圖形資料庫（Graph）**:
        - **例子**: Neo4j, OrientDB
        - **描述**: 這類資料庫專為高度互聯的資料結構設計，如社交網絡。可以查詢資料和它的關係，而不僅僅是單一的數據項目。
    4. **內存資料庫（In-Memory）**:
        - **例子**: Redis, Memcached
        - **描述**: 直接在記憶體中存儲數據，而非硬碟上，從而確保更快的反應時間且避免了硬碟存取。但在系統故障時，可能存在資料丟失的風險。提供了極快的讀取和寫入速度，適合於需要高速存取，但資料持久性要求較低的場景。
    5. **搜尋引擎資料庫（Search-Engine）**:
        - **例子**: Elasticsearch
        - **描述**: 這類資料庫主要提供高效的全文搜尋和分析功能。通常提供先進的查詢語法和資料索引功能，以支持復雜的搜尋需求。

> 既然關聯式資料庫 (SQL)具有這麼高安全性及資料完整性的特性，並且嚴謹的系統，為什麼我要選擇 NoSQL 又選擇其中的 MongoDB 呢？
> 

## 抉擇的 Q&A 時間

A. 我的資料結構是否已經固定下來？

Q. 在初期開發階段，需要調整許多資料結構，未來也預計有變動。

A. 我的資料中是否存在很多的關聯性和多表關聯查詢？

Q. 目前皆為由 user schema 與其他 schema 有關聯性，未來不排出新增其他關聯。

A. 我是否需要快速原型和靈活的資料 Schema？

Q. 是！

A. 是否有成本考量？

Q. 是！

A. 是否為 database 使用新手？

Q. 是！

A. 目前專案的規模？

Q. 使用於 Side Project

## 我選擇的是？

基於以上的考量，我選擇了非關聯式資料庫的 **MongoDB** 作為專案的資料庫。由於我的專案規模不大，且我使用 Next.js 作為前後端的開發工具，這使得搭配 MongoDB 的開發過程變得直觀，尤其是當 MongoDB 使用類似於 JSON 的 BSON 格式進行資料操作。另外，我希望這個專案能供大眾自由點擊，並考慮未來加入用戶行為分析功能。因此，需要一個既高效又能迅速查詢的資料庫。考量所有這些因素，MongoDB 無疑是我最佳的選擇。

> 再來，由於我是 database 使用的菜鳥，又有專案的成本考量，這時候我選擇了 MongoDB Atlas
> 

# MongoDB Atlas

MongoDB Atlas 是 MongoDB 官方的雲端資料庫服務，支援在 Amazon Web Services (AWS)、Google Cloud Platform (GCP) 或 Microsoft Azure 上部署，從而提供低延遲的資料存取以增強用戶效能。這項服務同時確保資料的安全性，通過自動化備份。此外，它還裝備了監控和警報工具來維護資料庫的穩定運作。

最重要的是，**一個帳號可以申請一個免費 cluster！！**

接下來讓我們看看怎麼申請吧！

### 註冊

網址：https://www.mongodb.com/
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073dcaQgFbG5U.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073dcaQgFbG5U.png)

### 設定

設定使用 Atlas 的目的以及主要的程式語言
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073qCoSiOyzP8.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073qCoSiOyzP8.png)

專案類型選擇，我自己是選擇【探索中】
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073Rs247B34on.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073Rs247B34on.png)
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073miCHtwfaSu.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073miCHtwfaSu.png)

程式語言選擇【JavaScript】
![https://ithelp.ithome.com.tw/upload/images/20230923/2015207354HHb4vO2e.png](https://ithelp.ithome.com.tw/upload/images/20230923/2015207354HHb4vO2e.png)

### 發佈 database

1. 視專案需求來選方案，免費仔當然是選免費！
2. **Provider**：選擇想要部署的供應商，三大供應商任你選
3. **Region**：選擇雲服務的地區，離你最近的地點 (GCP 有台灣可以選擇，其他兩者可以選香港)
4. **Name：cluster 的名稱，建立後就不可以更改了！**
5. 按下 Create 鍵！

![https://ithelp.ithome.com.tw/upload/images/20230923/20152073LTJ6ADt1ef.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073LTJ6ADt1ef.png)

### 設定 username 及 password

輸入 username 及 password 按下 `create user`
設定好後要記得牢牢記住，我們在下章節的專案這定中會需要用到他們！
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073F47s3NVPsv.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073F47s3NVPsv.png)

### 設定 IP Access List

在開發環境時，設定 `0.0.0.0/0` 允許任何來源以利開發，等待正式上線後再進行調整。
![https://ithelp.ithome.com.tw/upload/images/20230923/20152073gy0LmGAF4H.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073gy0LmGAF4H.png)

### 完成

可以在 Database 的頁籤查看建立的 cluster
![https://ithelp.ithome.com.tw/upload/images/20230923/201520732sWXEaBWAe.png](https://ithelp.ithome.com.tw/upload/images/20230923/201520732sWXEaBWAe.png)

# 結語

選擇合適的工具往往比設置本身更具挑戰性。當面臨「我應該選擇 SQL 還是 NoSQL？」這類問題時，背後涉及的不僅是技術選擇，更多的是根據應用程式的具體需求來做決策。有時，為了配合同一應用程式中的不同功能，甚至可以選擇混合使用多種資料庫。這其中的深度與細節遠超過本文所能涵蓋的範疇。但我希望對於初涉此領域、迷惘於選擇之中的讀者，這篇文章能提供一絲引路之光。在明天的章節，我們將了解未來應用程式中有高度關聯性的資料需求，以及如何在實際場景中配置資料庫。

## 相關文章
[營養師不開菜單的第九天 - 新世代 ORM 工具 Prisma 現學現賣](https://ithelp.ithome.com.tw/articles/10326491)

## 參考資料
是否該用 MongoDB？選擇資料庫前你該了解的事
https://tw.alphacamp.co/blog/mysql-and-mongodb-comparison
什麼是 NoSQL？
https://aws.amazon.com/tw/nosql/
MongoDB 是什麼？ MongoDB 優勢、適用場景、與 MySQL 的差異介紹
https://www.omniwaresoft.com.tw/product-news/mongodb-news/introduction-to-mongodb/

![https://ithelp.ithome.com.tw/upload/images/20230923/20152073wCSdwUhZFV.png](https://ithelp.ithome.com.tw/upload/images/20230923/20152073wCSdwUhZFV.png)