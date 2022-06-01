---
title: Vision 初探 — APP 頭像上傳 自動識別人臉裁圖 (Swift)
author: ZhgChgLi
date: 2018-10-16T16:01:24.511Z
tags: [swift,machine-learning,facedetection,ios,ios-app-development]
---

### Vision 初探 — APP 頭像上傳 自動識別人臉裁圖 (Swift)

Vision 實戰應用
#### 一樣不多說，先上一張成品圖：
![優化前 V.S 優化後 — 結婚吧APP](images/9a9aa892f9a9/1*c-ioRH_Z2nMYRxSbuBD71A.png "優化前 V.S 優化後 — 結婚吧APP")

前陣子iOS 12發佈更新，注意到新開放的CoreML 機器學習框架；覺得挺有趣的，就開始構想如果想用在當前的產品上能放在哪裡？
>  **CoreML嚐鮮文章現已發佈：** [**使用機器學習自動預測文章分類，連模型也自己訓練**](%E5%9A%90%E9%AE%AE-ios-12-coreml-%E4%BD%BF%E7%94%A8%E6%A9%9F%E5%99%A8%E5%AD%B8%E7%BF%92%E8%87%AA%E5%8B%95%E9%A0%90%E6%B8%AC%E6%96%87%E7%AB%A0%E5%88%86%E9%A1%9E-%E9%80%A3%E6%A8%A1%E5%9E%8B%E4%B9%9F%E8%87%AA%E5%B7%B1%E8%A8%93%E7%B7%B4-793bf2cdda0f)

CoreML提供文字、圖像的機器學習模型訓練及引用到APP裡的接口，我原先的想法是，使用CoreML來做到人臉識別，解決APP中有裁圖的項目頭或臉被卡掉的問題，如上圖左所示，若人臉出現在周圍則很容易因為縮放＋裁圖造成臉不完整．

經過網路搜尋一番後才發現我學識短淺，這個功能在iOS 11就已發佈：「Vision」框架，支援文字偵測、人臉偵測、圖像比對、QRCODE偵測、物件追蹤…功能

這邊使用的就是其中的人臉偵測項目，經優化後如右圖所示；找到人臉並以此為中心裁圖．
### 實戰開始：
#### 首先我們先做能標記人臉位置的功能，初步認識一下Vision怎麼用
![Demo APP](images/9a9aa892f9a9/1*cpGgpXsBhuiJoZI03WAGUw.png "Demo APP")

完成圖如上所示，能標記出照片中人臉的位置

p.s 僅能標記「人臉」，整個頭包含頭髮並不行😅

這塊程式主要分為兩部分，第一部分要解決 圖片原尺寸縮放放入 ImageView時會留白的狀況；簡單來說我們要的是Image的Size多大，ImageView的Size就有多大，若直接放入圖片會造成如下走位情形
![](images/9a9aa892f9a9/1*Mb70Ed6pALO-8sllCpb7Qg.png "")

你可能會想說直接改ContentMode變成fill、fit、redraw，但就會變形或圖片被卡掉
```Swift
let ratio = UIScreen.main.bounds.size.width
//這邊是因為我UIIMAGEVIEW 那邊設定左右對齊0，寬高比1:1

let sourceImage = UIImage(named: "Demo2")?.kf.resize(to: CGSize(width: ratio, height: CGFloat.leastNonzeroMagnitude), for: .aspectFill)
//使用KingFisher的圖片變形功能，已寬為基準，高度自由

imageView.contentMode = .redraw
//contentMode使用redraw填滿

imageView.image = sourceImage
//賦予圖片

imageViewConstraints.constant = (ratio - (sourceImage?.size.height ?? 0))
imageView.layoutIfNeeded()
imageView.sizeToFit()
//這一塊是我去改變 imageView的Constraints，詳情可看文末完整範例
```

以上就是針對圖片做的處理

_裁圖部分使用Kingfisher幫助我們，也可替換成其他套件或自刻方法_

第二部分，進入重點直接看Code
```Swift
if #available(iOS 11.0, *) {
    //iOS 11之後才支援
    let completionHandle: VNRequestCompletionHandler = { request, error in
        if let faceObservations = request.results as? [VNFaceObservation] {
            //辨識到的臉臉們
            
            DispatchQueue.main.async {
                //操作UIVIEW，切回主執行緒
                let size = self.imageView.frame.size
                
                faceObservations.forEach({ (faceObservation) in
                    //坐標系轉換
                    let translate = CGAffineTransform.identity.scaledBy(x: size.width, y: size.height)
                    let transform = CGAffineTransform(scaleX: 1, y: -1).translatedBy(x: 0, y: -size.height)
                    let transRect =  faceObservation.boundingBox.applying(translate).applying(transform)
                    
                    let markerView = UIView(frame: transRect)
                    markerView.backgroundColor = UIColor.init(red: 0/255, green: 255/255, blue: 0/255, alpha: 0.3)
                    self.imageView.addSubview(markerView)
                })
            }
        } else {
            print("未偵測到任何臉")
        }
    }
    
    //辨識請求
    let baseRequest = VNDetectFaceRectanglesRequest(completionHandler: completionHandle)
    let faceHandle = VNImageRequestHandler(ciImage: ciImage, options: [:])
    DispatchQueue.global().async {
        //辨識需要時間，所以放入背景子執行緒執行，避免當前畫面卡住
        do{
            try faceHandle.perform([baseRequest])
        }catch{
            print("Throws：\(error)")
        }
    }
  
} else {
    //
    print("不支援")
}

```

主要要注意的是，坐標系轉換部分；辨識出來的結果是Image的原始座標；我們須將它轉換成包在外面的ImageView的實際座標才能正確地使用它．
#### 再來我們來做今天的重頭戲 — 依照人臉的位置裁切出大頭貼的正確位置
```Swift
let ratio = UIScreen.main.bounds.size.width
//這邊是因為我UIIMAGEVIEW 那邊設定左右對齊0，寬高比1:1，詳情可看文末完整範例

let sourceImage = UIImage(named: "Demo")

imageView.contentMode = .scaleAspectFill
//使用scaleAspectFill模式填滿

imageView.image = sourceImage
//直接賦予原圖片，我們之後再操作

if let image = sourceImage,#available(iOS 11.0, *),let ciImage = CIImage(image: image) {
    let completionHandle: VNRequestCompletionHandler = { request, error in
        if request.results?.count == 1,let faceObservation = request.results?.first as? VNFaceObservation {
            //ㄧ張臉
            let size = CGSize(width: ratio, height: ratio)
            
            let translate = CGAffineTransform.identity.scaledBy(x: size.width, y: size.height)
            let transform = CGAffineTransform(scaleX: 1, y: -1).translatedBy(x: 0, y: -size.height)
            let finalRect =  faceObservation.boundingBox.applying(translate).applying(transform)
            
            let center = CGPoint(x: (finalRect.origin.x + finalRect.width/2 - size.width/2), y: (finalRect.origin.y + finalRect.height/2 - size.height/2))
            //這裡是計算臉的範圍中間點位置
            
            let newImage = image.kf.resize(to: size, for: .aspectFill).kf.crop(to: size, anchorOn: center)
            //將圖片依照中間點裁切
            
            DispatchQueue.main.async {
                //操作UIVIEW，切回主執行緒
                self.imageView.image = newImage
            }
        } else {
            print("偵測到多張臉或沒有偵測到臉")
        }
    }
    let baseRequest = VNDetectFaceRectanglesRequest(completionHandler: completionHandle)
    let faceHandle = VNImageRequestHandler(ciImage: ciImage, options: [:])
    DispatchQueue.global().async {
        do{
            try faceHandle.perform([baseRequest])
        }catch{
            print("Throws：\(error)")
        }
    }
} else {
    print("不支援")
}
```

道理跟標記人臉位置差不多，差別在大頭貼的部分是固定尺寸(如:300x300)，所以我們略過前面需要讓Image適應ImageView的第一部分

另一個差別是我們要多計算人臉範圍的中心點，並以這個中心點為準做裁切圖片
![紅點為臉的範圍中心點](images/9a9aa892f9a9/1*civytcKOguHfVFHYPVWecA.png "紅點為臉的範圍中心點")
#### 完成效果圖：
![頓丹前的那一秒是原始圖位置](images/9a9aa892f9a9/1*WocYjt0xLkqtGVilxfT2LA.gif "頓丹前的那一秒是原始圖位置")
### 完整APP範例:
![](images/9a9aa892f9a9/1*J8oByw8gBCamIac2TkT1SA.gif "")

程式碼已上傳至Github：[請點此](https://github.com/zhgchgli0718/VisionDemo)

[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/vision-%E5%88%9D%E6%8E%A2-app-%E9%A0%AD%E5%83%8F%E4%B8%8A%E5%82%B3-%E8%87%AA%E5%8B%95%E8%AD%98%E5%88%A5%E4%BA%BA%E8%87%89%E8%A3%81%E5%9C%96-swift-9a9aa892f9a9) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
