# 在沒有Coordinator的狀況下，Tangle如何抵擋雙重支付攻擊(Double-Spend)？

在每項分散式帳本技術(DLT)中，每筆交易會有一定機率驗證有效。在比特幣中，一筆交易是否有效決定於寫入最長的鏈，所以交易無效的機率取決於有更長串的區塊鏈出現而該交易不存在該鏈中。

在區塊鏈技術中，兩筆互斥的交易無法存在於最長的區塊鏈中。

在Tangle中則基本上非常不同，兩筆互斥交易可以存在於Tangle中，這不會造成問題因為最終只有一項會被判定為有效的。

每筆交易被驗證為有效的機率等同於tips驗證直接或間接該交易的比例。
以下圖示來解釋雙重支付如何被判定的，我們先從一條沒有衝突的Tangle開始：

![](https://i.imgur.com/Wc6y6Sm.png)


現在如果有兩筆互斥的交易(X和Y)進入Tangle，如下圖所示：

![](https://i.imgur.com/6JDifVz.png)

當我們加入更多筆交易，衝突便會被偵測到:

![](https://i.imgur.com/fyfRO1n.png)

![](https://i.imgur.com/xVB6gTN.png)

紅色部分最終會被孤立，這樣一來就解決了分歧的問題。

![](https://i.imgur.com/ZbnLFRZ.png)

在此例中，tangle選擇了Y為有效交易，基本上這項選擇是隨機的，但是一旦決定之後它便不可能再回朔。
至於偵測的時間，你可以依據以下因素判定：
- 新交易的數量
- tangle的平均「寬度」

`IOTA Donation:
ZDB9IS9WFPQQVLLQPREF9BSGNZUCWN9IDGOBXTGJEFIYEIJZYNHREMPAVYIDNNZUYHRBHICSXSVWZVSECTGBPNPRKB`