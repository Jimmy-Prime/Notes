# TestFlight App 一直 crash

tags: iOS, TestFlight, Xcode

最近遇到神奇的 crash，QC 回報說登入按鈕一點就 crash。

經過漫長的測試和等待 TestFlight 處理之後，得到的結論是把 bitcode 關掉。

在蘋果的 [What is App Thinning?](https://help.apple.com/xcode/mac/current/#/devbbdc5ce4f) 裡面有關 bitcode 的部分。

> “Bitcode is an intermediate representation of a compiled program. Apps you upload to App Store Connect that contain bitcode will be compiled and linked on the App Store. Including bitcode will allow Apple to re-optimize your app binary in the future without the need to submit a new version of your app to the App Store.” 

Apple 會在後台改你的 app，可能剛好遇到 Apple 演算法調整的更為激進，剛好就弄壞了。

這次找問題的過程中作得很混亂，覺得有一些可以改進的地方：

**測試的 build 應該另外開 branch。**不要讓測試的暫時修改污染 master，尤其是 xcproj 裡面的 config 開開關關製造很多 commit 業障。

**從之前正常的 commit 重 build。**先確認方法是正確的，再來做夾板測試，避免浪費時間作多餘的測試。

**確認環境沒有問題。**這次遇到的問題是 Apple 出問題，平常根本不會去想他可能會壞掉，所以最後才懷疑到他身上。使用其他服務也是一樣，可以先驗證服務還是正確運作，而且驗證不需要花很多時間，又可以免掉後面的其他測試。

---

後來又零星有遇到奇怪的 crash，進一步查資料後發現可能不是 bitcode 的問題，應該是 [SR-11564](https://bugs.swift.org/browse/SR-11564) 這個 bug。

Xcode compiler 會太激進了，他把他看起來沒用到的 code 都刪掉了，runtime 就會找不到 symbol 而 crash。

目前的 workaround 方法是到 project settings 裡面把 `DEAD_CODE_STRIPPING ` 設定成 `NO`。
