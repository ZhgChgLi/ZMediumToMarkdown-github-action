---
title: Converting Medium Posts to Markdown
author: ZhgChgLi
date: 2022-05-28T07:04:35.424Z
tags: [medium,markdown,backup,ruby,automation]
---

### Converting Medium Posts to Markdown

撰寫小工具將 Medium 心血文章備份下來 & 轉換成 Markdown 格式
![](images/ddd88a84e177/1*Widc44swFkytb1jRNhA6Lg.jpeg "")
### [EN] [ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)


I’ve written a project to let you download Medium post and convert it to markdown format easily.
#### Features
- Support download post and convert to markdown format
- Support download all posts and convert to markdown format from any user without login access.
- Support command line interface
- Download all of post’s images to local and convert to local path
- Convert [Gist](https://gist.github.com/) source code to markdown code block

- Convert youtube link which embed in post to preview image
- Adjust post’s last modification date from Medium to the local downloaded markdown file
- Auto skip when post has been downloaded and last modification date from Medium doesn’t changed (convenient for auto-sync or auto-backup service, to save server’s bandwidth and execution time)
- Highly optimized markdown format for Medium


[GitHub - ZhgChgLi/ZMediumToMarkdown
ZMediumToMarkdown lets you download Medium post and convert it to markdown format easily. This project can help you to…github.com](https://github.com/ZhgChgLi/ZMediumToMarkdown)
### [CH] [ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)


可針對 Medium 文章連結、Medium 使用者的所有文章，爬取其內容並轉換成 Markdwon 格式連同文章內圖片一同下載下來的備份小工具。
#### 特色功能
- 免登入、免特殊權限
- 支援單篇文章、使用者所有文章下載並轉換成 Markdown
- 支援下載備份文章內的所有圖片並轉換成對應圖片路徑
- 支援深度解析內嵌於文章中的 [Gist](https://gist.github.com/) 並轉換成相對語言的 Markdown Code Block

- 支援解析內嵌於文章中的 Youtube 影片，將轉換成影片預覽圖及連結顯示於 Markdown
- 使用者所有文章下載時會去掃描文章內有無嵌入關聯文章，有的話會將連結替換為本地
- 針對 Medium 格式樣式特別優化
- 自動將下載下來文章的最後修改/建立時間，更改為同 Medium 文章發佈時間
- 自動比對下載下來的文章最後修改，如果沒有小於 Medium 文章最後修改時間時則自動跳過
(方便大家使用此工具建立自動 Sync/Backup 工具，此機制能節省 server 流量/時間)
- CLI 操作，支援自動化


[GitHub - ZhgChgLi/ZMediumToMarkdown
ZMediumToMarkdown lets you download Medium post and convert it to markdown format easily. This project can help you to…github.com](https://github.com/ZhgChgLi/ZMediumToMarkdown)
> **_本項目及本篇文章僅供技術研究，請勿用於任何商業用途，請勿用於非法用途，如有任何人憑此做何非法事情，均於作者無關，特此聲明。  
> 請確認您有文章使用、著作權再行下載備份。_**  
### 起源

經營 Medium 第三年，已累積發表超過 65 篇文章；所有文章都是我直接使用 Medium 平台撰寫，沒有其他備份；老實說一直很怕 Medium 平台有狀況或是其他因素導致這幾年的心血結晶消失。

之前曾經手動備份過，非常無聊且浪費時間，所以一直在找尋一個可以自動把所有文章備份下載下來的工具、最好還能轉換成 Markdown 格式。
### 備份需求
- Markdown 格式
- 依照 User 能自動下載該 User 的所有 Medium Posts
- 文章圖片也要能被下載備份下來
- 要能 Parse Gist 成 Markdown Code Block
(我的 Medium 大量使用 gist 嵌入 Source Code 所以這個功能很重要)

### 備份方案
#### Medium 官方

官方雖然有提供匯出備份功能，但匯出格式僅能用於匯入 Medum、非 Markdown 或共通格式，而且不會處理 [Github Gitst](https://gist.github.com/) …等等 Embed 的內容。


Medium 提供的 [API](https://github.com/Medium/medium-api-docs) 沒什麼在維護且只提供 Create Post 功能。

>  **_合理，因為 Medium 官方不希望使用者能輕易地將內容轉移至其他平台。_** 
#### Chrome Extension

有找到試用了幾個 Chrome Extension (幾乎都被下架了)，效果不好，一是要手動一篇文章一篇文章點進去備份、二是 Parse 出來的格式很多錯誤而且也無法深度 Parse Gist Source Code 出來、也無法備份文章的所有圖片下來。
#### [medium-to-markdown command line](https://www.npmjs.com/package/medium-to-markdown)

某位大神用 js 寫的，能達成基本的下載及轉換成 Markdown 功能，但一樣沒圖片備份、深度 Parse Gist Source Code。
#### [ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)

苦無完美解決方案後，下定決心自行撰寫一個備份轉換工具；花費了大約三週的下班時間使用 Ruby 完成。
#### 技術細節

 **如何透過輸入使用者名稱得到文章列表？** 

1.取得 UserID：檢視使用者主頁(https://medium.com/@#{username}) 原始碼可以找到 `Username` 對應的 `UserID`[這邊要注意因為 Meidum 重新開放自訂網域](medium-%E8%87%AA%E8%A8%82%E7%B6%B2%E5%9F%9F%E5%8A%9F%E8%83%BD%E5%9B%9E%E6%AD%B8-d9a95d4224ea) 所以要多處理 30X 轉址


2.嗅探網路請求可以發現 Medium 使用 GraphQL 去取得主頁的文章列表資訊

3.複製 Query & 替換 UserID 到請求資訊
```
query = [{  
"operationName": "UserProfileQuery",  
"variables": {  
"homepagePostsFrom": **homepagePostsFrom** ,  
"includeDistributedResponses": true,  
"id": **userID** ,  
"homepagePostsLimit": 10  
},  
"query": "query Us...."  
}]


```

4.取得 Response

每次只能拿 10 筆，要分頁拿取。
- 文章列表：可以在 `result[0]->userResult->homepagePostsConnection->posts` 中取得

- `homepagePostsFrom` 分頁資訊 ：可以在 `result[0]->userResult->homepagePostsConnection->pagingInfo->next` 中取得  
將 `homepagePostsFrom` 帶入請求即可進行分頁存取，`nil` 時代表已沒有下一頁


 **如何剖析文章內容？** 

檢視文章原始碼後可發現，Medium 是使用 [Apollo Client](https://www.apollographql.com/docs/react/) 服務進行架設；其端 HTML 實際是從 JS 渲染而來；因此可以再檢視原始碼中的 \<script\> 區段找到 `window. __APOLLO_STATE__ ` 字段，內容就是整篇文章的段落架構，Medium 會把你整篇文章拆成一句一句的段落，再透過 JS 引擎渲染回 HTML。

![](images/ddd88a84e177/1*mH8iq7W-pJZrMBPpEyN6Zw.png "")

我們要做的事也一樣，解析這個 JSON，比對 Type 在 Markdown 的樣式，組合出 Markdown 格式。
#### 技術難點

這邊有一個技術困難點就是在渲染段落文字樣式時，Medium 給的結構如下：
```JSON
"Paragraph": {
    "text": "code in text, and link in text, and ZhgChgLi, and bold, and I, only i",
    "markups": [
      {
        "type": "CODE",
        "start": 5,
        "end": 7
      },
      {
        "start": 18,
        "end": 22,
        "href": "http://zhgchg.li",
        "type": "LINK"
      },
      {
        "type": "STRONG",
        "start": 50,
        "end": 63
      },
      {
        "type": "EM",
        "start": 55,
        "end": 69
      }
    ]
}
```

意思是 `code in text, and link in text, and ZhgChgLi, and bold, and I, only i` 這段文字的:

```
- 第 5 到第 7 字元要標示為 Code (用`Text`格式包裝)  
- 第 18 到第 22 字元要標示為 URL (用[Text](URL)格式包裝)  
- 第 50 到第 63 字元要標示為 Code (用\*Text\*格式包裝)  
- 第 55 到第 69 字元要標示為 Code (用\_Text\_格式包裝)


```

第 5 到 7 & 18 到 22 在這個例子裡好處理，因為沒有交錯到；但 50–63 & 55–69 會有交錯問題，Markdown 無法用以下交錯方式表示：
```
code `in` text, and [ink](http://zhgchg.li) in text, and ZhgChgLi, and \*\*bold,\_ and I, \*\*only i\_


```

正確的組合結果如下：
```
code `in` text, and [ink](http://zhgchg.li) in text, and ZhgChgLi, and \*\*bold,\_ and I, \_\*\*\_only i\_


```

50–55 STRONG 55–63 STRONG, EM 63–69 EM

另外要需注意：
- 包裝格式的字串頭跟尾要能區別，Strong 只是剛好頭跟尾都是 `**`，如果是 Link 頭會是 `[` 尾則是 `](URL)`

- Markdown 符號與字串結合時要注意前後不能有空白，否則會失效


[完整問題請看此。](https://gist.github.com/zhgchgli0718/e8a91e81053563bd9f40da9c780fd2f6)

這塊研究了好久，目前先使用現成套件解決 [reverse\_markdown](https://github.com/xijo/reverse_markdown) 。

>  **_特別感謝前同事_** [**_Nick_**](d713969ca7ed?source=post_page-----ddd88a84e177--------------------------------) **_,_** [**_Chun-Hsiu Liu_**](72361fccaa43?source=post_page-----ddd88a84e177--------------------------------)_,James_ **_協力研究，之後有時間再自己寫改成原生的。_** 
### 成果

[原文](%E5%AF%A6%E6%88%B0%E7%B4%80%E9%8C%84-4-%E5%80%8B%E5%A0%B4%E6%99%AF-7-%E5%80%8B-design-patterns-78507a8de6a5) -\> [轉換後的 Markdown 結果](https://github.com/ZhgChgLi/ZMediumToMarkdown/blob/main/example/%E5%AF%A6%E6%88%B0%E7%B4%80%E9%8C%84-4-%E5%80%8B%E5%A0%B4%E6%99%AF-7-%E5%80%8B-design-patterns-78507a8de6a5.md)
[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/converting-medium-posts-to-markdown-ddd88a84e177) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
