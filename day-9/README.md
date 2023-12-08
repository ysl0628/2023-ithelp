# 營養師不開菜單的第九天 - 新世代 ORM 工具 Prisma 現學現賣

![https://ithelp.ithome.com.tw/upload/images/20230924/201520735JHNGVJEid.png](https://ithelp.ithome.com.tw/upload/images/20230924/201520735JHNGVJEid.png)

Prisma 就像人體中的消化酵素 (enzyme)。就如消化酵素幫助我們分解和吸收食物中的營養，將大的複雜分子轉化為我們身體容易吸收的小分子，Prisma 也同樣地將複雜的資料庫操作轉化為易於使用和理解的查詢，並確保開發者能夠有效且安全地使用數據。Prisma 自動完成和編譯時的類型檢查功能就如同消化酶預先分解食物，確保我們獲得最佳的營養攝取。

# 什麼是 Prisma ?

![https://ithelp.ithome.com.tw/upload/images/20230924/20152073OYCeYye3Df.png](https://ithelp.ithome.com.tw/upload/images/20230924/20152073OYCeYye3Df.png)

Prisma 是一個新世代的開源 ORM 工具，包括的服務有bPrisma Client、Prisma Migrate、Prisma Studio：

1. **Prisma Client**：為 Node.js 和 TypeScript 自動生成且類型安全的查詢生成器，包括 REST API、GraphQL API、gRPC API 或需要資料庫的其他應用。
2. **Prisma Migrate**：一個資料庫遷移系統。
3. **Prisma Studio**：一個 GUI，用於查看和編輯資料庫中的數據。非開源產品，以整合為 Data Browser 在 Prisma 的數據視覺化及資料庫管理的平台 Prisma Data Platform

> ORM (Object-Relational Mapping，物件關聯對映) 技術，主要功能是建立物件導向程式語言和關聯性資料庫之間的橋樑。ORM 讓開發者更直觀地存取資料庫，避免直接撰寫 SQL 語句。儘管 ORM 提供了許多便利，但它也帶來了效能問題、複雜查詢的挑戰，以及相對較高的學習曲線。
> 

# 為什麼要用 Prisma?

![day9-1](https://www.prisma.io/docs/static/d42dccd02b6e0725c2cbcfd3f1590f15/42cbc/prisma-makes-devs-productive.png)
有鑒於傳統 ORM 的一些缺點，prisma 解決了這些瓶頸，並且使用類型安全的 API 來簡化查詢過程，該 API 返回普通的 JavaScript 物件。

以下關於 prisma 的主要想解決的問題：

1. **物件導向**：直接以物件的方式思考，而不是 ORM。
2. **簡潔的查詢**：使用直接的 queries 查詢，減少使用 class 複雜的模型物件。
3. **一致性**：確保資料庫和 models 之間的一致性。
4. **簡化的操作**：提供一個直觀的抽象，使得正確的操作變得簡單。
5. **類型安全**：所有的資料庫查詢都經過類型安全的檢查，且可以在編譯時驗證。
6. **減少冗余**：減少開發者的設定的程式碼。

> 另外在本專案中會使用 prisma 的重點是：支援 mongoDB
> 

Prisma 不只解決了我在未來擴增資料庫關聯性時可能遇到的複雜問題，還可以在享受 mongoDB 的優勢時，也能夠受益於關聯式資料庫的安全特性。這是因為Prisma為mongoDB提供了設定關聯性的方法，且支援簡潔且高效的查詢。

# 實作開始

---

prisma 的建置除了安裝及串聯資料庫，在後端主要的操作都是藉由 Prisma Client 進行，所以以下的步驟會先做安裝、初步的建置、連接資料庫、設置 model 再到 Prisma Client 設置及使用：

## 安裝

```
npm i -D prisma
```

## 安裝 Prisma CLI

```
npx prisma
```

## 初始化

```
npx prisma init
```

> 可自動產生設定文件 `prisma/schema.prisma` ，成功後會顯示以下訊息
> 

```
✔ Your Prisma schema was created at prisma/schema.prisma.
  You can now open it in your favorite editor.
```

![https://ithelp.ithome.com.tw/upload/images/20230924/20152073loVhs86UmR.png](https://ithelp.ithome.com.tw/upload/images/20230924/20152073loVhs86UmR.png)

> 在你的資料夾目錄下會多一個 prisma 的資料夾，當中會有 `schema.prisma` 的設定檔
> 

![https://ithelp.ithome.com.tw/upload/images/20230924/20152073kT6zTzmu3J.png](https://ithelp.ithome.com.tw/upload/images/20230924/20152073kT6zTzmu3J.png)

> `schema.prisma` 中會顯示以下內容，這邊特別注意如果程式碼沒有 syntax highlighting，可能尚未安裝 prisma plugin!，安裝後可以提供語法建議、型別檢查、格式化及錯誤提示等功能。
https://marketplace.visualstudio.com/items?itemName=Prisma.prisma
> 

```
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

## 連結 MongoDB

> 此步驟需要先替換 provider 類型再將 mongoDB 的連結碼設置在環境變數中
> 

### 1. 替換 `schema.prisma` 中的 provider 類型

```
datasource db {
  provider = "mongodb" // 將原本的 postgresql 改為 mongodb
  url      = env("DATABASE_URL")
}
```

### 2. 取得 mongoDB 的連結碼

先進入 atlas 的後台，點選 database 頁籤，再點選 connect 按鈕

![https://ithelp.ithome.com.tw/upload/images/20230924/20152073ypc3pxLADG.png](https://ithelp.ithome.com.tw/upload/images/20230924/20152073ypc3pxLADG.png)

選擇 **MongoDB for VS Code**

![https://ithelp.ithome.com.tw/upload/images/20230924/201520732z7HmDtdJo.png](https://ithelp.ithome.com.tw/upload/images/20230924/201520732z7HmDtdJo.png)

將 **3. Connect to your MongoDB deployment. 提供的連結 URL 複製下來**

```
mongodb+srv://<username>:<password>@******.*****.mongodb.net/
```

### 3. 將連結 URL 設置於目錄下的 .env 檔中

> `schema.prisma` 的 datasource db 設定中，需要取得環境變數 `env("DATABASE_URL")`
請記得將 `<username>:<password>` 改為你當初設定的 user，並且在最後的 `’/’` 後加上 `test`
> 

```
DATABASE_URL='mongodb+srv://<username>:<password>@******.*****.mongodb.net/test'
```

## 建立 Schema

> `schema.prisma` 檔案中，接續著定義專案所需的 model，model 中定義各個欄位及型別，也可以額外定義欄位的屬性、enum、type、relation
> 
- id：mongoDB 中的必有欄位，以下介紹定義的屬性
    - `@id` - 定義單一欄位的 ID
    - `@default(auto())` - 定義 ID 的類型，另有 `uuid()` 及 `cuid()` 可選
    - `@db.ObjectId` - 必須定義的屬性，可以確保 Prisma 正確地識別出哪些欄位是 **`ObjectId`** 類型
    - `@map("_id")` - 映射到資料庫的欄位名稱
- `@unique` - 代表只能有唯一值
- `@@unique([fieldA,fieldB])` - array 中兩個欄位的組合必須為唯一
- `@updatedAt` - 自動記錄資料更新時間的功能
- `@relation` - 建立起關聯
    - `fields` 是指 Account 的 userId，`references` 是指 User 的 id
        
        > 代表 Account 的 `userId` 會參照 User 的 `id`
        > 
    - `onDelete: Cascade` - 將 User 的某 id 刪除時，一併刪除 Account 中關聯的紀錄
- 也可以自訂義 enum 或 type - 例如下方的 `type LinkType`

```
generator client {
 ...
}

datasource db {
  ...
}

model User {
  id             String    @id @default(auto()) @map("_id") @db.ObjectId
  name           String?
  username       String?   @unique
  email          String?   @unique
  emailVerified  DateTime?
  hashedPassword String?
  createdAt      DateTime  @default(now())
  updatedAt      DateTime  @updatedAt
  image          String?
  customImage    String?
  title          String?
  description    String?
  themeColor     String?

  accounts Account[]
  links    Link[]
}

model Account {
	...

	user User @relation(fields: [userId], references: [id], onDelete: Cascade)

	@@unique([provider, providerAccountId])
}

model Link {
	...
	type      LinkType
	...
	user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

type LinkType {
  id    String
  label String
}

// 其他 model

```

## 安裝及生成 Prisma Client

一鍵式幫你生成 Prisma Client。使用 Prisma Client，可以執行 CRUD 操作、執行原始 SQL 查詢、使用過濾器和排序、操作關聯資料等

```
npm install @prisma/client
```

> 當執行上面指令後會自動調用 `prisma generate` 指令來依據剛剛設定的 `schema.prisma` 檔案生成 prisma client
> 

![install-prisma](https://www.prisma.io/docs/static/c24add4ac2d8984ecc6846f54a92a318/d880f/prisma-client-install-and-generate.png)

## 封裝 Prisma Client 功能

> 為了提高開發時的效率和減少每次執行資料庫操作時都要建立一個新的 client，將功能封裝起來
> 

### 原本方法

每次都要創建一個新的 client

```
// route handler
const { PrismaClient } = require('@prisma/client/edge')

const prisma = new PrismaClient()

export async function POST(request: Request) {
	const body = await request.json()
  const { email, username, password } = body
	...
	const user = await prisma.user.create({
	      data: {
	        email,
	        username,
	        password 
	      }
	    })

	return NextResponse.json(user)
}
```

### 封裝後

重複使用同一個 Prisma Client

```
// libs/prismadb

import { PrismaClient } from '@prisma/client'

declare global {
  var prisma: PrismaClient | undefined
}

const client = globalThis.prisma || new PrismaClient()
if (process.env.NODE_ENV !== 'production') globalThis.prisma = client

export default client

// route handler
import prisma from '@/libs/prismadb'

export async function POST(request: Request) {
	const body = await request.json()
  const { email, username, password } = body
	...
	const user = await prisma.user.create({
	      data: {
	        email,
	        username,
	        password 
	      }
	    })

	return NextResponse.json(user)
}
```

## ORM 語法

封裝好 client 後，就可以直接以 client 提供的 ORM 語法操作資料庫，基本調用 prisma 後再選擇要操控的 Collection，之後再調用操作方法

```
const data = prisma.<操作的Collection>.<要操作的方法>
```

以下就從簡單的增刪改查介紹：

### 新增

新增的方法使以 `create` 來建立一筆新的資料，參數中帶入以 data 為 key 的物件，下方範例中為以 email 、username、hashedPassword 欄位新增一個 user。

```
const user = await prisma.user.create({
  data: {
    email,
    username,
    hashedPassword,
  },
});
```

### 新增多個

新增多筆資料可以使用 `createMany` 語法，如果要將重複的資料忽略則可以加上 `skipDuplicates` 屬性：

```
const user = await prisma.user.create({
  data: [
    { email: "email@eamil.com", username: "emailman" },
    { email: "email111@eamil.com", username: "emailman" }, // username 重複了
    { email: "email222@eamil.com", username: "emailsir" },
  ],
  skipDuplicates: true, // 會忽略 'email111@eamil.com'
});
```

### 查詢單筆

查詢單筆資料使用的是 `findUnique` ，並搭配 `where` 方法指定搜尋條件：

```
const currentUser = await prisma.user.findUnique({
  where: {
    email: session.user.email as string
  }
});
```

### 查詢多筆

查詢多筆資料則使用 `findMany` ，以及 `where` 方法指定搜尋條件，也可以 `orderBy` 指定回傳的排序方法，範例為以 order 欄位進行降冪排序：

```
const links = await prisma.link.findMany({
  where: {
    userId: user.id,
  },
  orderBy: {
    order: "desc",
  },
});
```

### 更新資料

更新方法使用的是 `update` ，指定特定的單筆資料，並更新資料中的欄位 (可以多個欄位)：

```
const updatedLink = await prisma.link.update({
  where: {
    id: link.id,
  },
  data: {
    order: link.order,
  },
});
```

### 刪除資料

而刪除資料使用 delete，並且只要指定唯一欄位，即可刪除該筆資料：

```
const data = await prisma.link.delete({
  where: {
    id: linkId,
  },
});
```

# Prisma 結合 MongoDB 與其他資料庫有什麼不同？

有於 Prisma 的設計是以關聯式資料庫的方式為主要目的，在與 MongoDB 結合時雖然可以使用 SQL 的優勢，但在設定上會有些小不同：

1. 定義 ID：MongoDB 文檔具有 `_id` 字段，Prisma 支援以 `_` 開頭的 string，需要使用 `@map` 屬性進行映射。
    
    > `@map` 是可以讓開發者在 prisma 使用不同的名稱而不改變資料庫實際的 column name，只在 Prisma 和生成的 Prisma Client 中建立映射或別名，當你使用 Prisma Client 時，將按照 Prisma modal（及使用了`@map`的名稱）來操作，而不是資料庫中的原始名稱。
    > 
    
    ```
    // schema
    model User {
      id String @default(auto()) @map("_id") @db.ObjectId
    }
    
    // route handler
    const data = await prisma.user.update({
        where: {
          id: params.id
        },
    		...
    })
    ```
    
2. Migrate：因為 MongoDB 的彈性結構設計與傳統的關聯式資料庫不同，資料更新遷移需要特別注意。以下有幾個防範更新前後資料不一致的狀況
    - **按需更新**：新添加的任何欄位都明確地被定義為可選的或是設置預設值。
        
        ```
        model User {
          id          String  @id @default(auto()) @map("_id") @db.ObjectId
          email       String
          phoneNumber String?
        //phoneNumber String @default("000-000-0000")
        }
        ```
        
    - **無破壞性更新**：基於前一種更新方法，不更改原有的欄位，也不刪除原有的欄位，只是新增欄位
    - **一次性更新**：在添加或修改欄位之前，需要先對資料進行 migrate，以確保資料符合新結構。例如，添加一個新的必填欄位之前，需要在資料庫中為所有現有記錄添加此欄位的值。但這個方法可能需要一些前置作業，包括編寫 migrate 的 script 及測試
    - 更新資料庫的方法：當 model 有異動時，執行以下的命令：
        
        ```
        prisma db push
        ```
        
3. 過濾 null 值與缺漏的欄位：在 SQL 的世界中一個欄位必須要有值或是設置為 null，而 mongoDB 中除了這兩者還可以完全不設定值，直接缺漏的狀態，但是 prisma 目前不支援這個狀態的過濾，所以當要過濾 null 或 null 與缺漏的欄位時：
    - 直接使用 **`{ fieldName: null }`** 作為過濾條件
        
        ```
        const findNulls = await prisma.user.findMany({
          where: {
            name: null,
          },
        })
        ```
        
    - 使用 **`OR`** 操作並結合 **`isSet`** 過濾器
        
        ```
        const findNullOrMissing = await prisma.user.findMany({
          where: {
            OR: [
              {
                name: null,
              },
              {
                name: {
                  isSet: false,
                },
              },
            ],
          },
        })
        ```
        

# 結語

Prisma 讓開發者能夠更安全地進行數據操作，大大減少了相關的麻煩。不僅支援關聯式資料庫，Prisma 現在也擴展到了 MongoDB 的支援。這對於數據庫使用者而言無疑是一大福音。它不僅繼承了 MongoDB 的優勢，還在關聯式資料庫的安全性和一致性上提供了強大的操作支持。果然是大人的選擇，我全都要~

明天我們先跳脫資料庫，來進入前端畫面的 UI 庫介紹使用吧!

## 相關文章
[營養師不開菜單的第八天 - 為什麼要用 MongoDB](https://ithelp.ithome.com.tw/articles/10325586)
## 參考資料
Should you use Prisma?
https://www.prisma.io/docs/concepts/overview/should-you-use-prisma
後端工程師的第一堂課 (20) : 現代系統資料工具 — ORM
[https://medium.com/johnliu-的軟體工程思維/後端工程師的第一堂課-20-現代系統資料工具-orm-359da9a1d14a](https://medium.com/johnliu-%E7%9A%84%E8%BB%9F%E9%AB%94%E5%B7%A5%E7%A8%8B%E6%80%9D%E7%B6%AD/%E5%BE%8C%E7%AB%AF%E5%B7%A5%E7%A8%8B%E5%B8%AB%E7%9A%84%E7%AC%AC%E4%B8%80%E5%A0%82%E8%AA%B2-20-%E7%8F%BE%E4%BB%A3%E7%B3%BB%E7%B5%B1%E8%B3%87%E6%96%99%E5%B7%A5%E5%85%B7-orm-359da9a1d14a)
Introspection
https://www.prisma.io/docs/concepts/components/introspection
How to use Prisma with MongoDB
https://www.prisma.io/docs/guides/database/mongodb#how-to-add-in-missing-relations-after-introspection
Relations
https://www.prisma.io/docs/concepts/components/prisma-schema/relations

![https://ithelp.ithome.com.tw/upload/images/20230924/201520735kvD1RaGst.png](https://ithelp.ithome.com.tw/upload/images/20230924/201520735kvD1RaGst.png)