# The Tangle：圖解介紹

這篇是新手教學系列中的第一篇，旨在闡述 IOTA 內部是如何運作的。我們會大概按照[**白皮書**](https://iota.org/IOTA_Whitepaper.pdf)上所說的做解釋，不過會慢下腳步附上圖解一步步解釋基本概念。在本文中我們將介紹什麼是 _tangle_ 以及我們 IOTA 研究團隊是如何用數學的角度瞭解它的。

欲瞭解 tangle，我們首先需要知道何謂資工人所說的[**_有向圖_**](https://en.wikipedia.org/wiki/Directed_graph)。所謂的有向圖就是一堆 _頂點_ （方框）用 _邊_ （箭頭）連接起來的集合，下圖就是一張有像圖的例子：

![](https://cdn-images-1.medium.com/max/800/0*ugPaad_14ESxwsPi.)

**Tangle** 是 IOTA 背後運作的資料結構，它就是一種包含 _交易數據_ 的特殊有向圖，圖中每個頂點代表的是每筆交易。當有新的交易加入 tangle 時它必須 _確認_ 之前兩筆交易，新增兩個邊到圖中。在上面的例子中，交易 5 確認了交易 2 和 3。交易資訊的格式和大家通常想到得差不多，基本上就是「某 A 給某 B 10 IOTAs」。 目前我們不會太去在意交易確認的問題，之後我們會再去探討。

我們會稱尚未確認的交易為 _tips_ 。在此例中，交易 6 就是個 tip，因為還沒有任何人去驗證它。每個加入的交易都需要選擇兩個 tips 來確認（永遠至少會有一個）。選擇哪兩個 tips 來確認的策略非常重要，這是 IOTA 技術獨特的關鍵之一。不過為了方便起見，我們這邊會先採用比較簡單的策略：在所有 tips 中隨機選擇就好。每個加入的交易會查看所有尚未確認的交易，然後隨機選擇兩個即可。

為了顯示當所有人都使用隨機策略（技術上來說叫做「均勻隨機選擇」）時 tangle 會是怎麼樣子，我們做了個[**視覺模擬**](https://public-rdsdavdrpd.now.sh/)。

此模擬會產生隨機 tangle，最左側的就是第一筆交易（稱作 _創始交易_ ）而最右側則是最近的交易，tips 會用灰色方框標示起來。當你將滑鼠移到其中一個交易時，所有受到此交易驗證的交易會用紅色標示起來，而所有確認它的交易則是藍色。


![](https://cdn-images-1.medium.com/max/800/1*H8xoyZN5ESR1_FWa1xijEQ.gif)

目前我們先解說到此，我們邀請你玩玩看這個模擬，試試看不同的設定，要是有問題的話可以到 _#tanglemath_ [**Discord**](https://discord.gg/bFj3nM) 頻道詢問。在下一篇文章中，我們將會解釋什麼是 _交易速率_ (λ)、解釋更多進階概念像是間接驗證和隱藏的 tips 等以及學習更多複雜的 tip 選擇策略：無權重隨機漫步。

翻譯自：[The Tangle: an Illustrated Introduction
](https://blog.iota.org/the-tangle-an-illustrated-introduction-4d5eae6fe8d4)
