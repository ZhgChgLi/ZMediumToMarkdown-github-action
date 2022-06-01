---
title: AppStore APP’s Reviews Bot 那些事
author: ZhgChgLi
date: 2021-04-21T15:16:31.071Z
tags: [slackbot,ios-app-development,ruby,fastlane,automator]
---

### AppStore APP’s Reviews Slack Bot 那些事

使用 Ruby+Fastlane-SpaceShip 動手打造 APP 評價追蹤通知 Slack 機器人
![Photo by Austin Distel](images/cb0c68c33994/1*BMCG3cu21W5MbODBbhI-sA.jpeg "Photo by Austin Distel")
#### 吃米不知米價
![AppReviewBot 為例](images/cb0c68c33994/1*Iv6qvrBfyv3bU1NK1hPVHg.png "AppReviewBot 為例")

最近才知道 Slack 中轉發 APP 最新評價訊息的機器人是要付費的，我一直以為這功能是免費的；費用從 $5 到 $200 美金/月都有，因為各平台都不會只做「App Review Bot」的功能，其他還有數據統計、紀錄、統一後台、與競品比較…等等，費用也是照各平台能提供的服務為標準；Review Bot 只是他們的一環，但我就只想用這個功能其他不需要，如果是這樣付費蠻浪費的。
### 問題

本來是用免費開源的工具 [TradeMe/ReviewMe](https://github.com/TradeMe/ReviewMe) 來做 Slack 通知，但這個工具已年久失修，時不時 Slack 會爆噴一些舊的評價，看得讓人心驚膽顫（很多 Bug 都早已修復，害我們以為又有問題！），原因不明。


所以考慮找其他工具、方法取代。
### 原理探究

有了動機之後，再來研究下達成目標的原理。
#### 官方 API ❌

蘋果有提供 [App Store Connect API](https://developer.apple.com/app-store-connect/api/)，但沒提供撈取評價功能。

#### Public URL API (RSS) ⚠️

蘋果有提供公開的 APP 評價 [RSS 訂閱網址](https://rss.itunes.apple.com/zh-tw)，而且除了 rss xml 還提供 json 格式。

```
https://itunes.apple.com/ **國家碼** /rss/customerreviews/id= **APP\_ID** /page= **1** /sortBy= **mostRecent** / **json**


```
- 國家碼：可參考[這份文件](https://help.apple.com/app-store-connect/#/dev997f9cf7c)。

- APP\_ID：前往 App 網頁版，會得到網址：https://apps.apple.com/tw/app/APP名稱/id **12345678** ，id 後面的數字及為 App ID（純數字）。

- page：可請求 1~10 頁，超過無法取得。
- sortBy：`mostRecent/json` 請求最新的＆ json 格式，也可改為 `mostRecent/xml` 則為 xml 格式。



 **評價資料回傳如下：** 
```JSON
{
  "author": {
    "uri": {
      "label": "https://itunes.apple.com/tw/reviews/id123456789"
    },
    "name": {
      "label": "test"
    },
    "label": ""
  },
  "im:version": {
    "label": "4.27.1"
  },
  "im:rating": {
    "label": "5"
  },
  "id": {
    "label": "123456789"
  },
  "title": {
    "label": "很棒的存在！"
  },
  "content": {
    "label": "人生值得了～",
    "attributes": {
      "type": "text"
    }
  },
  "link": {
    "attributes": {
      "rel": "related",
      "href": "https://itunes.apple.com/tw/review?id=123456789&type=Purple%20Software"
    }
  },
  "im:voteSum": {
    "label": "0"
  },
  "im:contentType": {
    "attributes": {
      "term": "Application",
      "label": "應用程式"
    }
  },
  "im:voteCount": {
    "label": "0"
  }
}
```

 **優點：** 
. 公開、不需身份驗證步驟即可存取
. 簡單好用


 **缺點：** 
. 此 RSS API 很老舊都沒更新
. 回傳評價的資訊太少（沒留言時間、已編輯過評價？、已回覆？）
. 遇到資料錯亂問題（後面幾頁偶爾會突然噴舊資料）
. 最多存取 10 頁

> _關於我們遇到的最大問題是 3；但這部分不確定是我們用的_ [_Bot 工具_](https://github.com/TradeMe/ReviewMe)_問題，還是這個 RSS URL 資料有問題。_
#### Private URL API ✅

這個方法說來有點旁門左道，也是我突發奇想發現的；但在後續參考了其他 Review Bot 做法之後發現很多網站也都是這樣用，應該沒什麼問題而且我 4~5 年前就看過有工具這樣做了，只是當時沒深入研究。

 **優點：** 
. 同蘋果後台資料
. 資料完整且最新
. 可做更多細節篩選
. 具備深度整合的 APP 工具也是用這個方法（AppRadar/AppReviewBot…）


 **缺點：** 
. 非官方公布方法（旁門左道）
. 因蘋果實行全面兩步驟登入，所以登入 session 需要定期更新。


 **第一步 — 嗅探 App Store Connect 後台評論區塊 Load 資料的 API：** 
![](images/cb0c68c33994/1*74lbicQ_vPzrLfm1imk7Pg.png "")

得到蘋果後台是透過打：
```
https://appstoreconnect.apple.com/WebObjects/iTunesConnect.woa/ra/apps/ **APP\_ID** /platforms/ios/reviews?index=0&sort=REVIEW\_SORT\_ORDER\_MOST\_RECENT


```

這個 endpoint 取得評價列表：
![](images/cb0c68c33994/1*I00Znmzaivm_-7ous0-4Pw.png "")

index = 分頁 offset，一次最多顯示 100 筆。

 **評價資料回傳如下：** 
```JSON
{
  "value": {
    "id": 123456789,
    "rating": 5,
    "title": "很棒的存在！",
    "review": "人生值得了～",
    "created": null,
    "nickname": "test",
    "storeFront": "TW",
    "appVersionString": "4.27.1",
    "lastModified": 1618836654000,
    "helpfulViews": 0,
    "totalViews": 0,
    "edited": false,
    "developerResponse": null
  },
  "isEditable": true,
  "isRequired": false,
  "errorKeys": null
}
```

另外經過測試後發現，只需要在帶上 `cookie: myacinfo=<Token>` 即可偽造請求得到資料：

![](images/cb0c68c33994/1*b_vINNRMrAIQrkuouN7X1Q.png "")

API 有了、要求的 header 知道了，再來就要想辦法自動化取得後台這個 cookie 資訊。

 **第二步 —萬能 Fastlane** 

因蘋果現在實行全 Two-Step Verification，所以對於登入驗證自動化變得更加煩瑣，幸好與蘋果鬥智鬥勇的 [Fastlane](https://docs.fastlane.tools/best-practices/continuous-integration/)，除了正規的 App Store Connect API、iTMSTransporter、網頁認證(包含兩步驟認證)全都有實作；我們可以直接使用 Fastlane 的指令：

```
fastlane spaceauth -u \<**App Store Connect 帳號(Email)\>**


```

此指令會完成網頁登入驗證(包含兩步驟認證)，然後將 cookie 存入 FASTLANE_SESSION 檔案之中。

會得到類似如下字串：
```
!ruby/object:HTTP::Cookie   
name: **dqsid** value: **\<token\>**  
domain: appstoreconnect.apple.com for\_domain: false path: "/" secure: true httponly: true expires: max\_age: 1800  
created\_at: &1 2021-04-21 22:02:47.118437000 +08:00  
accessed\_at: \*1


```

將 `myacinfo = value` 帶入就能取得評價列表。


 **第三步 — SpaceShip** 

本來以為 Fastlane 只能幫我們到這了，再來要自己串起從 Fastlane 拿到 cookie 然後打 api 的 flow；沒想到經過一番探索發現 Fastlane 關於驗證這塊的模組 `SpaceShip` 還有更多強大的功能！

![SpaceShip](images/cb0c68c33994/1*OlYQLNXAOk1oNqDP7LSlrA.png "SpaceShip")

SpaceShip 裡面已經幫我們打包好撈評價列表的方法 [**Class: Spaceship::TunesClient::get\_reviews**](https://www.rubydoc.info/gems/spaceship/0.39.0/Spaceship/TunesClient#get_reviews-instance_method) 了！

```
app = Spaceship::Tunes::login(appstore_account, appstore_password)
```

*storefront = 地區

 **第四步 — 組裝** 

Fastlane、Spaceship 都是由 ruby 撰寫，所以我們也要用 ruby 來製作這個 Bot 小工具。

我們可以建立一個 `reviewBot.rb `檔案，編譯執行時只需在 Terminal 輸入：

```
ruby reviewBot.rb
```

即可。_(\*更多 ruby 環境問題可參考文末提示)_


 **首先** ，因原本的 get\_reviews 口的參數不符合我們需求；我想要的是全地區、全版本的評價資料、不需要篩選、支援分頁：
```Ruby
# Extension Spaceship->TunesClient
module Spaceship
  class TunesClient < Spaceship::Client
    def get_recent_reviews(app_id, platform, index)
      r = request(:get, "ra/apps/#{app_id}/platforms/#{platform}/reviews?index=#{index}&sort=REVIEW_SORT_ORDER_MOST_RECENT")
      parse_response(r, 'data')['reviews']
     end
  end
end
```

所以我們自己在 TunesClient 中擴充一個方法，裡面參數只帶 app\_id、platform = `ios` ( **全小寫** )、index = 分頁 offset。


 **再來組裝登入驗證、撈評價列表：** 
```Ruby
index = 0
breakWhile = true
while breakWhile
  app = Spaceship::Tunes::login(APPStoreConnect 帳號(Email), APPStoreConnect 密碼)
  reviews = app.get_recent_reviews($app_id, $platform, index)
  if reviews.length() <= 0
    breakWhile = false
    break
  end
  reviews.each { |review|
    index += 1
    puts review["value"]
  }
end
```

使用 while 遍歷所有分頁，當跑到無內容時終止。

 **再來要加上紀錄上次最新一筆的時間，只通知沒通知過的最新訊息：** 
```Ruby
lastModified = 0
if File.exists?(".lastModified")
  lastModifiedFile = File.open(".lastModified")
  lastModified = lastModifiedFile.read.to_i
end
newLastModified = lastModified
isFirst = true
messages = []

index = 0
breakWhile = true
while breakWhile
  app = Spaceship::Tunes::login(APPStoreConnect 帳號(Email), APPStoreConnect 密碼)
  reviews = app.get_recent_reviews($app_id, $platform, index)
  if reviews.length() <= 0
    breakWhile = false
    break
  end
  reviews.each { |review|
    index += 1
    if isFirst
      isFirst = false
      newLastModified = review["value"]["lastModified"]
    end

    if review["value"]["lastModified"] > lastModified && lastModified != 0  
      # 第一次使用不發通知
      messages.append(review["value"])
    else
      breakWhile = false
      break
    end
  }
end

messages.sort! { |a, b|  a["lastModified"] <=> b["lastModified"] }
messages.each { |message|
    notify_slack(message)
}

File.write(".lastModified", newLastModified, mode: "w+")
```

單純用一個 `.lastModified` 紀錄上一次執行時拿到的時間。


_\*第一次使用不發通知，否則會一次狂噴_

 **最後一步，組合推播訊息 & 發到 Slack：** 
```Ruby
# Slack Bot
def notify_slack(review)
  rating = review["rating"].to_i
  color = rating >= 4 ? "good" : (rating >= 2 ? "warning" : "danger")
  like = review["helpfulViews"].to_i > 0 ? " - #{review["helpfulViews"]} :thumbsup:" : ""
  date = review["edited"] == false ? "Created at: #{Time.at(review["lastModified"].to_i / 1000).to_datetime}" : "Updated at: #{Time.at(review["lastModified"].to_i / 1000).to_datetime}"
  
    
  isResponse = ""
  if review["developerResponse"] != nil && review["developerResponse"]['lastModified'] < review["lastModified"]
    isResponse = " (回覆已過時)"
  end
  
  edited = review["edited"] == false ? "" : ":memo: 使用者更新評論#{isResponse}："

  stars = "★" * rating + "☆" * (5 - rating)
  attachments = {
    :pretext => edited,
    :color => color,
    :fallback => "#{review["title"]} - #{stars}#{like}",
    :title => "#{review["title"]} - #{stars}#{like}",
    :text => review["review"],
    :author_name => review["nickname"],
    :footer => "iOS - v#{review["appVersionString"]} - #{review["storeFront"]} - #{date} - <https://appstoreconnect.apple.com/apps/APP_ID/appstore/activity/ios/ratingsResponses|Go To App Store>"
  }
  payload = {
   :attachments => [attachments],
   :icon_emoji => ":storm_trooper:",
   :username => "ZhgChgLi iOS Review Bot"
  }.to_json
  cmd = "curl -X POST --data-urlencode 'payload=#{payload}' SLACK_WEB_HOOK_URL"
  system(cmd, :err => File::NULL)
  puts "#{review["id"]} send Notify Success!"
 end
```

`SLACK_WEB_HOOK_URL` = [**Incoming WebHook URL**](https://slack.com/apps/A0F7XDUAZ-incoming-webhooks)
###  **最終結果** 
```Ruby
require "Spaceship"
require 'json'
require 'date'

# Config
$slack_web_hook = "目標通知的 web hook url"
$slack_debug_web_hook = "機器人有錯誤時的通知 web hook url"
$appstore_account = "APPStoreConnect 帳號(Email)"
$appstore_password = "APPStoreConnect 密碼"
$app_id = "APP_ID"
$platform = "ios"

# Extension Spaceship->TunesClient
module Spaceship
  class TunesClient < Spaceship::Client
    def get_recent_reviews(app_id, platform, index)
      r = request(:get, "ra/apps/#{app_id}/platforms/#{platform}/reviews?index=#{index}&sort=REVIEW_SORT_ORDER_MOST_RECENT")
      parse_response(r, 'data')['reviews']
     end
  end
end

# Slack Bot
def notify_slack(review)
  rating = review["rating"].to_i
  color = rating >= 4 ? "good" : (rating >= 2 ? "warning" : "danger")
  like = review["helpfulViews"].to_i > 0 ? " - #{review["helpfulViews"]} :thumbsup:" : ""
  date = review["edited"] == false ? "Created at: #{Time.at(review["lastModified"].to_i / 1000).to_datetime}" : "Updated at: #{Time.at(review["lastModified"].to_i / 1000).to_datetime}"
  
    
  isResponse = ""
  if review["developerResponse"] != nil && review["developerResponse"]['lastModified'] < review["lastModified"]
    isResponse = " (客服回覆已過時)"
  end
  
  edited = review["edited"] == false ? "" : ":memo: 使用者更新評論#{isResponse}："

  stars = "★" * rating + "☆" * (5 - rating)
  attachments = {
    :pretext => edited,
    :color => color,
    :fallback => "#{review["title"]} - #{stars}#{like}",
    :title => "#{review["title"]} - #{stars}#{like}",
    :text => review["review"],
    :author_name => review["nickname"],
    :footer => "iOS - v#{review["appVersionString"]} - #{review["storeFront"]} - #{date} - <https://appstoreconnect.apple.com/apps/APP_ID/appstore/activity/ios/ratingsResponses|Go To App Store>"
  }
  payload = {
   :attachments => [attachments],
   :icon_emoji => ":storm_trooper:",
   :username => "ZhgChgLi iOS Review Bot"
  }.to_json
  cmd = "curl -X POST --data-urlencode 'payload=#{payload}' #{$slack_web_hook}"
  system(cmd, :err => File::NULL)
  puts "#{review["id"]} send Notify Success!"
 end

begin
    lastModified = 0
    if File.exists?(".lastModified")
      lastModifiedFile = File.open(".lastModified")
      lastModified = lastModifiedFile.read.to_i
    end
    newLastModified = lastModified
    isFirst = true
    messages = []

    index = 0
    breakWhile = true
    while breakWhile
      app = Spaceship::Tunes::login($appstore_account, $appstore_password)
      reviews = app.get_recent_reviews($app_id, $platform, index)
      if reviews.length() <= 0
        breakWhile = false
        break
      end
      reviews.each { |review|
        index += 1
        if isFirst
          isFirst = false
          newLastModified = review["value"]["lastModified"]
        end

        if review["value"]["lastModified"] > lastModified && lastModified != 0  
          # 第一次使用不發通知
          messages.append(review["value"])
        else
          breakWhile = false
          break
        end
      }
    end
    
    messages.sort! { |a, b|  a["lastModified"] <=> b["lastModified"] }
    messages.each { |message|
        notify_slack(message)
    }
    
    File.write(".lastModified", newLastModified, mode: "w+")
rescue => error
    attachments = {
        :color => "danger",
        :title => "AppStoreReviewBot Error occurs!",
        :text => error,
        :footer => "*因蘋果技術限制，精準評價爬取功能約每一個月需要重新登入設定，敬請見諒。"
    }
    payload = {
        :attachments => [attachments],
        :icon_emoji => ":storm_trooper:",
        :username => "ZhgChgLi iOS Review Bot"
    }.to_json
    cmd = "curl -X POST --data-urlencode 'payload=#{payload}' #{$slack_debug_web_hook}"
    system(cmd, :err => File::NULL)
    puts error
end

```

另外還加上了 begin…rescue (try…catch) 保護，如果有出現錯誤則發 Slack 通知我們回來檢查（多半是 session 過期）。
>  **_最後只要將此腳本加到 crontab / schedule 等排程工具定時執行即可！_** 

 **效果圖：** 
![](images/cb0c68c33994/1*B0xW1CXU-avz2j8_ny3Ang.jpeg "")
### 免費的其他選擇
. [AppFollow](https://appfollow.io/)：使用 Public URL API (RSS)，只能說堪用吧。
. [feedis.io](https://feedis.io/product/proxime/features)：使用 Private URL API，需要把帳號密碼給他們。
. [TradeMe/ReviewMe](https://github.com/TradeMe/ReviewMe)：自架服務(node.js)，我們原先用這個，但遇到前述問題。
. [JonSnow](https://github.com/saiday/JonSnow)：自架服務(GO)，支援一鍵部署到 heroku，作者：[@saiday](https://twitter.com/saiday)

### 溫馨提示

1.⚠️Private URL API 方法，如果用有二階段驗證的帳號，最長每 30 天都需要重新驗證才能使用且目前無解；如果有辦法生出沒二階段的帳號就可以無痛爽爽用。
![#important-note-about-session-duration](images/cb0c68c33994/1*EE2J5HmdiIogMwC3Iiy0KA.png "#important-note-about-session-duration")

2.⚠️不論是免費、付費、本文的自架；切勿使用開發者帳號，務必開一個獨立的 App Store Connect 帳號使用，權限只開放「Customer Support」；防止資安問題。

3.Ruby 建議使用 [rbenv](https://gist.github.com/sandyxu/8aceec7e436a6ab9621f) 進行管理，因系統自帶 2.6 版容易造成衝突。


4.在 macOS Catalina 如遇到 GEM、Ruby 環境錯誤問題，可參考[此回覆](https://github.com/orta/cocoapods-keys/issues/198#issuecomment-510909030)解決。

### Problem Solved!

經過以上心路歷程，更瞭解的 Slack Bot 的運作方式；還有 iOS App Store 是如何爬取評價內容的，另外也摸了下 ruby！寫起來真不錯！
[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/appstore-apps-reviews-bot-%E9%82%A3%E4%BA%9B%E4%BA%8B-cb0c68c33994) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
