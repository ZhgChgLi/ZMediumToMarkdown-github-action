---
title: iOS UIViewController 轉場二三事
author: ZhgChgLi
date: 2020-01-11T18:41:06.640Z
tags: [ios,ios-app-development,swift,uiviewcontroller,mobile-app-development]
---

### iOS UIViewController 轉場二三事

UIViewController 下拉關閉/上拉出現/全頁右滑返回 效果全解
### 前言
![](images/14cee137c565/1*6IQTrlT4vIKR-NjLRsvZ-A.gif "")

一直以來都很好奇諸如 Facebook、Line、Spotify…等等常用的 APP 是如何實作「Present 的 UIViewController 可下拉關閉」、「上拉漸入 UIViewController」、「全頁面支援手勢右滑返回」這些效果的。

因為這些效果內建都沒有，下拉關閉也直到 iOS ≥ 13 才有系統的卡片樣式支援。
#### 探索之路

不知道是不會下關鍵字還是資料本身難找，一直找不到這類功能的實踐做法，找到的資料都很含糊零散，只能東拼西湊。

一開始自己研究做法時找到 `UIPresentationController` 這個 API ，沒再深掘其他資料，就用這個方法搭配 `UIPanGestureRecognizer` 用很土炮的方式完成下拉關閉的效果；一直都覺得哪裡怪怪的，感覺會有更好的方式。


直到最近接觸新專案拜讀[大大的文章](https://imnotyourson.com/draggable-view-controller-interactive-view-controller/)，擴大眼界才發現有其他 API 更漂亮、更有彈性的做法可以用。

> _本篇一方面是自我紀錄，另一方面希望有幫助到跟我有一樣困惑的朋友。  
> 內容有點多，嫌麻煩的可以直接拉到底看範例，或直接下載 Github 專案回來研究！_  
### iOS 13 卡片樣式呈現頁面

首先講最新系統內建的效果  
iOS ≥ 13 後 `UIViewController.present(_:animated:completion:)`默認的 `modalPresentationStyle` 效果就是 `UIModalPresentationAutomatic` 片樣式呈現頁面，若想要保持之前的全頁面呈現就要特別指定回 `UIModalPresentationFullScreen` 即可。

![內建行事曆新增效果](images/14cee137c565/1*j0NeJfAuR2fXP56KWglS7Q.gif "內建行事曆新增效果")
#### 如何取消下拉關閉？關閉確認？

更好的使用者體驗應該要能在觸發下拉關閉時檢查有無輸入資料，有的話需要提示使用者是否捨棄動作離開。

這部分蘋果也幫我們想好了，只需實作 `UIAdaptivePresentationControllerDelegate` 裡的方法即可。

```Swift
import UIKit

class DetailViewController: UIViewController {
    private var onEdit:Bool = true;
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //設置代理
        self.presentationController?.delegate = self
        //if uiviewcontroller embed in navigationController:
        //self.navigationController?.presentationController?.delegate = self
        
        //取消下拉關閉方式(1):
        self.isModalInPresentation = true;
        
    }
    
}

//代理實作
extension DetailViewController: UIAdaptivePresentationControllerDelegate {
    //取消下拉關閉方式(2):
    func presentationControllerShouldDismiss(_ presentationController: UIPresentationController) -> Bool {
        return false;
    }
    
    //下拉關閉取消時，下拉手勢觸發
    func presentationControllerDidAttemptToDismiss(_ presentationController: UIPresentationController) {
        if (onEdit) {
          let alert = UIAlertController(title: "資料尚未存儲", message: nil, preferredStyle: .actionSheet)
          alert.addAction(UIAlertAction(title: "捨棄離開", style: .default) { _ in
              self.dismiss(animated: true)
          })
          alert.addAction(UIAlertAction(title: "繼續編輯", style: .cancel, handler: nil))
          self.present(alert, animated: true)      
        } else {
          self.dismiss(animated: true, completion: nil)
        }
    }
}
```

取消下拉關閉可指定 `UIViewController` 的變數 `isModalInPresentation` 為 false 或實作 `UIAdaptivePresentationControllerDelegate` `presentationControllerShouldDismiss` 並回傳 `true` 擇一都可。


`UIAdaptivePresentationControllerDelegate presentationControllerDidAttemptToDismiss` 這個方法只有在 **下拉關閉取消時** 才會呼叫使用。
#### By the way…

卡片樣式呈現頁面對系統來說就是 `Sheet`，行為上跟 `FullScreen` 有所不同。

> _假設今天_ `RootViewController` _是_ `HomeViewController`_在卡片樣式呈現下 (UIModalPresentationAutomatic) 則：_
> `HomeViewController` `Present` `DetailViewController` _時…_  
> `HomeViewController` **_的_** `viewWillDisAppear` **_/_** `viewDidDisAppear` **_都不會觸發。_** 
> _當_ `DetailViewController` `Dismiss` _時…_  
> `HomeViewController` **_的_** `viewWillAppear` **_/_** `viewDidAppear` **_都不會觸發。_** 
> _⚠️_**_因 XCODE 11 之後版本打包的 iOS ≥ 13 APP 預設 Present 都會使用卡片樣式 (UIModalPresentationAutomatic)  
> 如果之前有把一些邏輯放在 viewWillAppear/viewWillDisappear/viewDidAppear/viewDidDisappear 的要多加檢查注意！_** _⚠️_
> 看完系統內建的，來看本篇重頭戲吧！如何自幹這些效果？
### 哪裡可做轉場動畫？

首先先整理哪裡可以做視窗切換轉場動畫。
![UITabBarController/UIViewController/UINavigationController](images/14cee137c565/1*G0us0AtYJCy3va1sh_bWhQ.gif "UITabBarController/UIViewController/UINavigationController")
#### UITabBarController 切換時

我們可以在 `UITabBarController` 設定 `delegate` 然後實作 `animationControllerForTransitionFrom` 方法，就能在切換 `UITabBarController` 時對內容套用自訂轉場特效。


系統預設無動畫，上方展示圖的是淡入淡出切換特效。
```Swift
import UIKit

class MainTabBarViewController: UITabBarController {

    override func viewDidLoad() {
        super.viewDidLoad()
        self.delegate = self
        
    }
    
}

extension MainTabBarViewController: UITabBarControllerDelegate {
    func tabBarController(_ tabBarController: UITabBarController, animationControllerForTransitionFrom fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        //return UIViewControllerAnimatedTransitioning
    }
}
```
#### UIViewController Present/Dismiss 時

理所當然，在 `Present/Dismiss` `UIViewController` 時可以指定要套用的動畫效果，不然就不會有此篇文章了XD；不過值得一提的是，如果只是單純要做 Present 動畫沒有要做手勢控制，可以直接使用 `UIPresentationController` 方便快速 (詳見文末參考資料)。


系統預設是上滑出現下滑消失！自己客製的話可以加入淡入、圓角、出現位置控制…等效果。
```Swift
import UIKit

class HomeAddViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        self.modalPresentationStyle = .custom
        self.transitioningDelegate = self
    }
    
}

extension HomeAddViewController: UIViewControllerTransitioningDelegate {
    
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        //回傳 nil 即走預設動畫
        return //UIViewControllerAnimatedTransitioning Present時要套用的動畫
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        //回傳 nil 即走預設動畫
        return //UIViewControllerAnimatedTransitioning Dismiss時要套用的動畫
    }
}
```
> _任何_ `UIViewController` _都能實作_ `transitioningDelegate` _告知_ `Present/Dismiss` _動畫；_`UITabBarViewController`_、_`UINavigationController`_、_`UITableViewController`_….都可_
#### UINavigationController Push/Pop 時

`UINavigationController` 大概是最不太需要會改動畫的，因為系統預設的左滑出現右滑返回動畫已經是最好的效果，能想得到要做這部分的客製可能可以用來做無縫 `UIViewController` 左右切換效果。

因為我們要做全頁都可手勢返回，需要配合自訂 POP 動畫，所以需要自己實作一個返回動畫效果。
```Swift
import UIKit

class HomeNavigationController: UINavigationController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.delegate = self
    }

}

extension HomeNavigationController: UINavigationControllerDelegate {
    func navigationController(_ navigationController: UINavigationController, animationControllerFor operation: UINavigationController.Operation, from fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        
        if operation == .pop {
            return //UIViewControllerAnimatedTransitioning 返回時要套用的動畫
        } else if operation == .push {
            return //UIViewControllerAnimatedTransitioning push時要套用的動畫
        }
        
        //回傳 nil 即走預設動畫
        return nil
    }
}
```
### 交互非交互動畫？

再講動畫實作、手勢控制前，先講一下何謂交互與非交互。

 **交互動畫：** 手勢觸發動畫，如 UIPanGestureRecognizer

 **非交互動畫：** 系統呼叫動畫，如 self.present()
### 怎麼實作動畫效果？

講完哪裡可以做，再來看怎麼做動畫效果。

我們需要實作 `UIViewControllerAnimatedTransitioning` 這個 `Protocol` 並在裡面對視窗做動畫。

#### 一般轉場動畫: UIView.animate

直接使用 `UIView.animate` 做動畫處理，此時的 `UIViewControllerAnimatedTransitioning` 需要實作 `transitionDuration` 告知動畫時長、`animateTransition` 實作動畫內容這兩個方法。

```Swift
import UIKit

class SlideFromLeftToRightTransition: NSObject, UIViewControllerAnimatedTransitioning {
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.4
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        
        //可用參數：
        //取得要展示的目標 UIViewController 的 View 內容:
        let toView = transitionContext.view(forKey: .to)
        //取得要展示的目標 UIViewController:
        let toViewController = transitionContext.viewController(forKey: .to)
        //取得要展示的目標 UIViewController 的 View 的初始化 Frame 資訊:
        let toInitalFrame = transitionContext.initialFrame(for: toViewController!)
        //取得要展示的目標 UIViewController 的 View 的最終 Frame 資訊:
        let toFinalFrame = transitionContext.finalFrame(for: toViewController!)
        
        //取得當前 UIViewController 的 View 內容:
        let fromView = transitionContext.view(forKey: .from)
        //取得當前 UIViewController:
        let fromViewController = transitionContext.viewController(forKey: .from)
        //取得當前 UIViewController 的 View 的初始化 Frame 資訊:
        let fromInitalFrame = transitionContext.initialFrame(for: fromViewController!)
        //取得當前 UIViewController 的 View 的最終 Frame 資訊: (在關閉動畫時可以取得之前顯示動畫時的最終Frame)
        let fromFinalFrame = transitionContext.finalFrame(for: fromViewController!)
        
        //toView.frame.origin.y = UIScreen.main.bounds.size.height
        
        UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: [.curveLinear], animations: {
            //toView.frame.origin.y = 0
        }) { (_) in
            if (!transitionContext.transitionWasCancelled) {
                //動畫沒中斷
            }
            
            // 告知系統動畫完成
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
        
    }
    
}

```
>  **_To 跟 From:_**  
> _假設今天_ `HomeViewController` _要_ `Present/Push` `DetailViewController` _時，  
> From = HomeViewController / To = DetailViewController_ `DetailViewController` _要_ `Dismiss/Pop` _時，  
> From = DetailViewController / To = HomeViewController_  

⚠️⚠️⚠️⚠️⚠️
> _官方建議從_ `transitionContext.view` _拿 View 使用，而不是從_ `transitionContext.viewController` _拿 .view 使用。  
> 但這邊有個問題，就是在做 Present/Dismiss 動畫時當_ `modalPresentationStyle = .custom` _；  
> Present 時使用_ `transitionContext.view(forKey: .from)` _會是_ **_nil_** _、  
> Dismiss 時使用_ `transitionContext.view(forKey: .to)` _也會是_ **_nil_** _；  
> 還是需要從 viewController.view 拿值來用。_  

⚠️⚠️⚠️⚠️⚠️
> `transitionContext.completeTransition(!transitionContext.transitionWasCancelled)` _動畫完成必須呼叫，否則_ **_畫面會卡死_** _；  
> 但因_ `UIView.animate` _若無可執行動畫就不會 Call_ `completion` _造成前述方法未被呼叫；所以務必確保動畫是會執行的 (EX: y從100到0)。_

ℹ️ℹ️ℹ️ℹ️ℹ️
> _參與動畫的_ `ToView/FromView`_，若因 View 較為複雜或動畫時有些問題；可改用_ `snapshotView(afterScreenUpdates:)`_截圖作為動畫展示，先截圖然後_ `transitionContext.containerView.addSubview(snapShotView)` _上去圖層，接著隱藏原本的_ `ToView/FromView (isHidden = true)`_，在動畫結束時在_ `snapShotView.removeFromSuperview()`_和恢復顯示原本的_ `ToView/FromView (isHidden = true)`_。_
#### 可中斷、繼續的轉場動畫: UIViewPropertyAnimator

另外也可以使用 **iOS ≥ 10** 新的動畫類別來實作動畫效果，  
看個人習慣或是動畫要做到多細節來做選擇，  
雖然官方的建議是有交互就使用 `UIViewPropertyAnimator` 但**不管是交互非交互(手勢控制) 一般都使用 UIView.animate 即可**；  
`UIViewPropertyAnimator` 的轉場動畫能做到中斷繼續的效果，雖然我不知道實際能應用在哪，有興趣的朋友可參考[此篇文章](https://juejin.im/post/5c3aa7ff518825551e285b8d)。

```Swift
import UIKit

class FadeInFadeOutTransition: NSObject, UIViewControllerAnimatedTransitioning {
    
    private var animatorForCurrentTransition: UIViewImplicitlyAnimating?

    func interruptibleAnimator(using transitionContext: UIViewControllerContextTransitioning) -> UIViewImplicitlyAnimating {
        
        //當前有轉場動畫時直接返回
        if let animatorForCurrentTransition = animatorForCurrentTransition {
            return animatorForCurrentTransition
        }
        
        //參數同前述
        
        //fromView.frame.origin.y = 100
        
        let animator = UIViewPropertyAnimator(duration: transitionDuration(using: transitionContext), curve: .linear)
        
        animator.addAnimations {
            //fromView.frame.origin.y = 0
        }
        
        animator.addCompletion { (position) in
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
        
        //抓著動畫
        self.animatorForCurrentTransition = animator
        return animator
    }
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.4
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        //如果是非交互會走這，就讓它也走交互的動畫
        let animator = self.interruptibleAnimator(using: transitionContext)
        animator.startAnimation()
    }
    
    func animationEnded(_ transitionCompleted: Bool) {
        //動畫完成，清空
        self.animatorForCurrentTransition = nil
    }
    
}

```
> _交互情況下 (後面講控制會細提)，會使用_ `interruptibleAnimator` _方法的動畫；非交互的情況則還是使用_ `animateTransition` _方法。  
> 因為能繼續、中斷的特性；所以_ `interruptibleAnimator` _是有可能會重複呼叫使用的；所以我們需要用一個全域變數做存取返回。_

 **Murmur…**  
其實我本來是想全都改用新的 `UIViewPropertyAnimator` 也想推薦大家都用新的來做，但我遇到一個很奇怪的問題，就是在做全頁手勢返回 Pop 動畫時，若手勢放開，動畫歸位，上方的 Navigation Bar 的 Item 會淡入淡出閃一下…找不到解，但回去用 `UIView.animate` 就沒這問題；如果有地方沒注意到歡迎跟我說\<( \_ \_ )\>。
![問題圖; + 按鈕是上一頁的](images/14cee137c565/1*cVg7iZ_rFC2nxm2H5ET1Gg.gif "問題圖; + 按鈕是上一頁的")

所以保險起見還是用舊的方式吧！

實際會依照不同的動畫效果建立個別的 Class，若覺得很檔案雜，可參考文末包好的方案；或是將同個連貫(Present+Dismii)動畫放在一起。
#### transitionCoordinator

另外如果需要更細緻的控制，例如 ViewController 裡面有某個元件需要配合轉場動畫改變；可在 `UIViewController` 中使用 `transitionCoordinator` 進行協作，這部分我沒用到；有興趣可參考[此篇文章](https://kemchenj.github.io/2018-12-24/)。

### 怎麼控制動畫？

這邊就是前述所說的「交互」，實際就是手勢控制；本篇最重要的章節，因為我們的要做的是手勢操作與轉場動畫的連動功能，才能達成我們要的下拉關閉、全頁返回功能。
#### 控制代理設置:

同前面 `ViewController` 代理動畫設計，交互處理的類也需要在代理中告知 `ViewController`。


**UITabBarController: 無  
UINavigationController (Push/Pop):**  
```Swift
import UIKit

class HomeNavigationController: UINavigationController {

    override func viewDidLoad() {
        super.viewDidLoad()
        
        self.delegate = self
    }

}

extension HomeNavigationController: UINavigationControllerDelegate {
    func navigationController(_ navigationController: UINavigationController, animationControllerFor operation: UINavigationController.Operation, from fromVC: UIViewController, to toVC: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        
        if operation == .pop {
            return //UIViewControllerAnimatedTransitioning 返回時要套用的動畫
        } else if operation == .push {
            return //UIViewControllerAnimatedTransitioning push時要套用的動畫
        }
        //回傳 nil 即走預設動畫
        return nil
    }
    
    //新增交互代理方法:
    func navigationController(_ navigationController: UINavigationController, interactionControllerFor animationController: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        //這邊無法得知是Pop還是Push 只能從要做的動畫本身做判斷
        if animationController is push時套用的動畫 {
            return //UIPercentDrivenInteractiveTransition push動畫的交互控制方法
        } else if animationController is 返回時套用的動畫 {
            return //UIPercentDrivenInteractiveTransition pop動畫的交互控制方法
        }
        //回傳 nil 即不做交互處理
        return nil
    }
}
```

**UIViewController (Present/Dismiss):**
```Swift
import UIKit

class HomeAddViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        self.modalPresentationStyle = .custom
        self.transitioningDelegate = self
    }
    
}

extension HomeAddViewController: UIViewControllerTransitioningDelegate {
    
    func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        //return nil 即不做交互處理
        return //UIPercentDrivenInteractiveTransition Dismiss時交互控制方法
    }
    
    func interactionControllerForPresentation(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        //return nil 即不做交互處理
        return //UIPercentDrivenInteractiveTransition Present時交互控制方法
    }
    
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        //回傳 nil 即走預設動畫
        return //UIViewControllerAnimatedTransitioning Present時要套用的動畫
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        //回傳 nil 即走預設動畫
        return //UIViewControllerAnimatedTransitioning Dismiss時要套用的動畫
    }
    
}
```

⚠️⚠️⚠️⚠️⚠️
> _有實作 interactionControllerFor … 這些方法，就算動畫是非交互(EX: self.present 系統呼叫轉場) 也會 Call 這些方法處理；我們需要控制的是裡面的_ `wantsInteractiveStart` _參數(下面介紹)。_
#### 動畫交互處理類 UIPercentDrivenInteractiveTransition:

再來講核心要實作的 `UIPercentDrivenInteractiveTransition`。

```Swift
import UIKit

class PullToDismissInteractive: UIPercentDrivenInteractiveTransition {
    
    //要加手勢控制交互的UIView
    private var interactiveView: UIView!
    //當前的UIViewController
    private var presented: UIViewController!
    //當托拉超過多少%後就完成執行，否則復原
    private let thredhold: CGFloat = 0.4
    
    //不同轉場效果可能需要不同資訊，可自訂
    convenience init(_ presented: UIViewController, _ interactiveView: UIView) {
        self.init()
        self.interactiveView = interactiveView
        self.presented = presented
        setupPanGesture()
        
        //默認值，告知系統當前非交互動畫
        wantsInteractiveStart = false
    }

    private func setupPanGesture() {
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        panGesture.maximumNumberOfTouches = 1
        panGesture.delegate = self
        interactiveView.addGestureRecognizer(panGesture)
    }

    @objc func handlePan(_ sender: UIPanGestureRecognizer) {
        switch sender.state {
        case .began:
            //reset 手勢位置
            sender.setTranslation(.zero, in: interactiveView)
            //告知系統當前開始的是手勢觸發的交互動畫
            wantsInteractiveStart = true
            
            //在手勢began時呼叫要做的轉場效果(不會直接執行，系統會抓住)
            //然後轉場效果有設對應的動畫就會跳到 UIViewControllerAnimatedTransitioning 處理
            // animated 一定為 true 否則沒動畫
            
            //Dismiss:
            self.presented.dismiss(animated: true, completion: nil)
            //Present:
            //self.present(presenting,animated: true)
            //Push:
            //self.navigationController.push(presenting)
            //Pop:
            //self.navigationController.pop(animated: true)
        
        case .changed:
            //手勢滑動的位置計算 對應動畫完成百分比 0~1
            //實際依動畫類型不同，計算方式不同
            let translation = sender.translation(in: interactiveView)
            guard translation.y >= 0 else {
                sender.setTranslation(.zero, in: interactiveView)
                return
            }
            let percentage = abs(translation.y / interactiveView.bounds.height)
            
            //update UIViewControllerAnimatedTransitioning 動畫百分比
            update(percentage)
        case .ended:
            //手勢放開完成時，看完成度有沒有超過 thredhold
            wantsInteractiveStart = false
            if percentComplete >= thredhold {
              //有，告知動畫完成
              finish()
            } else {
              //無，告知動畫歸位復原
              cancel()
            }
        case .cancelled, .failed:
          //取消、錯誤時
          wantsInteractiveStart = false
          cancel()
        default:
          wantsInteractiveStart = false
          return
        }
    }
}

//當UIViewController內有UIScrollView元件(UITableView/UICollectionView/WKWebView....)，防止手勢衝突
//當裡面的UIScrollView元件已滑到頂部，則啟用交互轉場的手勢操作
extension PullToDismissInteractive: UIGestureRecognizerDelegate {
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        if let scrollView = otherGestureRecognizer.view as? UIScrollView {
            if scrollView.contentOffset.y <= 0 {
                return true
            } else {
                return false
            }
        }
        return true
    }
    
}

```

[_\*關於 sender.setTranslation(.zero, in:interactiveView) 原因的補充點我\<_](https://stackoverflow.com/questions/29558622/pan-gesture-why-need-settranslation-to-zero)

我們需要依據不同的手勢操作效果，實作不同的 Class；若是同個連貫(Present+Dismii)的操作也可包在一起。

⚠️⚠️⚠️⚠️⚠️
> `wantsInteractiveStart` **_務必處於符合的狀態_** _，若在交互動畫時告知_ `wantsInteractiveStart = false` _也會造成卡畫面；  
> 要退出重進 APP 才會恢復正。_  

⚠️⚠️⚠️⚠️⚠️
> _interactiveView 也一定要是_ **_isUserInteractionEnabled = true_** _哦  
> 可以多加設置確保一下！_  
### 組合

當我們把這裡個 `Delegate` 設好、`Class` 建好後就能做到我們想要的功能了。  
再來不囉唆，直接上完成範例。

### 自製下拉關閉頁面效果

自製下拉的好處在能支援市面所有 iOS 版本、可控制蓋板百分比、控制觸發關閉位置、客製化動畫效果。
![點右上方 + Present 頁面](images/14cee137c565/1*Wz8y5UJSgS0IUN86upSqLw.gif "點右上方 + Present 頁面")

這是一個 `HomeViewController` Present `HomeAddViewController` 和`HomeAddViewController`Dismiss的範例。

```Swift
import UIKit

class HomeViewController: UIViewController {

    @IBAction func addButtonTapped(_ sender: Any) {
        guard let homeAddViewController = UIStoryboard(name: "Main", bundle: nil).instantiateViewController(identifier: "HomeAddViewController") as? HomeAddViewController else {
            return
        }
        
        //transitioningDelegate 可指定目標ViewController處理或當前的ViewController處理
        homeAddViewController.transitioningDelegate = homeAddViewController
        homeAddViewController.modalPresentationStyle = .custom
        self.present(homeAddViewController, animated: true, completion: nil)
    }

}
```
```Swift
import UIKit

class HomeAddViewController: UIViewController {

    private var pullToDismissInteractive:PullToDismissInteractive!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        //綁定轉場交互資訊
        self.pullToDismissInteractive = PullToDismissInteractive(self, self.view)
    }
    
}

extension HomeAddViewController: UIViewControllerTransitioningDelegate {
    
    func interactionControllerForDismissal(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        return pullToDismissInteractive
    }
    
    func animationController(forPresented presented: UIViewController, presenting: UIViewController, source: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return PresentAndDismissTransition(false)
    }
    
    func animationController(forDismissed dismissed: UIViewController) -> UIViewControllerAnimatedTransitioning? {
        return PresentAndDismissTransition(true)
    }
    
    func interactionControllerForPresentation(using animator: UIViewControllerAnimatedTransitioning) -> UIViewControllerInteractiveTransitioning? {
        //這邊無Present操作手勢
        return nil
    }
}

```
```Swift
import UIKit

class PullToDismissInteractive: UIPercentDrivenInteractiveTransition {
    
    private var interactiveView: UIView!
    private var presented: UIViewController!
    private var completion:(() -> Void)?
    private let thredhold: CGFloat = 0.4
    
    convenience init(_ presented: UIViewController, _ interactiveView: UIView,_ completion:(() -> Void)? = nil) {
        self.init()
        self.interactiveView = interactiveView
        self.completion = completion
        self.presented = presented
        setupPanGesture()
        
        wantsInteractiveStart = false
    }

    private func setupPanGesture() {
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        panGesture.maximumNumberOfTouches = 1
        panGesture.delegate = self
        interactiveView.addGestureRecognizer(panGesture)
    }

    @objc func handlePan(_ sender: UIPanGestureRecognizer) {
        switch sender.state {
        case .began:
            sender.setTranslation(.zero, in: interactiveView)
            wantsInteractiveStart = true
            
            self.presented.dismiss(animated: true, completion: self.completion)
        case .changed:
            let translation = sender.translation(in: interactiveView)
            guard translation.y >= 0 else {
                sender.setTranslation(.zero, in: interactiveView)
                return
            }

            let percentage = abs(translation.y / interactiveView.bounds.height)
            update(percentage)
        case .ended:
            if percentComplete >= thredhold {
                finish()
            } else {
                wantsInteractiveStart = false
                cancel()
            }
        case .cancelled, .failed:
            wantsInteractiveStart = false
            cancel()
        default:
            wantsInteractiveStart = false
            return
        }
    }
}

extension PullToDismissInteractive: UIGestureRecognizerDelegate {
    func gestureRecognizer(_ gestureRecognizer: UIGestureRecognizer, shouldRecognizeSimultaneouslyWith otherGestureRecognizer: UIGestureRecognizer) -> Bool {
        if let scrollView = otherGestureRecognizer.view as? UIScrollView {
            if scrollView.contentOffset.y <= 0 {
                return true
            } else {
                return false
            }
        }
        return true
    }
    
}
```
```Swift
import UIKit

//蓋在原本View上得半透明遮罩效果View
class DimmingView:UIView {
    override init(frame: CGRect) {
        super.init(frame: frame)
        self.backgroundColor = UIColor.black
        self.alpha = 0
    }
    
    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

class PresentAndDismissTransition: NSObject, UIViewControllerAnimatedTransitioning {
    
    private var isDismiss:Bool!
    
    convenience init(_ isDismiss:Bool) {
        self.init()
        self.isDismiss = isDismiss
    }
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.4
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        
        guard let toViewController = transitionContext.viewController(forKey: .to),let fromViewController = transitionContext.viewController(forKey: .from) else {
            return
        }
        
        if !self.isDismiss {
            //Present
            
            toViewController.view.frame.size.height -= 50
            toViewController.view.frame.origin.y = UIScreen.main.bounds.size.height
            transitionContext.containerView.addSubview(toViewController.view)
            
            let toViewpath = UIBezierPath(roundedRect: toViewController.view.bounds, byRoundingCorners: [.topLeft, .topRight], cornerRadii: CGSize(width: 6, height: 6))
            let toViewmask = CAShapeLayer()
            toViewmask.path = toViewpath.cgPath
            toViewController.view.layer.mask = toViewmask
            
            let fromViewpath = UIBezierPath(roundedRect: fromViewController.view.bounds, byRoundingCorners: [.topLeft, .topRight], cornerRadii: CGSize(width: 6, height: 6))
            let fromViewmask = CAShapeLayer()
            fromViewmask.path = fromViewpath.cgPath
            fromViewController.view.layer.mask = fromViewmask
            
            
            let dimmingView = DimmingView(frame: fromViewController.view.frame)
            transitionContext.containerView.insertSubview(dimmingView, belowSubview: toViewController.view)
            
            fromViewController.view.transform = CGAffineTransform(scaleX: 1.0, y: 1.0)
            
            UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: [.curveEaseOut], animations: {
                dimmingView.alpha = 0.7
                fromViewController.view.transform = CGAffineTransform(scaleX: 0.9, y: 0.9)
                toViewController.view.frame.origin.y = 50
            }) { (_) in
                fromViewController.view.transform = CGAffineTransform(scaleX: 0.9, y: 0.9)
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        } else {
            //Dismiss
            
            let dimmingView = transitionContext.containerView.subviews.first(where: { (view) -> Bool in
                return view is DimmingView
            })
            
            fromViewController.view.frame.origin.y = 50 //or use finalFrame
            
            let fromViewSnpaShot = fromViewController.view.snapshotView(afterScreenUpdates: false)
            
            if let fromViewSnpaShot = fromViewSnpaShot {
                fromViewController.view.isHidden = true
                fromViewSnpaShot.frame = fromViewController.view.frame
                transitionContext.containerView.addSubview(fromViewSnpaShot)
            }
            
            dimmingView?.alpha = 0.7
            toViewController.view.transform = CGAffineTransform(scaleX: 0.9, y: 0.9)
            
            
            UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: [.curveLinear], animations: {
                dimmingView?.alpha = 0
                fromViewSnpaShot?.frame.origin.y = UIScreen.main.bounds.size.height
                toViewController.view.transform = CGAffineTransform(scaleX: 1.0, y: 1.0)
            }) { (_) in
                if (!transitionContext.transitionWasCancelled) {
                    toViewController.view.transform = .identity
                    dimmingView?.removeFromSuperview()
                    toViewController.view.layer.mask = nil
                }
                fromViewSnpaShot?.removeFromSuperview()
                fromViewController.view.isHidden = false
                transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
            }
        }
        
    }
    
}

```

以上就能達到如圖的效果，這邊因教學展示不想弄的路太複雜，所以程式碼很醜，還有很多優化整合的空間。
>  **_值得一提的是…_**  
> _iOS ≥ 13，如果遇到 View 內容有 UITextView，在做下拉關閉動畫時，動畫當中 UITextView 的文字內容會一片空白；造成體驗會閃一下_[_(影片範例)_](https://twitter.com/zhgchgli/status/1207851671553892352)_…  
> 這邊的解決方案是在做動畫時用_ `snapshotView(afterScreenUpdates:)` _截圖取代原本的 View 圖層。_
### 全頁右滑返回

在尋找全畫面都能手勢右滑返回的解決方案時，找到個 **Tricky** 的方法：  
直接在畫面上加一個 `UIPanGestureRecognizer` 然後將 `target`、`action`   
都指定到原生的 `interactivePopGestureRecognizer` ，`action:handleNavigationTransition`。  
[_\*詳細方法點我\<_](https://juejin.im/entry/5795809dd342d30059ed5c60)


沒錯！看起來就很 Private API，感覺審核會被拒；而且不確定 Swift 是否可用，應該有用到 OC 才有的 Runtime 特性。
#### 還是走正規的吧：

ㄧ樣使用本篇的方式，我們在 `navigationController` POP 返回時自行處理；添加一個全頁右滑手勢控制配合自訂右滑動畫，即可！


其他省略，只貼關鍵的動畫跟交互處理類別：
```Swift
import UIKit

class SwipeBackInteractive: UIPercentDrivenInteractiveTransition {
    
    private var interactiveView: UIView!
    private var navigationController: UINavigationController!

    private let thredhold: CGFloat = 0.4
    
    convenience init(_ navigationController: UINavigationController, _ interactiveView: UIView) {
        self.init()
        self.interactiveView = interactiveView
        
        self.navigationController = navigationController
        setupPanGesture()
        
        wantsInteractiveStart = false
    }

    private func setupPanGesture() {
        let panGesture = UIPanGestureRecognizer(target: self, action: #selector(handlePan(_:)))
        panGesture.maximumNumberOfTouches = 1
        interactiveView.addGestureRecognizer(panGesture)
    }

    @objc func handlePan(_ sender: UIPanGestureRecognizer) {
        
        switch sender.state {
        case .began:
            sender.setTranslation(.zero, in: interactiveView)
            wantsInteractiveStart = true
            
            self.navigationController.popViewController(animated: true)
        case .changed:
            let translation = sender.translation(in: interactiveView)
            guard translation.x >= 0 else {
                sender.setTranslation(.zero, in: interactiveView)
                return
            }

            let percentage = abs(translation.x / interactiveView.bounds.width)
            update(percentage)
        case .ended:
            if percentComplete >= thredhold {
                finish()
            } else {
                wantsInteractiveStart = false
                cancel()
            }
        case .cancelled, .failed:
            wantsInteractiveStart = false
            cancel()
        default:
            wantsInteractiveStart = false
            return
        }
    }
}

```
```Swift
import UIKit

class SlideFromLeftToRightTransition: NSObject, UIViewControllerAnimatedTransitioning {
    
    func transitionDuration(using transitionContext: UIViewControllerContextTransitioning?) -> TimeInterval {
        return 0.4
    }
    
    func animateTransition(using transitionContext: UIViewControllerContextTransitioning) {
        
        guard let toView = transitionContext.view(forKey: .to), let fromView = transitionContext.view(forKey: .from) else {
            return
        }
        
        toView.frame.origin.x = -(UIScreen.main.bounds.size.width / 2)
        fromView.frame.origin.x = 0
        transitionContext.containerView.insertSubview(toView, belowSubview: fromView)
        
        let shadowRect: CGRect = CGRect(x: -4, y: -20, width: 4, height: fromView.frame.height)
        let shadowPath: UIBezierPath = UIBezierPath(rect: shadowRect)
        fromView.layer.shadowPath = shadowPath.cgPath
        fromView.layer.shadowOpacity = 0.8

        UIView.animate(withDuration: transitionDuration(using: transitionContext), delay: 0, options: [.curveLinear], animations: {
            toView.frame.origin.x = 0
            fromView.frame.origin.x = UIScreen.main.bounds.size.width
        }) { (_) in
            transitionContext.completeTransition(!transitionContext.transitionWasCancelled)
        }
        
    }
    
}

```
### 上拉漸入 UIViewController

在View上上拉漸入＋下拉關閉，就是在做類似 Spotify 的播放器轉場效果了！

這部分較為繁瑣，但原理一樣，這邊就不 PO 出來了，有興趣的朋友可參考 GitHub 範例內容。

要說哪裡要注意，大概就是 **在上拉漸入時，動畫要確保是使用「.curveLinear 線性」否則會出現上拉不跟手的問題** ；拉的程度跟顯示的位置不是正比。

### 完成！
![完成圖](images/14cee137c565/1*RRAVb3p7mZpUCNOpd64-Pw.gif "完成圖")
> 此篇很長，也花了我許久時間整理製作，感謝您的耐心閱讀。
#### 全篇 GitHub 範例下載：

[zhgchgli0718/UIViewControllerTransitionDemo
You can't perform that action at this time. You signed in with another tab or window. You signed out in another tab or…github.com](https://github.com/zhgchgli0718/UIViewControllerTransitionDemo)

 **參考資料：** 
. [Draggable view controller? Interactive view controller!](https://imnotyourson.com/draggable-view-controller-interactive-view-controller/)
. [系统学习iOS动画之四：视图控制器的转场动画](https://juejin.im/post/5c24745b6fb9a049d5198ce5#18-%E5%AF%BC%E8%88%AA%E6%8E%A7%E5%88%B6%E5%99%A8%E8%BD%AC%E5%9C%BA)
. [系统学习iOS动画之五：使用UIViewPropertyAnimator](https://juejin.im/post/5c3aa7ff518825551e285b8d)
. [用UIPresentationController来写一个简洁漂亮的底部弹出控件](https://juejin.im/post/5a9651d25188257a5911f666) (單純只做Present 動畫效果可直接用這個)


 **若需要參考優雅的程式碼封裝使用：** 
. Swift: [https://github.com/Kharauzov/SwipeableCards](https://github.com/Kharauzov/SwipeableCards)

. Objective-C: [https://github.com/saiday/DraggableViewControllerDemo](https://github.com/saiday/DraggableViewControllerDemo)


[Like Z Realm's work](https://cdn.embedly.com/widgets/media.html?src=https%3A%2F%2Fbutton.like.co%2Fin%2Fembed%2Fzhgchgli%2Fbutton&display_name=LikeCoin&url=https%3A%2F%2Fbutton.like.co%2Fzhgchgli&image=https%3A%2F%2Fstorage.googleapis.com%2Flikecoin-foundation.appspot.com%2Flikecoin_store_user_zhgchgli_main%3FGoogleAccessId%3Dfirebase-adminsdk-eyzut%2540likecoin-foundation.iam.gserviceaccount.com%26Expires%3D2430432000%26Signature%3DgFRSNto%252BjjxXpRoYyuEMD5Ecm7mLK2uVo1vGz4NinmwLnAK0BGjcfKnItFpt%252BcYurx3wiwKTvrxvU019ruiCeNav7s7QUs5lgDDBc7c6zSVRbgcWhnJoKgReRkRu6Gd93WvGf%252BOdm4FPPgvpaJV9UE7h2MySR6%252B%252F4a%252B4kJCspzCTmLgIewm8W99pSbkX%252BQSlZ4t5Pw22SANS%252BlGl1nBCX48fGg%252Btg0vTghBGrAD2%252FMEXpGNJCdTPx8Gd9urOpqtwV4L1I2e2kYSC4YPDBD6pof1O6fKX%252BI8lGLEYiYP1sthjgf8Y4ZbgQr4Kt%252BRYIicx%252Bg6w3YWTg5zgHxAYhOINXw%253D%253D&key=a19fcc184b9711e1b4764040d3dc5c07&type=text%2Fhtml&schema=like)

有任何問題及指教歡迎[與我聯絡](https://www.zhgchg.li/contact)。




+-----------------------------------------------------------------------------------+

| **[View original post on Medium](https://medium.com/zrealm-ios-dev/ios-uiviewcontroller-%E8%BD%89%E5%A0%B4%E4%BA%8C%E4%B8%89%E4%BA%8B-14cee137c565) - Converted by [ZhgChgLi](https://blog.zhgchg.li)/[ZMediumToMarkdown](https://github.com/ZhgChgLi/ZMediumToMarkdown)** |

+-----------------------------------------------------------------------------------+
