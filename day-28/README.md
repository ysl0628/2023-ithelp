
iT邦幫忙全新系列【數位轉型攻略 V：戰略新 IT】租虛擬主機架站，再附信箱MWC揭開軟體開發新紀元！
iT邦幫忙
技術問答
技術文章
iT 徵才
Tag
聊天室
2023 鐵人賽
鐵人發文
發問
發文 
技術
徵才
ysl0628
我的主頁
我的發問
我的回答
我的發文
追蹤的問答
追蹤的文章
追蹤的tag
追蹤的邦友
邀請我回答
設定
找邦友
登出
今天挑戰第 21 天，加油！
SideProject30 
營養師不開菜單要用 Next.js 13 寫全端
團隊所以隊名要叫什麼 | 開賽日：2023-09-16
營養師不開菜單的第二十八天 - 跟上容器化技術，Docker 的建置
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073oN4E0EQTBn.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073oN4E0EQTBn.png)
# 關於容器化
​
在科技迅速發展的今日，部署的技術也經歷了不少的變革。從早期的物理伺服器到現代的容器化技術，每個階段都是為了追求更高效率和更好的資源利用率。
​
在**傳統部署時代**，應用程式主要在單一物理伺服器上運行，但資源分配困難，常造成資源浪費以及高維護成本。於是逐漸引入了**虛擬化技術**，允許一台伺服器運行多個虛擬機(VM)，每個 VM 都運行一套完整的操作系統，雖有安全的隔離性，也提高了資源利用率以及彈性，但仍存在部分資源浪費。隨後，**容器技術**出現，例如本篇中使用的 Docker，允許更輕量級的部署。容器共享操作系統，僅安裝所需應用，節省資源。容器的好處包括快速部署、跨平台一致性、支援微服務架構、資源隔離，以及高效的資源利用。
​
# Docker 簡介
​
對於剛接觸容器化的前端工程師，由於直觀操作和豐富的網路教學資源，所以在這個專案中我們使用 Docker 進行建置。Docker 不僅簡化了環境配置，降低了新手的學習曲線，而且 Docker Hub 上的專用映像使前端開發更加迅速。
​
在 Docker 的世界裡，有三個核心元素，分別是：**鏡像 (Image)**、**容器 (Container)**、和**倉庫 (Repository)**。以下是對這三個核心元素的簡短介紹：
​
1. **鏡像 (Image)**:
    
    Image 是一個輕量級、獨立、可執行的檔案，集合了應用程式的全部資源，例如程式碼、運行時的環境、套件、環境變數和設定文件。一旦建立，就不會改變的 read only 檔。並且提供了一致性，確保在不同的部署環境中，應用程式都能以相同的方式運行。作為「模板」供創建容器使用。
    
2. **容器 (Container)**:
    
    容器是 Docker Image 的運行實例。提供了獨立的執行環境，隔離自其他容器。可以快速啟動、運行、停止和刪除。每個容器都有自己的文件系統、網路和獨立的進程空間。開發者能夠在相同的環境中開發、測試和運行應用程式。如果說 Image 是食譜，Container 就是依據這個食譜運行起這個應用程式。
    
3. **倉庫 (Repository)**:
    
    Repository 是存放 Docker Image 的地方。它可以是公開的或私有的，並允許用戶 push 和 pull Image。可以讓開發者輕鬆地分享和分發他們的應用程式 Image，並在不同的環境或團隊間重複使用，以及確保所有人都使用最新版本。
    
​
# Docker 建置實作
​
接下來就讓我們從下載安裝、建立 Dockerfile 再到運行進入實作階段。
​
## 下載安裝
​
在工作環境中安裝 Docker。無論是在本地開發機、伺服器或雲端環境，都需要先有 Docker 軟體。
​
首先在官網下載 `docker desktop` ：https://docs.docker.com/desktop/install/mac-install/ (本人開發以 Mac 為例)
​
安裝啟動後會是這樣的應用程式介面
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073u7PYgsjGZp.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073u7PYgsjGZp.png)
​
## 建立 Dockerfile
​
Dockerfile 是一個用於自動化生成 Docker Image 的檔案。透過 Dockerfile 可以定義一個鏡像的建立過程，包括 base image 的選擇、所需的軟體套件安裝、檔案的複製或新增、環境變數設定、暴露的端口等等。
​
- 根目錄新增一個 dockerfile
- 指定這個專案的 base image
    
    建立新容器的起始 image。通常是一個最小化的系統環境，只包含運行某些特定應用或服務所需的最基本元件和工具。本專案是以 node.js 建置，可以使用官方提供的 [Node.js image](https://hub.docker.com/_/node/)，已經內建了所有運行 Node.js 應用所需的工具和套件。
    
    > alpine 是基於輕量級的 Alpine Linux 而製作的 Node.js image，大小較小，通常只有數十 MB。
    > 
    
    ```docker
    FROM node:18-alpine
    ```
    
- 設定環境變數
    
    ```docker
    ENV NODE_ENV=development
    ```
    
- 在 container 建立名為 app 的根目錄，若該目錄不存在，Docker 會自動建立此目錄。後續的指令（例如 `COPY` 或 `RUN`）將在此目錄中執行。
    
    ```docker
    WORKDIR /app
    ```
    
- 將本地端所有檔案複製到 container 的 app 目錄下
    
    > 在運行 `npm install` 前，我們需要將 `package.json` 和 `package-lock.json` 兩個檔案，或是直接將所有目錄放入 Docker image 中。
    > 
    
    ```docker
    COPY ["<src>", "<dest>"]
    
    # 如果要複製很多個來源
    COPY ["<src1>", "<src2>",..., "<dest>"]
    
    # 範例
    COPY . .
    ```
    
- 執行 `npm install` 指令，用以安裝專案所需的套件
    
    ```docker
    RUN npm install
    ```
    
- 容器對外開放的埠號，告訴 Docker 該 container 內有一個應用程式會監聽 3000 port，因此該 port 可能需要開放給外部連接。
    
    ```docker
    EXPOSE 3000
    ```
    
- container 啟動後要執行的指令
    
    ```docker
    CMD ["npm", "run", "dev"]
    ```
    
​
## 建立 `.dockerignore` 檔案
​
當 image 執行到 `COPY . .` 會將本地根目錄中的檔案全部複製到 container 中 app 底下，其中也包括 node_modules 檔案，但本地的 node_modules 容易被更改內容導致在本地中可以運行，上到 container 中時卻出現 error。所以為了安全性及穩定性，會建立 `.dockerignore` 將 node_modules 在複製階段忽略，直接在 container 中完全地重新下載。
​
```docker
node_modules
```
​
## 執行 Docker 指令
​
建置完基本的 `Dockerfile` 內容，接下來可以執行一些指令將 Image build 起來！！
​
### 建構一個 Image - `docker build`
​
Docker 會尋找當前目錄下的 `Dockerfile` 並根據其內容來建構 Docker Image，並指定 `-t` 或 `--tag` 來為這個鏡像命名。
​
```bash
docker build -t <image name> .
# 例如：
docker build -t link-orchard .
```
​
執行過程
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073oK6hnXucwX.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073oK6hnXucwX.png)
​
建立成功後，`docker desktop` 中會顯示該 Image
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073FEJcjFEWIV.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073FEJcjFEWIV.png)
​
### 移除一個 Image - `docker rmi`
​
移除本地機器上的 Docker Image，只要該 Image 沒有被任何 container 使用，命令會成功移除。如果有 container 仍然使用該 Image，則移除操作將會失敗。
​
```bash
# 移除 image
docker rmi link-orchard
```
​
### 啟動一個 Image 作為 container - `docker run`
​
```bash
docker run <image name>
​
# 發佈暴露的 port 號
# docker run --publish <本地 port>:<容器暴露的 port> <image name>
docker run -p 3000:3000 link-orchard
​
# 在背景中執行 加上 -d 或是 --detached
docker run -d -p 3000:3000 link-orchard
​
```
​
啟動後會在 `docker desktop` 中 Container 頁籤顯示
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073Fxz7DsHiR2.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073Fxz7DsHiR2.png)
​
### 查看目前運作的 container - `docker ps`
​
列出 container 的 ID、創建時間、狀態、名稱等相關資訊。
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073uHrLTchbxS.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073uHrLTchbxS.png)
​
### 停止 container - `docker stop`
​
以上方 `docker ps` 查詢解果的 `NAMES` 操作
​
```bash
docker stop <NAMES>
# 例如
docker stop adoring_wozniak
```
​
### 移除 container - `docker rm`
​
```bash
docker rm <NAMES>
# 例如
docker rm adoring_wozniak
​
# 可一次移除多個
docker rm <NAMES> <NAMES> <NAMES>
```
​
## 建立 docker-compose.yaml
​
雖然使用 `Dockerfile` 可以滿足大部分的需求，但當你的應用程式涉及多個容器協同工作時，例如前端、後端、資料庫、緩存等 server 各自在一個容器中運行，`docker-compose` 就提供了一個更統一、組織化的方式來管理和運行容器。
​
> `docker-compose` 使用 `docker-compose.yaml`（或 `.yml`）檔案來描述和執行**多容器** Docker 應用。
> 
​
以下為 docker-compose 中可以建立的屬性
​
- version：指定了 Compose 文件的版本。在這個例子中，使用的是 3.9 版本。
- services：定義了所有要運行的容器服務。每個服務都會從一個 Docker 鏡像創建一個容器。在這個例子中，只定義了名為 `link-orchard-dev` 和 `link-orchard-prod` 的 server。
- build：告訴 Docker 從哪裡獲取 `Dockerfile` 來構建鏡像，可以是相對路徑或是一個 URL。在這個例子中，使用當前目錄下的 `Dockerfile`。
- ports：容器內的端口映射到主機上的端口。在 `link-orchard-dev` 中，將容器的 4000 端口映射到主機的 3000 端口上。
- volumes：允許容器與主機之間共享文件或目錄。`link-orchard-dev` 中，將主機中的根目錄資料夾掛載到容器中的 "/app" 路徑上，從而實現數據持久化。
- profiles：若有設定，在指定環境變數下才會啟用，例如：dev 環境下 `link-orchard-dev` 才會啟用，`link-orchard-prod` 則在 prod 環境下啟動。
- image：為 server 指定或命名 Docker Image，未設定標籤時默認使用 `latest`。
​
### 根目錄建立 `docker-compose.yaml`
​
### 範例檔
​
```yaml
version: "3.8"
​
services:
  link-orchard-dev:
    build: .
    ports:
      - "4000:3000"
    volumes:
      - ./:/app
    profiles: ["dev"]
    
  link-orchard-prod:
    build: 
      context: .
      dockerfile: dockerfile.prod 
    ports:
      - "80:80"
    profiles: ["prod"]
    
  project-image:
    image: ysl0628/link-orchard:v1
    ports:
      - "3300:80"
    profiles: ["prod"]
```
​
### 以 docker-compose 啟動 container
​
```bash
docker compose -d up
```
​
如果要指定特定的 profiles 可以下這個指令：此指令只會啟動 `link-orchard-dev` server
​
```bash
docker compose --profile dev up
```
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073vlSY8Ce6jA.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073vlSY8Ce6jA.png)
​
### 停止 container
​
```bash
docker compose down
```
​
### 啟動完成
​
當頁面輸入 `http://127.0.0.1:4000/` 時成功啟動專案
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073ZAyilATmeo.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073ZAyilATmeo.png)
​
# 結語
​
自己在剛接觸 docker 時，對於容器化的概念一知半解，有大半的實作都是按照說明書填鴨式寫法，在經過多次的實作後，逐漸發現 Docker 的真正威力和便利性。容器化不僅僅是將應用程式打包，更重要的是能確保應用在任何環境下都能一致運作。透過 Docker 可以快速地部署、更新和複製我的應用，也不用擔心「只有在我的電腦上可以運行」的問題。透過這篇文章，希望可以為容器化小白從容器化的演進到基礎的操作啟動，更快地上手和體驗到容器化技術帶來的便利性和高效性。
​
## 參考資料
​
https://ithelp.ithome.com.tw/articles/10287154
​
https://docs.docker.com/compose/compose-file/03-compose-file/
​
https://ithelp.ithome.com.tw/articles/10192824
​
![https://ithelp.ithome.com.tw/upload/images/20231005/20152073loI5QO0EER.png](https://ithelp.ithome.com.tw/upload/images/20231005/20152073loI5QO0EER.png)