---
title: Universal Links 新鮮事
author: ZhgChgLi
date: 2021-02-04T03:57:25.914Z
tags: [ios,ios-app-development,universal-links,app-store,deeplink]
---

### Universal Links 新鮮事

iOS 13, iOS 14 Universal Links 新鮮事＆建立本地測試環境
![Photo by NASA](images/12c5026da33d/1*HYAd1aal5Et1A-Qzs6VAtQ.jpeg "Photo by NASA")
### 前言

對於一個有網站又有 APP 的服務， Universal Links 的功能對於使用者體驗來說無比的重要，能達到 Web 與 APP 之間的無縫接軌；但一直以來都只有簡單設置，沒有太多的著墨；前陣子剛好又遇到花了點時間研究了一下，把一些有趣的事記錄下來。
### 常見考量

經手過的服務，對於實作 Universal Links 的考量都是 APP 上並沒有實作完整的網站功能，Universal Links 認的是域名，只要域名匹配到就會開啟 APP；關於這個問題可以下 NOT 排除 APP 上沒有相應功能的網址，若網站服務網址很極端，那乾脆新建一個 subdomain 用來做 Universal Links。
### apple-app-site-association 何時更新？
- iOS < 14，APP 在第一次安裝、更新時會去詢問 Universal Links 網站的 apple-app-site-association。
- iOS ≥ 14 ，則是由 Apple CDN 做快取定期更新 Universal Links 網站的 apple-app-site-association；APP 在第一次安裝、更新時會去跟 Apple CDN 拿取；但這邊就會有個問題，Apple CDN 的 apple-app-site-association 可能還是舊的。


關於 Apple CDN 的更新機制，查了一下文件，沒有提到；查了下[討論](https://developer.apple.com/forums/thread/651737)，官方也只回應「會定期更新」細節之後會發佈在文件…但至今依然還沒看到。

> _我自己覺得應該最慢 48 小時，就會更新吧。。。所以下次有更改到 apple-app-site-association 的話建議在 APP 上架更新前幾天就先改好 apple-app-site-association 上線。_
#### apple-app-site-association Apple CDN 確認：
```
Headers: HOST=app-site-association.cdn-apple.com  
GET https://app-site-association.cdn-apple.com/a/v1/ **你的網域**


```
![](images/12c5026da33d/1*dgDfMgkFPUfeuAuEhl7RFQ.png "")

可以取得當前 Apple CDN 上的版本長怎樣。（記得加上 Request Header `Host=https://app-site-association.cdn-apple.com/`）

#### iOS ≥ 14 Debug

因前述的 CDN 問題，那我們在開發階段該如何 debug 呢？

還好這部分蘋果有給解決方法，不然沒辦法即時更新真的要吐血了；我們只需要再 `applinks:domain.com` 加上 `?mode=developer` 即可，另外還有 `managed(for 企業內部 APP)`, or `developer+managed` 模式可設定。

![](images/12c5026da33d/1*z4R7wEHHAlLyF1rdAEAmew.png "")

加上 mode=developer 後，APP 在模擬器上每次 Build & Ｒun 時都會直接跟網站拿最新的 app-site-association 來用。

如果要 Build & Run 在實機則要先去「設定」->「開發者」-> 打開「Associated Domains Development」選項即可。
![](images/12c5026da33d/1*gj4Qm445mFERa25t6PZV1Q.jpeg "")
> _⚠️_ **_這邊有個坑_** _，app-site-association 可以放在網站根目錄或是_ `./.well-known` _目錄下；但在 mode=developer 下他只會問_ `./.well-known/app-site-association` _，害我以為怎麼沒效。_
### 開發測試

如果是 iOS <14 記得有更改過 app-site-association 的話要刪掉再重 Build & Run APP 才會去抓最新的回來，iOS ≥ 14 請參考前述方法加上 mode=developer。

app-site-association 內容的修改，好一點的話可以自行修改伺服器上的檔；但對於有時候碰不到伺服器端的我們來說，如果要做 universal links 的測試會非常的麻煩，要不停的麻煩後端同事幫忙，變成要很確定 app-site-association 內容後一次上線，一直改來改去會把同事逼瘋。
#### 在本地建一個模擬環境

為了解決上述問題，我們可以在本地起一個小服務。

首先在 mac 上安裝 nginx：
```
brew install nginx
```

如果沒安裝過 [brew](https://brew.sh/index_zh-tw) 可先安裝：

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"


```

安裝完 nginx 後，前往 `/usr/local/etc/nginx/` 打開編輯 `nginx.conf` 檔案：

```
location / {  
 root **/Users/zhgchgli/Documents** ;  
 index index.html index.htm;  
 }  
...略


```

大概在第 44 行的位置將 location / 裡的 root 換成你想要的目錄位置（這邊以 Documents 為例）。
> _listen on_ **_8080_** _port ，如果沒有衝突則不需要修改。_

儲存修改完後，下指令啟動 nginx：
```
nginx
```

若要停止時，則下：
```
nginx -s stop
```

停止。

如果有更改`nginx.conf` 記得要下：

```
nginx -s reload
```

重新啟用服務。

建立一個 `./.well-known` 目錄在剛設定的 `root` 目錄內，並將 `apple-app-site-association` 檔案放到 `./.well-known`內。

> _⚠️_ `.well-known` _建立後若消失，請注意 Mac 要打開「顯示隱藏資料夾」功能：_

在 terminal 下：
```
defaults write com.apple.finder AppleShowAllFiles TRUE


```

再下 killall finder 重啟所有 finder，即可。
![](images/12c5026da33d/1*AzM6lK0kzT-M-2OdXoyIXA.png "")
> _⚠️_ `apple-app-site-association` _看起來沒有副檔名，但實際還是有 .json 副檔名：_

在檔案上按右鍵 -> 「取得資訊 Get Info」->「Name & Extension」-> 檢查有無副檔名＆同時可取消勾選「隱藏檔案類型 Hide extension」
![](images/12c5026da33d/1*UFwnnjCot8xRqslhdQktKg.png "")

沒問題後，打開瀏覽器測試以下連結是否正常下載 apple-app-site-association：
```
[http://localhost:8080/.well-known/apple-app-site-association](http://localhost:8080/.well-known/apple-app-site-association)
```

如果能正常下載代表本地環境模擬成功！
> _如果出現 404/403 錯誤則請檢查 root 目錄是否正確、目錄/檔案是否有放入、apple-app-site-association 是否不小心帶了副檔名(.json)。_

 **註冊＆下載** [**Ngrok**](http://ngrok.com)
![ngrok.com](images/12c5026da33d/1*Shk9u59HgRRSiMw0wt899Q.png "ngrok.com")
![解壓縮出 ngrok 執行檔](images/12c5026da33d/1*ljBqKrOFb9Gq48dO0GeIeA.png "解壓縮出 ngrok 執行檔")
![進入 Dashboard 頁面執行 Config 設定](images/12c5026da33d/1*fnEUyJMtVhUGurU5vX5K6A.png "進入 Dashboard 頁面執行 Config 設定")
```
./ngrok authtoken 你的TOKEN


```

設定好之後，下：
```
./ngrok http **8080**


```
> _因我們的 nginx 在 8080 port。_

啟動服務。
![](images/12c5026da33d/1*8i6EP7KKwxihLZ1PG1RUGw.png "")

這時候我們會看到一個服務啟動狀態視窗，可以從 Forwarding 中取的此次分配到的公開網址。
> _⚠️_**_每次啟動分配到的網址都會變，所以僅能作為開發測試使用。  
> 這邊以此次分配到的網址_** `https://ec87f78bec0f.ngrok.io/` _為例_

回到瀏覽器改輸入 `https://ec87f78bec0f.ngrok.io/.well-known/apple-app-site-association` 看看能不能正常下載瀏覽 apple-app-site-association 檔案，如果沒問題則可繼續下一步。


將 ngrok 分配到的網址輸入到 Associated Domains applinks: 設定中。
![](images/12c5026da33d/1*K5Eio0Yi7nNHQuLSuIsYeA.png "")

記得帶上 `?mode=developer` 方便我們測試。


 **重新 Build & Run APP：** 
![](images/12c5026da33d/1*VFIKU-UxCHNQVnf8DOV8Qw.png "")

打開瀏覽器輸入相應的 Universal Links 測試網址（EX: `https://ec87f78bec0f.ngrok.io/buy/123`）查看效果。

> _頁面出現 404 不要理他，因為我們實際沒有那一頁；我們只是要測 iOS 對網址匹配的功能符不符合我們預期；如果上方有出現 「Open」代表匹配成功，另外也可以測 NOT 反向的狀況。_

點擊「Open」後開啟 APP -> 測試成功！
> _開發階段都測試 OK 後，將確認修改過之後的 apple-app-site-association 檔案再交給後端上傳到伺服器就能確保萬無一失囉～  
> 最後記得將 Associated Domains applinks: 改為正試機網址。_  

另外我們也可以從 ngrok 運行狀態視窗中看到每次 APP Build & Run 有沒有跟我們要 apple-app-site-association 檔案：
![](images/12c5026da33d/1*d6yvnEaiOPbqy57PDMe2Mw.png "")
### Applinks 設定內容
#### iOS < 13 之前：

設定檔較簡單，只有以下內容可設定：
```JSON
{
  "applinks": {
      "apps": [],
      "details": [
           {
             "appID" : "TeamID.BundleID",
             "paths": [
               "NOT /help/",
               "*"
             ]
           }
       ]
   }
}
```

將 `TeamID.BundleId` 換成你的專案設定 (ex: TeamID = `ABCD`, BundleID = `li.zhgchg.demoapp` =\> `ABCD.li.zhgchg.demoapp`)。

> _如果有多個 appID 則要重複加入多組。_

 **paths 部分則為匹配規則，能支援以下幾種語法：** 
- `*` ：匹配 0~多個字元，ex: `/home/*` (home/alan…)
- `?`：匹配 1 個字元，ex: `201?` (2010~2019)
- `?*`：匹配 1 個~多個字元，ex: `/?*` (/test、/home..)
- `NOT` ：反向排除，ex: `NOT /help` (any url but /help)


更多玩法組合可自己依照實際情況決定，更多資訊可參考[官方文件](https://developer.apple.com/library/archive/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW1)。

> _- 請注意，他不是 Regex，不支援任何 Regex 寫法。  
> - 舊版不支援 Query (?name=123)、Anchor (#title)。  
> - 中文網址須先轉成 ASCII 後才能放在 paths 中 (所有url 字元均要是 ASCII)。_  
#### iOS ≥ 13 之後：

強化了設定檔內容的功能，多增加支援 Query/Anchor、字符集、編碼處理。
```JSON
"applinks": {
  "details": [
    {
      "appIDs": [ "TeamID.BundleID" ],
      "components": [
        {
          "#": "no_universal_links",
          "exclude": true,
          "comment": "Matches any URL whose fragment equals no_universal_links and instructs the system not to open it as a universal link"
        },
        {
          "/": "/buy/*",
          "comment": "Matches any URL whose path starts with /buy/"
        },
        {
          "/": "/help/website/*",
          "exclude": true,
          "comment": "Matches any URL whose path starts with /help/website/ and instructs the system not to open it as a universal link"
        },
        {
          "/": "/help/*",
          "?": { "articleNumber": "????" },
          "comment": "Matches any URL whose path starts with /help/ and that has a query item with name 'articleNumber' and a value of exactly 4 characters"
        }
      ]
    }
  ]
}
```

轉貼自官方文件，可以看到格式有所改變。

`appIDs` 為陣列，可放入多組 appID，這樣就不用像以前一樣只能整個區塊重複輸入。
> _WWDC 有提到與舊版兼容，_ **_當 iOS ≥ 13 有讀到新的格式就會忽略舊的 paths_** _。_

匹配規則改放在 `components` 中；支援 3 種類型：

- `/`： URL
- `?` ：Query，ex: ?name=123&place=tw
- `#` ：Anchor，ex: #title


並且可以搭配使用，假設今天 `/user/?id=100#detail` 才需要跳到 APP 則可寫成：

```
{
  "/": "/user/*",
  "?": { "id": "*" },
  "#": "detail"
}
```

其中匹配語法同原本語法，也是支援 `*` `?` `?*` 。


新增 `comment` 註解欄位，可輸入註解方便辨識。（但請注意這是公開的，別人也看得到）


反向排除則改為指定`exclude: true` 。


新增 `caseSensitive` 指定功能，可指定匹配規則是否對大小寫敏感，`預設：true`，有這需求的話可以少寫許多規則。


新增 `percentEncoded` 前面說到的，舊版需要先將網址轉為 ASCII 放到 paths 中（如果是中文字會變得很醜無法辨識）；這個參數就是是否要幫我們自動 encode，`預設是 true`。  
假設是中文網址就能直接放入了(ex: `/客服中心` )。


詳細官方文件可[參考此](https://developer.apple.com/documentation/bundleresources/applinks/details/components)。


 **預設字符集：** 

這算是這次更新蠻重要的功能之一，新增支援字符集。

系統幫我們定義好的字符集：
- `$(alpha)` ：A-Z 和 a-z
- `$(upper)` ：A-Z
- `$(lower)` ：a-z
- `$(alnum)`：A-Z 和 a-z 和 0–9
- `$(digit)`：0–9
- `$(xdigit)`：十六進制字符，0–9 和 a,b,c,d,e,f,A,B,C,D,E,F
- `$(region)`：ISO 地區編碼 [isoRegionCodes](https://developer.apple.com/documentation/foundation/locale/2293271-isoregioncodes)，Ex: TW
- `$(lang)`：ISO 語言編碼 [isoLanguageCodes](https://developer.apple.com/documentation/foundation/locale/2293744-isolanguagecodes)，Ex: zh


假設我們的網址有多語系，我想要支援 Universal links 時，可以這樣設定：
```
"components": [        
     { "/" : "/$(lang)-$(region)/$(food)/home" }      
]
```

這樣不管是 `/zh-TW/home` 、`/en-US/home` 都能支援，非常方便，不用自己寫一整排規則！


 **自訂字符集：** 

除了預設字符集之外，我們也能自訂字符集，增加設定檔復用、可讀性。

在 `applinks` 中加入 `substitutionVariables` 即可：

```
{
  "applinks": {
    "substitutionVariables": {
      "food": [ "burrito", "pizza", "sushi", "samosa" ]
    },
    "details": [{
      "appIDs": [ ... ],
      "components": [
        { "/" : "/$(food)/" }
      ]
    }]
  }
}
```

範例中自訂了一個 `food` 字符集，並在後續 `components` 中使用。


以上範例可匹配 `/burrito`, `/pizza`, `/sushi`, `/samosa`。


細節可參考[此篇](https://developer.apple.com/documentation/bundleresources/applinks/substitutionvariables)官方文件。

#### 沒有靈感？

如果對設定檔內容沒有靈感，可偷偷參考其他網站福的內容，只要在服務網站首頁網址加上 `/app-site-association` 或 `/.well-known/app-site-association` 即可讀取他們的設定。


例如： [https://www.netflix.com/apple-app-site-association](https://www.netflix.com/apple-app-site-association)

### 補充

在有使用 `SceneDelegate` 的情況下，open universal link 的進入點是在SceneDelegate 中：

```
 **func** scene( **\_** scene: UIScene, continue userActivity: NSUserActivity)
```

 **而非 AppDelegate 的：** 
```
 **func** application( **\_** application: UIApplication, continue userActivity: NSUserActivity, restorationHandler: **@escaping** ([UIUserActivityRestoring]?) -\> Void) -\> Bool
```
### 延伸閱讀
- [iOS 跨平台帳號密碼整合，加強登入體驗](ios-%E8%B7%A8%E5%B9%B3%E5%8F%B0%E5%B8%B3%E8%99%9F%E5%AF%86%E7%A2%BC%E6%95%B4%E5%90%88%E5%8A%A0%E5%BC%B7%E7%99%BB%E5%85%A5%E9%AB%94%E9%A9%97-948ed34efa09)
- [iOS Deferred Deep Link 延遲深度連結實作(Swift)](ios-deferred-deep-link-%E5%BB%B6%E9%81%B2%E6%B7%B1%E5%BA%A6%E9%80%A3%E7%B5%90%E5%AF%A6%E4%BD%9C-swift-b08ef940c196)

#### 參考資料
- [What’s new in Universal Links](https://www.wwdcnotes.com/notes/wwdc20/10098/)
- [Apple Documentation](https://developer.apple.com/documentation/bundleresources/applinks)

[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/universal-links-%E6%96%B0%E9%AE%AE%E4%BA%8B-12c5026da33d) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
