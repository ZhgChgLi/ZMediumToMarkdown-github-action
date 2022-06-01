---
title: APP有用HTTPS傳輸，但資料還是被偷了。
author: ZhgChgLi
date: 2019-09-20T10:01:01.345Z
tags: [mitmproxy,man-in-the-middle,ios,ios-app-development,hacking]
---

### APP有用HTTPS傳輸，但資料還是被偷了。

iOS+MacOS 使用 mitmproxy 進行中間人攻擊(Man-in-the-middle attack) 嗅探API傳輸資料教學及如何防範？
### 前言

前陣子剛在公司辦完一場內部的 [CTF競賽](%E5%A6%82%E4%BD%95%E6%89%93%E9%80%A0%E4%B8%80%E5%A0%B4%E6%9C%89%E8%B6%A3%E7%9A%84%E5%B7%A5%E7%A8%8Bctf%E7%AB%B6%E8%B3%BD-729d7b6817a4) ，在發想題目時回想起大學時候還在做後端(PHP)時經手的專案，一個集點的APP，大概就是有個任務列表，然後觸發條件完成就Call API獲得點數；老闆認為Call API有經過HTTPS加密傳輸資料就很安全了 — 直到我向他展示中間人攻擊，直接嗅探傳輸資料，偽造API呼叫獲得點數….


再加上最近幾年大數據崛起，網路爬蟲滿天飛；爬蟲攻防戰日漸白熱化，[爬取與防爬之間花招百出](https://coolcao.com/2018/06/09/tips-of-anti-spider-in-fe/)，只能說道高一尺魔高一丈啊！


爬蟲的另一條下手對象就是APP的API，如果沒有任何防範幾乎等於門戶大開；不但好操作而且格式也乾淨，更不容易被識別阻擋；所以如果網頁端已經費盡全力阻擋，資料還是不段被爬，不妨檢查一下APP的API有無漏洞。

因為這個議題我不知道該如何出在 CTF比賽中 ，所以就單拉出一篇文章作為紀錄 ；本篇只是粗淺給個概念 — [HTTPS能透過憑證替換進行傳輸內容解密](http://www.aqee.net/post/man-in-the-middle-attack.html)、如何加強安全性防止；實際網路理論不是我的強項也都還給老師了，如果已經有這方面概念的朋友就不用花時間看這篇，或拉到底看APP該如果保護！

### 實際操作

環境: MacOS + iOS
> _Android 使用者可以直接下載_ [_Packet Capture_](https://play.google.com/store/apps/details?id=app.greyshirts.sslcapture&hl=en) _(免費)、iOS 用戶可使用_ [_Surge 4_](https://apps.apple.com/us/app/surge-3/id1442620678) _這套軟體(_**_付費)_**_解鎖中間人攻擊功能、MacOS也可以使用另一套付費軟體Charles。  
> 本文章主要講解iOS使用_ **_免費_** _的 mitmproxy 進行操作，如果您有上述的環境就不用這麼麻煩啦，直接APP打開在手機上掛載VPN替換掉憑證就能進行中間人攻擊！(ㄧ樣請直接下拉到底看該如何保護！)_

**[2021/02/25 更新]：**Mac 有新的免費圖形化介面程式 ([Proxyman](https://proxyman.io/)) 可以用，可搭配[參考此篇文章](%E4%BD%BF%E7%94%A8-python-google-cloud-platform-line-bot-%E8%87%AA%E5%8B%95%E5%9F%B7%E8%A1%8C%E4%BE%8B%E8%A1%8C%E7%91%A3%E4%BA%8B-70a1409b149a)的第一部分。
#### 安裝 [mitmproxy](https://mitmproxy.org)


 **直接使用 brew 安裝** ：
```
brew install mitmproxy
```

 **安裝完成!** 

_p.s 如果你出現 brew: command not found 請先安裝_ [_brew_](https://brew.sh/index_zh-tw) _套件管理工具_：
```
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"


```
#### mitmproxy 使用

安裝完成後，在 Terminal 輸入以下指令啟用：
```
mitmproxy
```
![啟動成功](images/46410aaada00/1*VTtl6EUMOTV4oRNUjRQHNg.png "啟動成功")
#### 讓手機跟Mac在同個區域網路內&取得Mac的IP位址

方法(1) Mac 連接 WiFi、手機也使用同個 WiFi  
**Mac的IP位址 =** 「系統偏好設定」-\>「網路」-\>「Wi-Fi」-\>「IP Address」


方法(2) Mac 使用有線網路，開啟網路分享；手機連上該熱點網路:
![「系統偏好設定」-> 「共享」->選擇「乙太網路」->「Wi-Fi」打勾-> 「Internet 共享」啟用](images/46410aaada00/1*R9fthpHlrWzTh4R3fEwO5Q.gif "「系統偏好設定」-> 「共享」->選擇「乙太網路」->「Wi-Fi」打勾-> 「Internet 共享」啟用")

 **Mac的IP位址 = 192.168.2.1** （️️注意⚠️ 不是乙太網路網路的IP，是Mac用做網路分享基地台的IP)
#### 手機網路設置WiFi — Proxy伺服器資訊
![「設定」-> 「WiFi」-> 「HTTP 代理伺服器」-> 「手動」-> 「伺服器輸入 Mac的IP位址」-> 「連接埠輸入 8080」-> 「儲存」](images/46410aaada00/1*ziIFrGQaMr2kYrQHwLYNJg.jpeg "「設定」-> 「WiFi」-> 「HTTP 代理伺服器」-> 「手動」-> 「伺服器輸入 Mac的IP位址」-> 「連接埠輸入 8080」-> 「儲存」")
> _這時網頁打不開、出現憑證錯誤是正常的；我們繼續往下做…_
#### 安裝 mitmproxy 自訂 https 憑證

如同上述所說，中間人攻擊的實現方式就是在通訊之中使用自己的憑證做抽換加解密資料；所以我們也要在手機上安裝這個自訂的憑證。

 **1.用手機safari打開** [**http://mitm.it**](http://mitm.it)
![出現左邊->Proxy設定✅/ 出現右邊代表 Proxy設定有誤🚫](images/46410aaada00/1*BuvCYx9WRzG0ECO3H_BS0A.jpeg "出現左邊->Proxy設定✅/ 出現右邊代表 Proxy設定有誤🚫")
![「Apple」->「安裝描述檔」->「安裝」](images/46410aaada00/1*qKDHxi9HxUP41oDJahBfBA.jpeg "「Apple」->「安裝描述檔」->「安裝」")
> _⚠️到這裡還沒結束，我們還要去關於裡啟用描述檔_
![「一般」->「關於」->「憑證信任設定」->「mitmproxy」啟用](images/46410aaada00/1*mOijblpQepazFPIwob4r8Q.jpeg "「一般」->「關於」->「憑證信任設定」->「mitmproxy」啟用")

 **完成！這時我們再回去瀏覽器就能正常瀏覽網頁了。** 
#### 回到Mac 上操作 mitmproxy
![可以在mitmproxy Terminal上看到剛手機的資料傳輸紀錄](images/46410aaada00/1*kiEPaTm5bhnFLBfQngQPgA.png "可以在mitmproxy Terminal上看到剛手機的資料傳輸紀錄")
![找到想嗅探的紀錄進入查看Request(送出哪些參數)/Response(回傳了什麼內容)](images/46410aaada00/1*5I6l9cO3LeXfcwGLpWGKPQ.gif "找到想嗅探的紀錄進入查看Request(送出哪些參數)/Response(回傳了什麼內容)")
#### 常用操作按鍵集：
```
「 ? 」= 查看按鍵操作集文檔
「 k 」/「⬆」= 上 
「 j 」/「⬇」= 下 
「 h 」/「⬅」= 左 
「 l 」/「➡️」️= 右 
「 space 」= 下一頁
「 enter 」= 進入查看詳情
「 q 」= 返回上一頁/退出
「 b 」= 匯出response body到指定path文字檔 
「 f 」= 篩選紀錄條件
「 z 」= 清除所有紀錄
「 e 」= 編輯Request(cookie、headers、params...)
「 r 」= 重新發送Request
```
#### 不習慣CLI? 沒關係，可以改用 Web GUI !

除了 mitmproxy 啟用方式之外，我們可以改下：
```
mitmweb
```

就能使用新的 Web GUI 進行操作觀察
![mitmweb](images/46410aaada00/1*Stbf8gUk8iXwNkozOKyOjA.png "mitmweb")
#### 重頭戲，嗅探APP資料：

上述環境都建置完成也熟悉之後，就可以進入我們的重頭戲；嗅探APP API的資料傳輸內容！
> _這邊以某房屋APP作為範例，無惡意純學術交流使用!_
> _我們想知道物件列表的API是如何請求和回傳什麼內容!_
![首先先按「z」清除所有紀錄(避免搞亂)](images/46410aaada00/1*HKppSomeMK5U3Z0kbaRvkQ.png "首先先按「z」清除所有紀錄(避免搞亂)")
![開啟目標 APP](images/46410aaada00/1*mpNLXzUwb7-jiikrHkoTcA.png "開啟目標 APP")

開啟目標 APP 嘗試「下拉重整」或觸發「載入下一頁」的動作。
>  **_🛑若你的目標APP打不開、連不上；那抱歉了，代表APP有做防範無法用這招嗅探，請直接下拉到如何保護的章節🛑_** 
![mitmproxy 紀錄](images/46410aaada00/1*KOkJugn95bcUCPl-dZEaRA.png "mitmproxy 紀錄")

回到 mitmproxy 查看紀錄，發揮偵探的精神猜測哪個API請求紀錄是我們想要的並進入查看詳細！
![Request](images/46410aaada00/1*n6mUgej-2_U8PRUbQo_j1g.png "Request")

Request 部分可以看到 請求傳遞了哪些參數

搭配「e」編輯與「r」重新發送，並觀察 Response 就可以猜到每個參數的用途囉！
![Response](images/46410aaada00/1*zxdLiXMP-KapoEYou_TlZg.png "Response")

Response 也能直接獲得原始回傳內容。
>  **_🛑若Response內容是一堆編碼；那也抱歉了，代表APP可能有自己再做一次加解密無法用這招嗅探，請直接下拉到如何保護的章節🛑_** 

很難閱讀？中文亂碼？沒關係，這邊可以用「b」匯出成文字檔到桌面，再將內容複製到[Json Editor Online](https://jsoneditoronline.org/)解析即可!

![](images/46410aaada00/1*7qOTLAIQHH6V782OnvFVFQ.png "")
>  **_\*或是直接使用 mitmweb 使用 web gui 直接瀏覽、操作_** 
![mitmweb](images/46410aaada00/1*ujOlDBdjp8tECeAwRzWRPw.png "mitmweb")

經過嗅探、觀察、過濾、測試之後就能知道APP API的運作方式，藉此就能利用，用爬蟲爬取資料。
> _\*蒐集完所需資訊記得關閉mitmproxy、手機網路Proxy代理伺服器改回自動，才能正常使用網路。_
### APP 該如何自保？

若掛上 mitmproxy 之後發現APP不能用、回傳內容是編碼，代表APP有做保護。

**做法(1):**

大略是將憑證資訊放一份到APP中，若當前HTTPS使用的憑證與APP中的資訊不符則拒絕訪問，詳細可以[看此](https://www.anquanke.com/post/id/147090)或找[SSL Pinning](/@dzungnguyen.hcm/ios-ssl-pinning-bffd2ee9efc)相關資源。缺點可能就是要注意憑證有效期的問題吧！

![https://medium.com/@dzungnguyen.hcm/ios-ssl-pinning-bffd2ee9efc](images/46410aaada00/1*31rODDIlYPidTP3L8W_C7A.jpeg "https://medium.com/@dzungnguyen.hcm/ios-ssl-pinning-bffd2ee9efc")

**作法(2):**

APP端在資料要傳輸前先進行編碼加密，API後端收到後解密取得原始請求內容；API回傳內容一樣先進行編碼加密，APP端收到資料後解密取得回傳內容；步驟很煩瑣也耗效能，但的確是個方法；據我所知好像某數字銀行就是用這招進行保護!
#### 不過….

作法1，依然有破解方法：[如何在iOS 12上绕过SSL Pinning](https://www.anquanke.com/post/id/179514)


作法2，透過反編譯工程也能獲得編碼加密用的密鑰

 **⚠️沒有100%的安全⚠️** 

`或是乾脆挖個洞讓它爬，邊搜集各種證據，再用法務解決（？`
#### 還是那句話：
> 「 NEVER TRUST THE CLIENT」
### mitmproxy 的更多玩法：

 **1.使用mitmdump** 

除 `mitmproxy`、`mitmweb` ，`mitmdump` 可直接將所有紀錄匯出到文字檔中

```
mitmdump -w /log.txt
```

並且能使用**玩法(2)**python程式，設定、篩選流量:

```
mitmdump -ns examples/filter.py -r /log.txt -w /result.txt
```

 **2.搭配python程式做請求參數設定、訪問控制、轉址：** 
```Python
from mitmproxy import http

def request(flow: http.HTTPFlow) -> None:
    # pretty_host takes the "Host" header of the request into account,
    # which is useful in transparent mode where we usually only have the IP
    # otherwise.
    
    # 請求參數設定 Example:
    flow.request.headers['User-Agent'] = 'MitmProxy'
    
    if flow.request.pretty_host == "123.com.tw":
        flow.request.host = "456.com.tw"
    # 將123.com.tw的訪問全導到456.com.tw
```

啟用mitmproxy時加上參數：
```
mitmproxy -s /redirect.py
or
mitmweb -s /redirect.py
or
mitmdump -s /redirect.py
```
### 補個坑

在使用 mitmproxy 觀察使用 HTTP 1.1 及 Accept-Ranges: bytes、 Content-Range 長連接片段持續拿取資源的請求時，會等到 response 全部回來才會顯示，而不是顯示分段、使用持久連接接續下載！

[踩坑在這](avplayer-%E9%82%8A%E6%92%AD%E9%82%8A-cache-%E5%AF%A6%E6%88%B0-ee47f8f1e2d2)。
### 延伸閱讀
- [以簽到獎勵 APP 為例，打造每日自動簽到腳本](%E4%BD%BF%E7%94%A8-python-google-cloud-platform-line-bot-%E8%87%AA%E5%8B%95%E5%9F%B7%E8%A1%8C%E4%BE%8B%E8%A1%8C%E7%91%A3%E4%BA%8B-70a1409b149a)
- [如何打造一場有趣的工程CTF競賽](%E5%A6%82%E4%BD%95%E6%89%93%E9%80%A0%E4%B8%80%E5%A0%B4%E6%9C%89%E8%B6%A3%E7%9A%84%E5%B7%A5%E7%A8%8Bctf%E7%AB%B6%E8%B3%BD-729d7b6817a4)
- [揭露一個幾年前發現的巧妙網站漏洞](%E6%8F%AD%E9%9C%B2%E4%B8%80%E5%80%8B%E5%B9%BE%E5%B9%B4%E5%89%8D%E7%99%BC%E7%8F%BE%E7%9A%84%E5%B7%A7%E5%A6%99%E7%B6%B2%E7%AB%99%E6%BC%8F%E6%B4%9E-142244e5f07a)
- [iOS 15 / MacOS Monterey Safari 將能隱藏真實 IP](/zrealm-ios-dev/ios-15-macos-monterey-safari-%E5%B0%87%E8%83%BD%E9%9A%B1%E8%97%8F%E7%9C%9F%E5%AF%A6-ip-755a8b6acc35)

### 後記

因為我沒有網域權限無法取得ssl憑證資訊，所以也無法實作；看程式碼感覺並不困難，雖然沒有100%安全的方法，但多一道防護至少能更安全一些，再繼續攻需要花費很多時間研究，應該能勸退90%的爬蟲了！

這篇文章可能有點含金量太低，medium荒廢了一陣子(跑去玩單眼)，主要為本週六日(2019/09/21–2019/09/22)的 [iPlayground 2019](https://iplayground.io/2019/) 提前熱熱寫文手感；期待今年的議程🤩，希望回來後能吸收產出更多精品文章！

> **_[2019/02/22 更新文章]_** [**_iPlayground 2019 是怎麼樣的體驗？_**](iplayground-2019-%E6%98%AF%E6%80%8E%E9%BA%BC%E6%A8%A3%E7%9A%84%E9%AB%94%E9%A9%97-4079036c85c2)
[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/app%E6%9C%89%E7%94%A8https%E5%82%B3%E8%BC%B8-%E4%BD%86%E8%B3%87%E6%96%99%E9%82%84%E6%98%AF%E8%A2%AB%E5%81%B7%E4%BA%86-46410aaada00) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
