---
title: 現實使用 Codable 上遇到的 Decode 問題場景總匯(下)
author: ZhgChgLi
date: 2020-06-25T17:56:31.959Z
tags: [ios,ios-app-development,codable,json,core-data]
---

### 現實使用 Codable 上遇到的 Decode 問題場景總匯(下)

合理的處理 Response Null 欄位資料、不一定都要重寫 init decoder
![Photo by Zan](images/cb00b1977537/1*zoN0YxCnWdvMs35FaP5tNA.jpeg "Photo by Zan")
### 前言

既上篇「[現實使用 Codable 上遇到的 Decode 問題場景總匯](%E7%8F%BE%E5%AF%A6%E4%BD%BF%E7%94%A8-codable-%E4%B8%8A%E9%81%87%E5%88%B0%E7%9A%84-decode-%E5%95%8F%E9%A1%8C%E5%A0%B4%E6%99%AF%E7%B8%BD%E5%8C%AF-1aa2f8445642)」後，開發進度繼續邁進又遇到了新的場景新的問題，故出了此下篇，繼續把遇到的情景、研究心路都記錄下來，方便日後回頭查閱。


前篇主要解決了 JSON String -\> Entity Object 的 Decodable Mapping，有了 Entity Object 後我們可以轉換成 Model Object 在程式內傳遞使用、View Model Object 處理資料顯示邏輯…等等； **另一方面我們需要將 Entity 轉換成 NSManagedObject 存入本地 Core Data 中** 。

### 主要問題

假設我們的歌曲 Entity 結構如下：
```Swift
struct Song: Decodable {
    var id: Int
    var name: String?
    var file: String?
    var converImage: String?
    var likeCount: Int?
    var like: Bool?
    var length: Int?
}
```

因 API EndPoint 並不一定會回傳完整資料欄位(只有 id 是一定會給)，所以除 id 之外的欄位都是 Optional；例如：取得歌曲資訊的時候會回傳完整結構，但若是對歌曲收藏喜歡時僅會回傳 `id`、 `likeCount`、`like` 三個有關聯更動的欄位資料。


我們希望 API Response 有什麼欄位資料都能一併存入 Core Data 裡，如果資料已存在就更新變動的欄位資料（incremental update）。
> _但此時問題就出現了：Codable Decode 換成 Entity Object 後我們無法區別_ **_「資料欄位是想要設成 nil」 還是 「Response 沒給」_** 
```
 **B Response:**  
{  
 "id": 1,  
 "like": true,  
 "likeCount": 1  
}
```

對於 A Response、B Response 的 file 來說都是 null 、但意義不一一樣 ；A 是想把 file 欄位設為 null (清空原本資料)、 B 是想 update 其他資料，單純沒給 file 欄位而已。
> Swift 社群有開發者提出[增加類似 date Strategy 的 null Strategy 在 JSONDecoder 中](https://forums.swift.org/t/pitch-jsondecoder-nulldecodingstrategy/13980)，讓我們能區分以上狀況，但目前沒有計畫要加入。
> 
#### 解決方案

如前所述，我們的架構是JSON String -> Entity Object -> NSManagedObject，所以當拿到 Entity Object 時已經是 Decode 後的結果了，沒有 raw data 可以操作；這邊當然可以拿原始 JSON String 比對操作，但與其這樣不如不要用 Codable。

首先參考[上一篇](%E7%8F%BE%E5%AF%A6%E4%BD%BF%E7%94%A8-codable-%E4%B8%8A%E9%81%87%E5%88%B0%E7%9A%84-decode-%E5%95%8F%E9%A1%8C%E5%A0%B4%E6%99%AF%E7%B8%BD%E5%8C%AF-1aa2f8445642)使用 Associated Value Enum 當容器裝值。

```Swift
enum OptionalValue<T: Decodable>: Decodable {
    case null
    case value(T)
    init(from decoder: Decoder) throws {
        let container = try decoder.singleValueContainer()
        if let value = try? container.decode(T.self) {
            self = .value(value)
        } else {
            self = .null
        }
    }
}
```

使用泛型，T 為真實資料欄位型別；.value(T) 能放 Decode 出來的值、.null 則代表值是 null。
```Swift
struct Song: Decodable {
    enum CodingKeys: String, CodingKey {
        case id
        case file
    }
    
    var id: Int
    var file: OptionalValue<String>?
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        
        self.id = try container.decode(Int.self, forKey: .id)
        
        if container.contains(.file) {
            self.file = try container.decode(OptionalValue<String>.self, forKey: .file)
        } else {
            self.file = nil
        }
    }
}

var jsonData = """
{
    "id":1
}
""".data(using: .utf8)!
var result = try! JSONDecoder().decode(Song.self, from: jsonData)
print(result)

jsonData = """
{
    "id":1,
    "file":null
}
""".data(using: .utf8)!
result = try! JSONDecoder().decode(Song.self, from: jsonData)
print(result)

jsonData = """
{
    "id":1,
    "file":\"https://test.com/m.mp3\"
}
""".data(using: .utf8)!
result = try! JSONDecoder().decode(Song.self, from: jsonData)
print(result)
```
> _範例先簡化成只有_ `id`_、_`file` _兩個資料欄位。_

Song Entity 自行複寫實踐 Decode 方式，使用 `contains(.KEY)` 方法判斷 Response 有無給該欄位(無論值是什麼)，如果有就 Decode 成 OptionalVale ；OptionalValue Enum 中會再對真正我們要的值做 Decode ，如果有值 Decode 成功則會放在 .value(T) 、如果給的值是 null (或 decode 失敗)則放在 .null 。

. Response 有給欄位&值時：OptionalValue.value(VALUE)
. Response 有給欄位&值是 null 時：OptionalValue.null
. Response 沒給欄位時：nil

> _這樣就能區分出是有給欄位還是沒給欄位，後續要寫入 Core Data 時就能判斷是要更新欄位成 null、還是沒有要更新此欄位。_
#### 其他研究 — Double Optional ❌

Optional!Optional! 在 Swift 上就很適合處理這個場景。
```Swift
struct Song: Decodable {
    var id: Int
    var name: String??
    var file: String??
    var converImage: String??
    var likeCount: Int??
    var like: Bool??
    var length: Int??
}
```
. Response 有給欄位&值時：Optional(VALUE)
. Response 有給欄位&值是 null 時：Optional(nil)
. Response 沒給欄位時：nil


但是….Codable JSONDecoder Decode 對 Double Optional 跟 Optional 都是 decodeIfPresent 在處理，都視為 Optional ，不會特別處理 Double Optional；所以結果跟原本一樣。
#### 其他研究 — Property Wrapper ❌

本來預想可以用 Property Wrapper 做優雅的封裝，例如：
```
@OptionalValue var file: String?
```

但還沒開始研究細節就發現有 Property Wrapper 標記的 Codable Property 欄位，API Response 就必須要有該欄位，否則會出現 keyNotFound error，即使該欄位是 Optional。?????

官方論壇也有針對此問題的[討論串](https://forums.swift.org/t/using-property-wrappers-with-codable/29804)…估計之後會修正。

> 所以選用 [BetterCodable](https://github.com/marksands/BetterCodable)、[CodableWrappers](https://github.com/GottaGetSwifty/CodableWrappers) 這類套件的時候要考慮到目前 Property Wrapper 的這個問題。
> 
### 其他問題場景
#### 1.API Response 使用 0/1 代表 Bool，該如何 Decode?
```Swift
import Foundation

struct Song: Decodable {
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case like
    }
    
    var id: Int
    var name: String?
    var like: Bool?
    
    init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.id = try container.decode(Int.self, forKey: .id)
        self.name = try container.decodeIfPresent(String.self, forKey: .name)
        
        if let intValue = try container.decodeIfPresent(Int.self, forKey: .like) {
            self.like = (intValue == 1) ? true : false
        } else if let boolValue = try container.decodeIfPresent(Bool.self, forKey: .like) {
            self.like = boolValue
        }
    }
}

var jsonData = """
{
    "id": 1,
    "name": "告五人",
    "like": 0
}
""".data(using: .utf8)!
var result = try! JSONDecoder().decode(Song.self, from: jsonData)
print(result)

```

延伸前篇，我們可以自己在 init Decode 中，Decode 成 int/Bool 然後自己賦值、這樣就能擴充原本的欄位能接受 0/1/true/false了。
#### 2.不想要每每都要重寫 init decoder

在不想要自幹 Decoder 的情況下，複寫原本的 JSON Decoder 擴充更多功能。

我們可以自行 extenstion [KeyedDecodingContainer](https://developer.apple.com/documentation/swift/keyeddecodingcontainer) 對 public 方法自行定義，swift 會優先執行 module 下我們重定義的方法，複寫掉原本 Foundation 的實作。

> **_影響的就是整個 module。  
> 且不是真的 override，無法 call super.decode，也要小心不要自己 call 自己(EX: decode(Bool.Type,for:key) in decode(Bool.Type,for:key))_**  

 **decode 有兩個方法：** 
- **decode(Type, forKey:)** 處理非 Optional 資料欄位
- **decodeIfPresent(Type, forKey:)** 處理 Optional 資料欄位


 **範例1. 前述的主要問題就我們可以直接 extenstion：** 
```Swift
extension KeyedDecodingContainer {
    public func decodeIfPresent<T>(_ type: T.Type, forKey key: Self.Key) throws -> T? where T : Decodable {
        //better:
        switch type {
        case is OptionalValue<String>.Type,
             is OptionalValue<Int>.Type:
            return try? decode(type, forKey: key)
        default:
            return nil
        }
        // or just return try? decode(type, forKey: key)
    }
}

struct Song: Decodable {
    var id: Int
    var file: OptionalValue<String>?
}
```

因主要問題是 Optional 資料欄位、Decodable 類型，所以我們複寫的是 decodeIfPresent<T: Decodable> 這個方法。

這邊推測原本 decodeIfPresent 的實作是，如果資料是 null 或 Response 未給 會直接 return nil，並不會真的跑 decode。

所以原理也很簡單，只要 Decodable Type 是 OptionValue<T> 則不論如何都 decode 看看，我們才能拿到不同狀態結果；但其實不判斷 Decodable Type 也行，那就是所有 Optional 欄位都會試著 Decode。

 **範例2. 問題場景1 也能用此方法擴充：** 
```Swift
extension KeyedDecodingContainer {
    public func decodeIfPresent(_ type: Bool.Type, forKey key: KeyedDecodingContainer<K>.Key) throws -> Bool? {
        if let intValue = try? decodeIfPresent(Int.self, forKey: key) {
            return (intValue == 1) ? (true) : (false)
        } else if let boolValue = try? decodeIfPresent(Bool.self, forKey: key) {
            return boolValue
        }
        return nil
    }
}

struct Song: Decodable {
    enum CodingKeys: String, CodingKey {
        case id
        case name
        case like
    }
    
    var id: Int
    var name: String?
    var like: Bool?
}

var jsonData = """
{
    "id": 1,
    "name": "告五人",
    "like": 1
}
""".data(using: .utf8)!
var result = try! JSONDecoder().decode(Song.self, from: jsonData)
print(result)
```
### 結語

Codable 在使用上的各種奇技淫巧都用的差不多了，有些其實很繞，因為 Codable 的約束性實在太強、犧牲許多現實開發上需要的彈性；做到最後甚至開始思考為何當初要選擇 Codable，優點越做越少….
#### 參考資料
- [或许你并不需要重写 init(from:)方法](https://kemchenj.github.io/2018-07-09/)

### 回看
- [現實使用 Codable 上遇到的 Decode 問題場景總匯(上)](%E7%8F%BE%E5%AF%A6%E4%BD%BF%E7%94%A8-codable-%E4%B8%8A%E9%81%87%E5%88%B0%E7%9A%84-decode-%E5%95%8F%E9%A1%8C%E5%A0%B4%E6%99%AF%E7%B8%BD%E5%8C%AF-1aa2f8445642)

[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://blog.zhgchg.li/%E7%8F%BE%E5%AF%A6%E4%BD%BF%E7%94%A8-codable-%E4%B8%8A%E9%81%87%E5%88%B0%E7%9A%84-decode-%E5%95%8F%E9%A1%8C%E5%A0%B4%E6%99%AF%E7%B8%BD%E5%8C%AF-%E4%B8%8B-cb00b1977537) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
