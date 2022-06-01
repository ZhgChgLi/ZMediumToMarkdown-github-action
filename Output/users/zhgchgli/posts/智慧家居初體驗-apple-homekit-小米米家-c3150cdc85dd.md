---
title: 智慧家居初體驗 - Apple HomeKit & 小米米家
author: ZhgChgLi
date: 2019-07-05T17:13:47.487Z
tags: [生活,開箱,3c,米家,homekit]
---

### 智慧家居初體驗 - Apple HomeKit & 小米米家

米家智慧攝影機及米家智慧檯燈、Homekit設定教學

**[2020/04/20]** [**進階篇已發**](%E6%89%93%E9%80%A0%E8%88%92%E9%81%A9%E7%9A%84-wfh-%E6%99%BA%E6%85%A7%E5%B1%85%E5%AE%B6%E7%92%B0%E5%A2%83-%E6%8E%A7%E5%88%B6%E5%AE%B6%E9%9B%BB%E7%9B%A1%E5%9C%A8%E6%8C%87%E5%B0%96-99db2a1fbfe5) **：**  
[**有經驗的朋友請直接左轉前往\>\>**  **示範使用樹莓派當 HomeBridge 主機，將所有米家家電串上 HomeKit**](%E6%89%93%E9%80%A0%E8%88%92%E9%81%A9%E7%9A%84-wfh-%E6%99%BA%E6%85%A7%E5%B1%85%E5%AE%B6%E7%92%B0%E5%A2%83-%E6%8E%A7%E5%88%B6%E5%AE%B6%E9%9B%BB%E7%9B%A1%E5%9C%A8%E6%8C%87%E5%B0%96-99db2a1fbfe5)
### 雜談：

最近剛搬完家；有別於原本住的地方，天花板是辦公室輕鋼架燈，亮到要拔掉幾根燈管眼睛才比較舒適；現在住的地方則是裝潢反射燈，使用電腦、看書亮度稍嫌不足，兩週下來眼睛覺得更容易乾澀不舒服；本想直接去IKEA採購，但考量到光色、護眼，最後比較一下CP值，於是還是選擇了小米檯燈(加上之前已有買小米智慧攝影機，都是米家系列產品)。
### 本篇：

其實我在選購時並沒有特別注意是否支援Apple HomeKit，身為一個iOS 開發者實在太失格了，因為我壓跟沒想到小米會支援．

所以本篇會分別介紹 **Apple HomeKit 使用** 、 **不支援Apple HomeKit的智慧家居怎麼使用第三方串接HomeKit？** 及 **使用米家本身搭建智慧家庭的方法（搭配IFTTT）**


大家可以根據自己的裝置需求跳著看。
### 採買：

我一共買了兩盞檯燈，一盞(Pro)放電腦桌工作用、另一盞放床頭當閱讀燈。
#### [米家檯燈 Pro](https://www.mi.com/tw/mi-adjustable-smart-desk-lamp/)：
![NT$ 1,795 支援米家、Apple HomeKit](images/c3150cdc85dd/1*Aa9zfAh7xclVOZS0IkcaMQ.jpeg "NT$ 1,795 支援米家、Apple HomeKit")
#### [米家 LED 智慧檯燈](https://www.mi.com/tw/smartlamp/)：
![NT$ 995 僅支援米家](images/c3150cdc85dd/1*PCaRI8AroRgFELEA1elaiA.jpeg "NT$ 995 僅支援米家")

詳細介紹可參考官網，兩盞都支援智慧控制、變色、調亮度、護眼，Pro版支援Apple HomeKit、三段角度調整；目前使用下來，以一盞燈的功能來說已經相當滿意，硬要挑一個缺點的話就是Pro版的角度調整只有底座能水平轉，燈不行，這樣就不能調整光線的角度了！
### 理想的智慧家居目標：
#### 目前有的裝置：
. 米家智慧攝影機雲台版 1080P (支援：米家)
. 米家檯燈 Pro (支援：Apple HomeKit、米家)
. 米家 LED 智慧檯燈 (支援：米家)

#### 理想目標：

 **回到家時：** 自動關閉攝影機(為了隱私及防止誤觸看家警報，米家APP有BUG看家警報無法照設定時間開啟關閉)、打開電腦桌的Pro燈(不想摸黑)  
**離家時：** 自動打開攝影機(預設啟用看家)、關閉所有燈具
####  **本篇最終達成：** 

離家、回家時發推播提醒，手機按一下觸發操作(已現有裝置沒辦法達到理想的自動化目標)
### 智慧家居設定之路：
####  **Apple HomeKit 使用** 

 **\*僅限米家檯燈 Pro！米家檯燈 Pro！米家檯燈 Pro！** 

這是最簡單的一部分，因為都是原生功能。
![只需四步驟](images/c3150cdc85dd/1*pv62RZ_TjL8X6t-gXnwWtQ.jpeg "只需四步驟")
. 找到家庭APP（如沒有請到App Store搜尋「家庭」安裝）
. 打開家庭APP
. 點擊右上角「+」加入配近
. 掃描Pro檯燈底部HomeKit QRCode加入配件即可！

![](images/c3150cdc85dd/1*0Rm1Ij86bD-fld-N-N1qJw.jpeg "")

加入配件成功後，在配件上重壓(3D TOUCH)/長壓，即可調整亮度、顏色。
#### 那不 **支援Apple HomeKit的智慧家居怎麼使用第三方串接HomeKit？**


除了以上本身就支援的智慧裝置，那不支援Apple HomeKit的裝置是不是就完全無法透過家庭控制呢了？
本章節手把手教你將不支援的裝置(攝影機、一般版檯燈)也加入到「家庭」中！
> **Mac ONLY，WIN使用者請直接跳到使用米家的章節  
> 我的裝置是MacOS 10.14/iOS 12**  

 **使用** [**HomeBridge**](https://github.com/nfarina/homebridge) **：** 

HomeBridge透過使用Mac電腦作為橋接器，將不支援的裝置模擬成HomeKit設備，就可以加入到「家庭」的配件之中．
![運作比較](images/c3150cdc85dd/1*q2ctcxaaxLFExKXd-9NjPg.png "運作比較")

可以看到一個重點就是 **你要有一台Mac電腦保持開機狀態，才能保持橋接通道順暢** ；一但電腦關機、休眠，就無法控制那些HomeKit裝置。


當然網路上也有神人做法，自行買一塊樹莓派來玩，將樹莓派當成橋接器；但這涉及到太多技術，本篇不會介紹。

知道缺點後如果還想玩玩，可以繼續往下看或是跳到下一個直接使用米家的章節。

 **第一步：** 

安裝 [node.js](https://nodejs.org/en/)：[點我](https://nodejs.org/dist/v12.6.0/node-v12.6.0.pkg)[下載](https://nodejs.org/dist/v12.6.0/node-v12.6.0.pkg)，安裝即可


 **第二步：** 

打開「終端機」輸入
```
sudo npm -v
```
![](images/c3150cdc85dd/1*RBRWT93L_abbhzTItL9Mhg.png "")

查看node.js npm套件管理工具是否安裝成功：顯示出版本號即表示成功！

 **第三步：** 

透過 npm 安裝 HomeBridge套件：
```
sudo npm -g install homebridge --unsafe-perm
```

等待安裝完成後…HomeBridge工具就算裝完了！

前面有提到 “HomeBridge就是透過使用Mac電腦作為橋接器，將不支援的裝置模擬成HomeKit設備”， **實際上HomeBridge只是一個平台，各裝置要加入要再另外找HomeBridge的外掛資源** 。


很好找，只要google或在github 搜尋「mija 產品英文名 homebridge」就會有許多資源；這邊介紹兩個我在用的裝置的資源：

 **1.米家攝影機雲臺版資源：** [**MijiaCamera**](https://github.com/josepramon/homebridge-mijia-camera)

攝影機是比較棘手的裝置，花了些時間研究並整理了一下；希望有幫助到有需要的人！

首先ㄧ樣用「終端機」下命令安裝這MijiaCamera這個npm套件
```
sudo npm install -g homebridge-mijia-camera
```

安裝完成後，我們需要取得攝影機的網路 **IP位址** 跟 **Token** 兩個資訊

![](images/c3150cdc85dd/1*n0TIhqyCoKZo7--ePZwuLA.jpeg "")

打開米家APP → 攝影機 → 右上角「…」→設定→網路訊息，得到 **IP位址** ！


 **Token** 資訊就比較麻煩了，需要你將手機連接到Mac上：
![打開 Itunes 介面](images/c3150cdc85dd/1*0ewSMEH7K2rzUlUtSB61vw.png "打開 Itunes 介面")

選備份 **不要勾替本機備份加密** ，點「立即備份」


備份完成後，[下載](http://www.imactools.com/iphonebackupviewer/download/mac)安裝備份查看軟體：[iBackupViewer](http://www.imactools.com/iphonebackupviewer/download/mac)


打開「iBackupViewer」，初次啟動會要你去 Mac「系統偏好設定」- 「安全性與隱私權」-「隱私權」-「+」- 加入「iBackupViewer」  
**_\*如有隱私顧慮可關閉網路使用這套軟體、並在使用後移除_**

![](images/c3150cdc85dd/1*kEOxJCkOxDRuFfoxumssgA.png "")

再次打開「iBackupViewer」成功讀取到備份檔後，點擊右上角切換到「Tree View」模式
![](images/c3150cdc85dd/1*R4l6tRzDaqtiN7xutPKtQg.png "")

左側會顯示你所有安裝的APP，找到米家的APP「AppDomain-com.xiaomi.mihome」->「Documents」

在右側文件列表中找到並選擇 「 **數字\_mihome.sqlite」** 這個檔案


點擊右上角「Export」匯出 ->「Selected」

將剛剛匯出的sqlite檔案丟到 [https://inloop.github.io/sqlite-viewer/](https://inloop.github.io/sqlite-viewer/) 查看內容

![](images/c3150cdc85dd/1*oRK8tHqom2tnR3CE5xz_-w.png "")

可以看到所有米家APP上的裝置資訊欄位，向右滾動到尾端，找到 **ZTOKEN** 欄位，雙擊編輯全選複製


最後再打開 [http://aes.online-domain-tools.com/](http://aes.online-domain-tools.com/) 網站將 **ZTOKEN** 轉成最終 **Token**

![](images/c3150cdc85dd/1*ZXTe6MEFXjYhqtf9uAJWwQ.png "")

1.將剛剛複製出來的 ZTOKEN貼在「Input Text」，選「Hex」  
2.Key輸入「00000000000000000000000000000000」32個0，ㄧ樣選「Hex」  
3.然後按下「Decrypt!」轉換  
4.全選複製右下角藍匡＆去掉空格後就是我們要的結果 **Token**

> Token 這邊有嘗試用「miio」直接嗅探的方式，但好像是米家攝影機韌體有更新過，已無法用這個方法快速方便得到Token了！

 **回到HomeBridge！編輯設定檔 config.json** 
![](images/c3150cdc85dd/1*Zh_BWLwMUg5pOxFEVipgiQ.png "")

使用「Finder」->「前往」->「前往檔案夾」-> 輸入「~/.homebridge」前往

使用文字編輯器打開「config.json」，若沒有此檔案請自行建立一個或[點此下載](https://drive.google.com/file/d/1S67NZwXrVqOpps_Cl9l0494foDaHxuWF/view?usp=sharing) 直接放進去

```JSON
{
   "bridge":{
      "name":"Homebridge",
      "username":"CC:22:3D:E3:CE:30",
      "port":51826,
      "pin":"123-45-568"
   },
   "accessories":[
      {
         "accessory":"MijiaCamera",
         "name":"Mi Camera",
         "ip":"",
         "token":""
      }
   ]
}
```

在config.json裡加入以上內容，IP 及 Token部分帶入上面取得的資訊。

這時候再次回到「終端機」下以下命令啟動 HomeBridge
```
sudo homebridge start
```

如果已啟動之後又更改了config.json內容的話可以改下：
```
sudo homebridge restart
```

重新啟動
![](images/c3150cdc85dd/1*vwCS3QHu285oCrChau9mpw.png "")

這時會出現HomeKit QRCode 讓您掃描加入配件（步驟如上面提到的，Apple HomeKit裝置加入方式）
![](images/c3150cdc85dd/1*CB76x9ryWBve2bssFd0nzA.jpeg "")

下方也會有狀態訊息：
[2019–7–4 23:45:03] [Mi Camera] connecting to camera at 192.168.0.100…
[2019–7–4 23:45:03] [Mi Camera] current power state: off

有出現這些＆沒出現錯誤error訊息即表示設定成功！

一般常見的錯誤都是Token有錯，確認一下上面流程有無遺漏即可。

現在你就可以從「家庭」APP中開關米家智慧攝影機囉！

 **2.米家 LED 智慧檯燈 HomeBridge 資源：** [**homebridge-yeelight-wifi**](https://github.com/vieira/homebridge-yeelight-wifi)

再來是米家 LED 智慧檯燈，由於不像Pro版有支援Apple HomeKit，所以我們還是要用HomeBridge的方法來加入；雖然步驟 **不需經過繁瑣流程取得IP、Token** ，相對攝影機來說較簡單，但檯燈有檯燈的坑，要用另一個YeeLight APP配對後將區域網路控制設定打開：

![](images/c3150cdc85dd/1*uuaLjWduzC5RrOf-gd2-Jw.jpeg "")

這點不得不吐槽一下這個糟糕的整合性，原生米家APP是無法做這項設定的；所以請到APP Store搜尋「[Yeelight](https://apps.apple.com/tw/app/yeelight/id977125608)」APP 下載＆安裝


開啟APP -> 直接使用米家帳號登入 -> 增加裝置 -> 米家檯燈 -> 照指示將檯燈改綁定到 Yeelight APP
![](images/c3150cdc85dd/1*GTcap563FDdC0TsH09hZww.jpeg "")

裝置綁定完成後回到「裝置」頁 -> 點「米家檯燈」進入 -> 點右下角「△」Tab -> 點「局域網控制」進入設定 -> 打開按鈕允許局域網(區域網路)控制

 **檯燈的設置到這裡即可，你可以保留這個APP控制檯燈或再重新綁定回米家．** 

 **再來是HomeBridge設定；ㄧ樣先打開「終端機」下命令安裝** [**homebridge-yeelight-wifi**](https://github.com/vieira/homebridge-yeelight-wifi) **npm套件** 
```
sudo npm install -g homebridge-yeelight-wifi
```

安裝完成後同上攝影機的步驟，前往 ~/.homebridge 資料夾，建立或編輯修改 config.json，這次只需要在最後一個}裡面加上
```
"platforms": [
   {
         "platform" : "yeelight",
         "name" : "yeelight"
   }
 ]
```

即可！

 **最後結合上述攝影機的 config.json檔如下：** 
```JSON

{
	"bridge": {
		"name": "Homebridge",
		"username": "CC:22:3D:E3:CE:30",
		"port": 51826,
		"pin": "123-45-568"
	},

	"accessories": [
		{
			"accessory": "MijiaCamera",
			"name": "Mi Camera",
			"ip": "",
			"token": ""
		}
	],

	"platforms": [
	  {
	        "platform" : "yeelight",
	        "name" : "yeelight"
	  }
	]
}
```

然後一樣回到「終端機」下：
```
sudo homebridge start
```

或
```
sudo homebridge restart
```

即可看到原本不支援的米家 LED 智慧檯燈也加入HomeKit「家庭」APP囉！
![](images/c3150cdc85dd/1*3jm0Kd4545DcmzNtPY-dXA.jpeg "")

而且同樣支援顏色、光度調整！
#### HomeKit配件都加好了，怎麼讓他智慧呢？

全都加好、橋接好後ㄧ樣打開「家庭」APP
![](images/c3150cdc85dd/1*s33BtesqfNSUNyyR069m_Q.jpeg "")

依照步驟新增場景情境，這裡以回家為例：

右上角點擊「+」-> 加入情境 -> 自訂 -> 配件名稱自行輸入(EX:回家) -> 點下方「加入配件」-> 選擇已串接好的HomeKit配件 -> 設定這個場景時的配件狀態(攝影機：關/臺燈：開) -> 可點「測試情境」進行測試 -> 右上角「完成」！

這樣就設定好場囉～這時候在首頁點場景就換執行裡面所有配件的設定！
![](images/c3150cdc85dd/1*VSArlFmoFERbjH13Cns5TQ.jpeg "")

還有一個快捷小撇步，就是在上拉控制選單直接點房子形狀的按鈕快速操作HomeKit/執行情境(右上可切換模式)！
#### 智慧有了，那怎麼自動化呢？

智慧已經有了，現在我想要達成終極目標，回家自動關閉攝影機、開燈；離家自動開攝影機、關燈．
![](images/c3150cdc85dd/1*tCpQ3io2Q2DDCVFxJpBm_g.png "")

切到第三個Tab「自動化」就可設定，很抱歉這邊沒有一個上述設備(iPad/Apple TV/HomePod)可以做 **”** [**家庭中樞**](https://support.apple.com/zh-tw/HT207057) **”** 所以這塊我就沒研究了。


原理好像是回到家，感應到 **”家庭中樞”** 手機/手錶即可精準觸發！

#### 這邊我有找到一個tricky的做法：(感應GPS)

使用第三方的APP串接「家庭」加入自動化設定，就可以透過使用手機GPS定位來做到自動化破解封鎖使用「自動化」Tab的功能

p.s GPS會有約100公尺的誤差
![](images/c3150cdc85dd/1*Rm101LKv29Avb5wv4isg4A.jpeg "")

這邊我使用的第三方串接APP是：[myHome Plus](https://apps.apple.com/us/app/myhome-plus-control-for-nest-wemo-and-homekit/id1050479330)


下載＆安裝後開啟APP -> 允許存取「家庭資料」-> 會看到「家庭」的資料配置 -> 點選右上角「設定按鈕」-> 點「我家」進入 
->下拉到「Triggers」區域 -> 點「Add Trigger」
![](images/c3150cdc85dd/1*Kk6AMnhSYP4sM8JD_66Iow.jpeg "")

Trigger 類型選「Location」-> Name 輸入名字(EX:回家) -> 點「Set Location」設定位置區域 -> 再來 REGION STATUS 可以設定是進入還是離開該區域 -> 最後 SCENES 可以選擇對應要執行的「情景」(上面建立的)

按右上角「完成」儲存後，再回到「家庭」APP，可以看到「自動化」Tab 被打開可以用了！
![](images/c3150cdc85dd/1*SXYVBHk9-pMD8YufRQA4zw.png "")

這時候就可以選擇右上角「＋」使用「家庭」APP直接新增自動化腳本！！
![](images/c3150cdc85dd/1*qbtjNCj9mOvjuX7an6rhXw.jpeg "")

步驟也如第三方APP，不過整合性更佳！使用原生「家庭」APP建立好自動化後也可以滑動刪除剛剛用第三方APP建的。
>  **_！！僅需注意，至少要保留一項；否則Tab就會回到原始封鎖狀態！！_** 

 **Siri 語音控制的部分：** 

相較下面介紹的米家，HomeKit的整合性相當高，可直接使用語音控制設定的配件、執行場景，無需額外，無需額外設定。
![](images/c3150cdc85dd/1*q_ui00ruJl1Fd3_5M-0EhQ.png "")

HomeKit的設定介紹就到這邊了，再來講解米家原生智慧家庭的用法。
####  **使用米家本身搭建智慧家庭的方法：** 

這邊遇到一個困惑點，就是我在米家新增設備中找不到長得一樣的米家檯燈，答案就是：
![看字就好，這個就是](images/c3150cdc85dd/1*xLM5-khndWjvEDdTaFiPfw.png "看字就好，這個就是")

其他設備：攝影機、Pro檯燈就直接照官方說明設定加入就好，這邊不在冗述．

 **場景情境設定：** 
![](images/c3150cdc85dd/1*leO3Z492pJPh3hEASYr-ww.jpeg "")

同「家庭設定方式」-> 切換到「智慧」Tab -> 選擇「手動執行」-> 下方選擇裝置操作(由於是原生所以可選更多功能) -> 繼續增加其他裝置(檯燈) -> 「儲存」完成！
> _一定會有人想問為什麼不直接選「離開或到達某地」？，因為這功能根本沒用，他APP沒針對台灣優化GPS是錯的，而且他的定位只能定在地標上，如果你的位置有那可以直接使用此功能，_ **_文章後續也都可跳過！_** 
>  **_冷知識：_** [**_Google Maps 裡的中國地圖全是錯的！_**](https://buzzorange.com/techorange/2019/05/09/china-map-is-wrong/)
![](images/c3150cdc85dd/1*ZjdH5A0QnLq2LNh9lWvCCw.jpeg "")

快捷開關部分，可以從「我的」->「小元件」設定小工具元件！

這樣就能從通知中心快速執行場境、裝置囉！
![](images/c3150cdc85dd/1*DMmicpzKUIr2xtN8JtP3wQ.png "")

也可從[Apple Watch](apple-watch-series-4-%E5%BE%9E%E5%85%A5%E6%89%8B%E5%88%B0%E4%B8%8A%E6%89%8B%E5%85%A8%E6%96%B9%E4%BD%8D%E5%BF%83%E5%BE%97-a2920e33e73e)上控制元件！  
_\*如果手錶APP一直出現空白請刪除重裝手錶或手機APP，這個APP真的蠻多BUG的_

#### 智慧有了，那怎麼自動化呢？

這邊ㄧ樣要使用GPS感應方式， **如果上述新增場景用的就是「離開或到達某地」，以下介紹設定都可略過囉！**


*****
#### [2019/09/26] 更新 iOS ≥ 13 只使用內建 捷徑 APP 達成自動化 :

[iOS ≥ 13.1 使用「捷徑」自動化功能搭配米家智慧家居，點擊前往查看\>\>](ios-13-1-%E4%BD%BF%E7%94%A8-%E6%8D%B7%E5%BE%91-%E8%87%AA%E5%8B%95%E5%8C%96%E5%8A%9F%E8%83%BD%E6%90%AD%E9%85%8D%E7%B1%B3%E5%AE%B6%E6%99%BA%E6%85%A7%E5%AE%B6%E5%B1%85-21119db777dd)

*****
> _iOS ≥ 12，iOS \< 13 Only :_  
> **使用內建的捷徑APP搭配IFTTT** 
![](images/c3150cdc85dd/1*e9ld6Qn7D64CG-DZA1vAsA.jpeg "")

首先到「我的」-> 「實驗室功能」->「iOS 捷徑」-> 「將米家場景加入捷勁」

打開系統內建的「[捷徑](https://apps.apple.com/tw/app/%E6%8D%B7%E5%BE%91/id915249334)」APP（若找不到請到App Stroe 搜尋下載回來）

![](images/c3150cdc85dd/1*-rjtmZ6PHzSzOoBvjJ-FJQ.jpeg "")

點擊右上角「+」建立捷徑 -> 點右上完成下方的設定按鈕 -> 名稱 -> 輸入名稱（建議用英文，因為等等還要用到）
![](images/c3150cdc85dd/1*5aUsslYvZvlFiSQYJrGgRw.jpeg "")

回到新增捷徑頁面 -> 在下方選單輸入搜尋「米家」-> 加入對應的在米家設定的場景，關閉「執行時顯示」否則執行完會開啟米家APP。
> *如果找不到米家請回到米家APP嘗試開關「我的」-> 「實驗室功能」->「iOS 捷徑」-> 「將米家場景加入捷勁」、滑掉「捷徑」APP重開。

這時候又要使用第三方APP了，我們使用IFTTT做GPS進入、離開的背景觸發器，到App Store搜尋「[IFTTT](https://apps.apple.com/us/app/ifttt/id660944635)」下載＆安裝。

![](images/c3150cdc85dd/1*5tXhFP4uT1ySSFAZnnDQGw.jpeg "")

打開IFTTT、登入帳號後，切換到「My Applets」Tab，點右上角「+」新增->
點擊「+this」-> 搜尋「Location」-> 選擇是進入還是離開
![](images/c3150cdc85dd/1*2vs32eIxtEmvqzxOsDLGEw.jpeg "")

設定位置 -> 點擊「Create trigger」確定 -> 換點下面「+that」-> 搜尋「notification」
![](images/c3150cdc85dd/1*bVmWLH5tUcko5eeOmnR3kQ.jpeg "")

選擇「Send a rich notification from the IFTTT app」：

Title = 通知標題 , Message = 通知內容

Link URL 請輸入：shortcuts://run-shortcut?name= **_捷徑名稱_**


所以才說捷徑名稱盡量設英文比較好

-> 點選「Create action」-> 可點選「Edit title」設定名稱

-> 「Finish」儲存完成！

**當你下次離開/進入設定的區域範圍就會收到觸發的通知(一樣有約100公尺的誤差範圍)，點選通知後就會自動執行米家場景囉！**
![點選通知就會在背景自動執行場景](images/c3150cdc85dd/1*a9zXd_JSpz9IKInJlPoJ1w.png "點選通知就會在背景自動執行場景")

 **Siri 語音控制的部分：** 

由於米家不是Apple內建APP，所以要支援Siri語音控制就得另外設置：
![](images/c3150cdc85dd/1*lyzEU2cKxafbnXkWnR7ltg.jpeg "")

在「智慧」Tab -> 「加入Siri」-> 選擇「目標場景」按「加入Siri」

-> 點紅色錄製指令(EX:關燈) -> 完成！

即可在Siri中直接呼叫控制執行場景！
### 總結

上述一大堆的設定步驟，總結一下就是：

如果要好的體驗就是得花大錢買有HomeKit標誌的電器（就可不需放台Mac做HomeBridge開機待命，直接與原生Apple 家庭功能完美結合）還有要再買HomePod或Apple TV、iPad做家庭中樞；不管是HomeKit標誌的電器、家庭中樞都不便宜！

如果有技術能力可考慮使用第三方智慧裝置（如米家）搭配樹莓派做HomeBridge。

如果像我一個就是個普通人那還是直接用米家最為方便上手，目前的使用習慣是回家、離開家會從通知中心點快捷小工具執行場景操作；捷徑APP搭配IFTTT的部份僅作為通知提醒，怕有時候忘記。

目前體驗雖沒達到目標理想，但已經離 **“智慧家庭”** 更進一步了！

### 進階篇

[**示範使用樹莓派當 HomeBridge 主機，將所有米家家電串上 HomeKit**](%E6%89%93%E9%80%A0%E8%88%92%E9%81%A9%E7%9A%84-wfh-%E6%99%BA%E6%85%A7%E5%B1%85%E5%AE%B6%E7%92%B0%E5%A2%83-%E6%8E%A7%E5%88%B6%E5%AE%B6%E9%9B%BB%E7%9B%A1%E5%9C%A8%E6%8C%87%E5%B0%96-99db2a1fbfe5)
### 延伸閱讀
. [小米智慧家居新添購（AI音箱、溫濕度感應器、體重計2、直流變頻電風扇）](%E5%B0%8F%E7%B1%B3%E6%99%BA%E6%85%A7%E5%AE%B6%E5%B1%85%E6%96%B0%E6%B7%BB%E8%B3%BC-bcff7c157941)
. [iOS ≥ 13.1 使用「捷徑」自動化功能搭配米家智慧家居（直接使用 iOS ≥ 13.1 內建的捷徑APP完成自動化操作）](ios-13-1-%E4%BD%BF%E7%94%A8-%E6%8D%B7%E5%BE%91-%E8%87%AA%E5%8B%95%E5%8C%96%E5%8A%9F%E8%83%BD%E6%90%AD%E9%85%8D%E7%B1%B3%E5%AE%B6%E6%99%BA%E6%85%A7%E5%AE%B6%E5%B1%85-21119db777dd)
. [米家 APP / 小愛音箱地區問題](%E7%B1%B3%E5%AE%B6-app-%E5%B0%8F%E6%84%9B%E9%9F%B3%E7%AE%B1%E5%9C%B0%E5%8D%80%E5%95%8F%E9%A1%8C-94a4020edb82)

[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-life/%E6%99%BA%E6%85%A7%E5%AE%B6%E5%B1%85%E5%88%9D%E9%AB%94%E9%A9%97-apple-homekit-%E5%B0%8F%E7%B1%B3%E7%B1%B3%E5%AE%B6-c3150cdc85dd) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
