---
title: iOS tintAdjustmentMode 屬性
author: ZhgChgLi
date: 2019-02-06T16:10:43.225Z
tags: [uikit,swift,ios-app-development,autolayout,顧小事成大事]
---

### iOS tintAdjustmentMode 屬性

Present UIAlertController 時本頁上的 Image Assets (Render as template) .tintColor 設定失效問題
### 顧小事成大事的第一篇：
> _2019年新主題，「_ **_顧小事成大事_** _」意指_ **_完善小細節聚沙成塔成大事_** _，如同郭董說的「_ **_魔鬼藏在細節裡_** _」；主要都是整理_ **_小問題及解決方法_** _，另一方面也當筆記紀錄，如果你也有發現一樣的問題希望能幫助到你：）_
### 問題修正前後比較

ㄧ樣不囉唆解釋，直接上比較圖．
![左修正前/右修正後](images/6012b7b4f612/1*zwbk9bi9RKQ-MEuzlQHosA.jpeg "左修正前/右修正後")

可以看到左方ICON圖在有Present UIAlertController時tintColor顏色設定失效，另外當Present的視窗關閉後就會恢復顏色設定顯示正常．
#### 問題修正

首先介紹一下 **tintAdjustmentMode** 的屬性設置，此屬性控制了 **tintColor** 的顯示模式，此屬性有三個枚舉可設定：

.  **.Automatic** ：視圖的 **tintAdjustmentMode** 與包覆的父視圖設定一致
.  **.Normal** ： **預設模式** ，正常顯示設定的 **tintColor** 
.  **.Dimmed** ：將 **tintColor** 改為低飽和度、暗淡的顏色（就是灰色啦！）

#### _上述問題不是什麼BUG而是系統本身機制即是如此：_
> _在Present UIAlertController時會將本頁Root ViewController上View的_ **_tintAdjustmentMode_** _改為_ **_Dimmed_** _（所以準確來說也不叫顏色設定「失效」，只是_ **_tintAdjustmentMode_** _模式更改）_

但有時我們希望ICON顏色能保持ㄧ致則只需在UIView中tintColorDidChange事件保持tintAdjustmentMode設定ㄧ致：
```Swift
extension UIButton { 
   override func tintColorDidChange() {
        self.tintAdjustmentMode = .normal //永遠保持normal
    }
}
```
#### 結束！

不是什麼大問題，不改也沒差，但就是礙眼

其實每一個頁面遇到present UIAlertController、action sheet、popover…都會將本頁view的tintAdjustmentMode改為灰色，但我在這個頁面才發現

查找了一陣子資料才發現跟這個屬性有關係，設定之後就解決我的小疑惑．
[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/%E9%A1%A7%E5%B0%8F%E4%BA%8B%E6%88%90%E5%A4%A7%E4%BA%8B-1-ios-tintadjustmentmode-%E5%B1%AC%E6%80%A7-6012b7b4f612) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
