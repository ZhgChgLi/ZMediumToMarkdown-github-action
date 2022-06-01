---
title: iOS ≥ 12 在使用者的「設定」中增加「APP通知設定頁」捷徑 (Swift)
author: ZhgChgLi
date: 2018-11-12T14:38:42.897Z
tags: [ios-app-development,ios,swift,push-notification,ios-12]
---

### iOS ≥ 12 在使用者的「設定」中增加「APP通知設定頁」捷徑 (Swift)

除了從系統關閉通知，讓使用者還有其他選擇
#### 緊接著前三篇文章：
- [iOS ≥ 10 Notification Service Extension 應用 (Swift)](ios-10-notification-service-extension-%E6%87%89%E7%94%A8-swift-cb6eba52a342)
- [什麼？iOS 12 不需使用者授權就能傳送推播通知(Swift)](%E4%BB%80%E9%BA%BC-ios-12-%E4%B8%8D%E9%9C%80%E4%BD%BF%E7%94%A8%E8%80%85%E6%8E%88%E6%AC%8A%E5%B0%B1%E8%83%BD%E6%94%B6%E5%88%B0%E6%8E%A8%E6%92%AD%E9%80%9A%E7%9F%A5-swift-ade9e745a4bf)
- [從 iOS 9 到 iOS 12 推播通知權限狀態處理(Swift)](%E5%BE%9E-ios-9-%E5%88%B0-ios-12-%E6%8E%A8%E6%92%AD%E9%80%9A%E7%9F%A5%E6%AC%8A%E9%99%90%E7%8B%80%E6%85%8B%E8%99%95%E7%90%86-swift-fd7f92d52baa)


我們繼續針對推播進行改進，不管是原有的技術或是新開放的功能，都來嘗試嘗試！
### 這次是啥？

iOS ≥ 12 可以在使用者的「設定」中增加您的APP通知設定頁面捷徑，讓使用者想要調整通知時，能有其他選擇；可以跳轉到「APP內」而不是從「系統面」直接關閉，ㄧ樣不囉唆先上圖：
![「設定」->「APP」->「通知」->「在APP中設定」](images/f644db1bb8bf/1*BAdVMElIjgg34meOSdHhOw.gif "「設定」->「APP」->「通知」->「在APP中設定」")

另外在使用者收到通知時，若欲使用3D Touch調整設定「關閉」通知，會多一個「在APP中設定」的選項供使用者選擇
![「通知」->「3D Touch」->「…」->「關閉…」->「在APP中設定」](images/f644db1bb8bf/1*KMKbYQU3nPfF9XpMS5NbPQ.gif "「通知」->「3D Touch」->「…」->「關閉…」->「在APP中設定」")
### 怎麼實作？

這部分的實作非常簡單，第一步僅需在要求推播權限時多要求一個 **.providesAppNotificationSettings** 權限即可

```Swift
//appDelegate.swift didFinishLaunchingWithOptions or....
if #available(iOS 12.0, *) {
    let center = UNUserNotificationCenter.current()
    let permissiones:UNAuthorizationOptions = [.badge, .alert, .sound, .provisional,.providesAppNotificationSettings]
    center.requestAuthorization(options: permissiones) { (granted, error) in
        
    }
}
```
![](images/f644db1bb8bf/1*_xztNYANTU6ilOXY_qKOKA.png "")

在詢問過使用者要不要允許通知之後，通知若為開啟狀態下方就會出現選項囉（ **不論前面使用者按允許或不允許** ）。

#### 第二步：

第二步，也是最後一步；我們要讓 **appDelegate** 遵守 **UNUserNotificationCenterDelegate** 代理並實作 **userNotificationCenter(\_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?)** 方法即可！

```Swift
//appDelegate.swift
import UserNotifications
@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {
    var window: UIWindow?
    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        if #available(iOS 10.0, *) {
            UNUserNotificationCenter.current().delegate = self
        }
        
        return true
    }
    //其他部份省略...
}
extension AppDelegate: UNUserNotificationCenterDelegate {
    @available(iOS 10.0, *)
    func userNotificationCenter(_ center: UNUserNotificationCenter, openSettingsFor notification: UNNotification?) {
        //跳轉到你的設定頁面位置..
        //EX:
        //let VC = SettingViewController();
        //self.window?.rootViewController.present(alertController, animated: true)
    }
}
```
- 在Appdelegate的didFinishLaunchingWithOptions中實現代理
- Appdelegate遵守代理並實作方法


完成！相較於前幾篇文章，這個功能實作相較起來非常簡單 🏆
### 總結

這個功能跟[前一篇](%E4%BB%80%E9%BA%BC-ios-12-%E4%B8%8D%E9%9C%80%E4%BD%BF%E7%94%A8%E8%80%85%E6%8E%88%E6%AC%8A%E5%B0%B1%E8%83%BD%E6%94%B6%E5%88%B0%E6%8E%A8%E6%92%AD%E9%80%9A%E7%9F%A5-swift-ade9e745a4bf)提到的先不用使用者授權就發干擾性較低的靜音推播給使用者試試水溫有點類似！


都是在開發者與使用者之前架起新的橋樑，以往APP太吵，我們會直接進到設定頁無情地關閉所有通知，但這樣對開發者來說，以後不管好的壞的有用的…任何通知都無法再發給使用者，使用者可能也因此錯過重要消息或限定優惠．

這個功能讓使用者欲關閉通知時能有進到APP調整通知的選擇，開發者可以針對推播項目細分，讓使用者決定自己想要收到什麼類型的推播。
![](images/f644db1bb8bf/1*ju98WxxFonEimTx2tEFO3Q.jpeg "")

以[結婚吧APP](https://itunes.apple.com/tw/app/%E7%B5%90%E5%A9%9A%E5%90%A7-%E4%B8%8D%E6%89%BE%E6%9C%80%E8%B2%B4-%E5%8F%AA%E6%89%BE%E6%9C%80%E5%B0%8D/id1356057329?ls=1&mt=8)來說，使用者若覺得專欄通知太干擾，可個別關閉；但依然能收到重要系統消息通知．

>  **p.s 個別關閉通知功能是我們APP本來就有的功能，但透過結合iOS ≥12的新通知特性能有更好的效果及使用者體驗的提升** 
![](images/f644db1bb8bf/1*DEOMdPwDxyHca-GnYr8HIQ.jpeg "")
[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/ios-12-%E5%9C%A8%E4%BD%BF%E7%94%A8%E8%80%85%E7%9A%84-%E8%A8%AD%E5%AE%9A-%E4%B8%AD%E5%A2%9E%E5%8A%A0-app%E9%80%9A%E7%9F%A5%E8%A8%AD%E5%AE%9A%E9%A0%81-%E6%8D%B7%E5%BE%91-swift-f644db1bb8bf) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
