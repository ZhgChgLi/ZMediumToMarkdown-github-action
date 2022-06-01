---
title: iOS UUID 的那些事 (Swift/iOS ≥ 6)
author: ZhgChgLi
date: 2018-10-25T14:26:20.002Z
tags: [iplayground,swift,ios-app-development,uuid,idfv]
---

### iOS UUID 的那些事 (Swift/iOS ≥ 6)

iPlayground 2018 回來 & UUID那些事
### 前言：

上週六、日跑去參加 [iPlayground](https://iplayground.io/) Apple 軟體開發者研討會，這個活動訊息是同事PASS過來的，去之前我也不清楚這個活動。

![](images/a4bc3bce7513/1*gEmmuDOD92d2b2fLp4AKsw.jpeg "")

兩天下來，整題活動跟時程安排流暢，議程內容：
. 趣味的：腳踏車、凋零的Code、iOS/API 演進史、威利在哪裡(CoreML Vision)
. 實用的：測試類 (XCUITest、依賴注入)、SpriteKit 做動畫效果的替代方案、GraphQL
. 真功夫：深入拆解Swift、iOS 越獄/Tweak開發、Redux


腳踏車Project 印象深刻，用iPhone手機當感測器感測腳踏車踏板轉動，直接在台上騎腳踏車切換投影片(前輩主要目標是要做開源版zwift，也分享了許多地雷，例如Client/Sever通信、延遲問題、磁場干擾)

凋零的Dirty Code；聽得心有戚戚，在心裡會心一笑；技術債就是這樣一直累積下來的，開發時程趕，所以用架構性較差的快速做法，後人接手改也沒時間重構，就越積越多；到最後可能真的只有打掉這條路了

測試類(Design Patterns in XCUITest) [KKBOX的前輩](https://www.facebook.com/TestingWithKK/)，完全沒藏私直接公開他們的作法及程式範例細節還有遇到的雷、解決辦法，這堂也是對我們工作上最有幫助的項目；測試這塊是我一直想加強的部分，可以回去好好研究研究


Lighting Talk的部分在台下聽得也好想上去分享😂 下次要提早做好準備了!

會後的offical party，酒水食物場地都很有誠意，聽前輩們的真心話吐露，很輕鬆有趣之外還吸收許多職場軟實力．
![台大後台咖啡](images/a4bc3bce7513/1*Xwk_96lVKcMKgeL7IOC70g.jpeg "台大後台咖啡")

我才知道原來這是第一屆，真的有榮幸能夠參加，所有工作人員跟講者辛苦了！

去參加研討會的目的不外乎就是要： **增加廣度** ，吸收新知、了解生態、碰一些平常不會接觸的項目跟 **增加深度** ，如果是自己已經摸過的項目就是去聽聽看有沒有遺漏的地方或是還有其他做法沒發現．


抄了許多筆記可以回來慢慢研究回味。
### UUID的那些事

因為我聽完回去後馬上實際應用到APP上；這堂是由Zonble前輩主講，我聽到從iPhone OS 2寫到iOS 12我就跪了；由於入行較晚，我是從iOS 11/Swift 4 才開始寫，所以沒碰到那些因為蘋果修改API的動亂時期。

想想UUID從可以取得到封鎖也是蠻合理的；如果是用在良善的地方：辨識使用者裝置、廣告或第三方運用唯一性去做廣告操作；但如果有廠商想做惡，也可以透過這個機制反查，知道你這隻手機的主人是怎麼樣的人？(例如有裝旅遊+台北等公車＋BMW APP+嬰兒照護 就能推測你很常出國家裡有小孩而且住在台北 之類的資訊)再加上你在APP上輸入的個資，能拿去做什麼應用不敢想像

但這其中也波及到很多正當守法的用戶，像是本來用UUID當使用者的資料解密KEY或用UUID當裝置判斷都受到很大的影響；真佩服那個時期的工程師前輩們，這些影響老闆跟使用者一定會狂罵，要急中生智找其他替代辦法．
#### 替代方案：

本篇文章以取得UUID辨識裝置唯一值為主，如果是要找知道使用者裝了哪些APP的替代方案可參考以下關鍵字搜尋做法：[UIPasteboard pasteboardWithName: create: (運用剪貼簿在APP間共享)](https://link.medium.com/YTheNPnHH7)、canOpenURL: info.plist LSApplicationQueriesSchmes (運用canOpenURL檢查APO有無安裝，要在info.plist列舉，最多50筆)

. 用MAC Address當UUID，但後來也被BAN了
. [Finger Printing (Canvas/User-Agent…)](/@ravielakshmanan/web-browser-uniqueness-and-fingerprinting-7eac3c381805)：沒研究，不過這項目主要拿來讓safari跟app能產生同樣的UUID， [Deferred Deep Linking](https://www.jianshu.com/p/fa48387d56ea) (延遲深度連結)用  
[AmIUnique?](https://amiunique.org/)
. [**ID** entifier **F** or **V** endor](https://www.jianshu.com/p/b810d7e007ad) (IDFV)：目前主流的解決方案🏆  
概念是蘋果會根據你的Bundle ID前輟為使用者產生UUID，相同的Bundle ID前輟會產生相同的UUID，例如:com.518.work/com.518.job 同個裝置會得到相同的UUID  
如同原文ID For Vendor，相同的前輟蘋果認為即是相同廠商的APP，所以共享UUID是允許的。

####  **ID** entifier **F** or **V** endor (IDFV)：
```
 **let** DEVICE\_UUID:String = UIDevice.current.identifierForVendor?.uuidString ?? UUID().uuidString
```

**唯需注意：當所有同Vendor的APP都移除後再重裝就會產生新的UUID (**com.518.work跟com.518.job都被刪除，再裝回com.518.work這時就會產生新的UUID**)  
同理如果你只有一個APP，刪掉重裝就會產生新的UUID**  

因為這個特性，我們公司的其他APP是使用Key-Chain來解決這個問題，聽了講者前輩的指點也驗證了這個做法是正確的！

 **流程如下：** 
![Key-Chain UUID欄位有值時取值，無則取IDFA的UUID值並回寫](images/a4bc3bce7513/1*-8rufG1QW-J5tn6ZadT17A.jpeg "Key-Chain UUID欄位有值時取值，無則取IDFA的UUID值並回寫")

Key-Chain寫入方式：
```Swift
if let data = DEVICE_UUID.data(using: .utf8) {
    let query = [
        kSecClass as String       : kSecClassGenericPassword as String,
        kSecAttrAccount as String : "DEVICE_UUID",
        kSecValueData as String   : data ] as [String : Any]
    
    SecItemDelete(query as CFDictionary)
    SecItemAdd(query as CFDictionary, nil)
}
```

Key-Chain讀取方式：
```Swift
let query = [
    kSecClass as String       : kSecClassGenericPassword,
    kSecAttrAccount as String : "DEVICE_UUID",
    kSecReturnData as String  : kCFBooleanTrue,
    kSecMatchLimit as String  : kSecMatchLimitOne ] as [String : Any]

var dataTypeRef: AnyObject? = nil
let status: OSStatus = SecItemCopyMatching(query as CFDictionary, &dataTypeRef)
if status == noErr,let dataTypeRef = dataTypeRef as? Data,let uuid = String(data:dataTypeRef, encoding: .utf8) {
   //uuid
} 
```

如果嫌Key-Chain操作太繁瑣可以自行封裝或使用第三方套件。
#### 完整CODE：
```Swift
let DEVICE_UUID:String = {
    let query = [
        kSecClass as String       : kSecClassGenericPassword,
        kSecAttrAccount as String : "DEVICE_UUID",
        kSecReturnData as String  : kCFBooleanTrue,
        kSecMatchLimit as String  : kSecMatchLimitOne ] as [String : Any]
    
    var dataTypeRef: AnyObject? = nil
    let status: OSStatus = SecItemCopyMatching(query as CFDictionary, &dataTypeRef)
    if status == noErr,let dataTypeRef = dataTypeRef as? Data,let uuid = String(data:dataTypeRef, encoding: .utf8) {
        return uuid
    } else {
        let DEVICE_UUID:String = UIDevice.current.identifierForVendor?.uuidString ?? UUID().uuidString
        if let data = DEVICE_UUID.data(using: .utf8) {
            let query = [
                kSecClass as String       : kSecClassGenericPassword as String,
                kSecAttrAccount as String : "DEVICE_UUID",
                kSecValueData as String   : data ] as [String : Any]
        
            SecItemDelete(query as CFDictionary)
            SecItemAdd(query as CFDictionary, nil)
        }
        return DEVICE_UUID
    }
}()
```
[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/ios-uuid-%E7%9A%84%E9%82%A3%E4%BA%9B%E4%BA%8B-swift-ios-6-a4bc3bce7513) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
