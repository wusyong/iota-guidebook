# 海綿函數

海綿函數（sponge function）又稱作海綿建構（sponge construction）是由 Guido Bertoni和其團隊整人在 2007 年提出。海綿函數是一種密碼學的演算法，它使用有限的狀態，接收任何長度的輸入位元流，然後可以滿足任何長度的輸出。簡單來說就是將數據「吸收」進海綿後，在「擠出」所需的結果出來。所以海綿函數大致上可以分成兩個階段，首先是將輸入訊息反覆壓縮的吸收階段，然後再反覆取出結果的擠出階段。


## IOTA 中應用的地方

海綿函數可以在用來架構或者實做密碼學的原始函數，像是加密雜湊函數等等。本文以 IOTA 中的 javascript library 為例，IOTA 用此方式來[產生地址](https://github.com/iotaledger/iota.lib.js/blob/master/lib/api/api.js#L758)以及[檢驗碼](https://github.com/iotaledger/iota.lib.js/blob/master/lib/utils/utils.js#L62)（checksum）等等。在上述連結中再深入 [signing.js](https://github.com/iotaledger/iota.lib.js/blob/master/lib/crypto/signing/signing.js) 尋找的話可以找到 key 和 digest 等函數，其中就可以看到不少 absorb 和 squeeze 的步驟。

而這些吸收擠出的步驟的 wrapper 來自 [Kerl](https://github.com/iotaledger/iota.lib.js/blob/master/lib/crypto/kerl/kerl.js#L8)，它使用的 module 是 [CryptoJS](https://code.google.com/archive/p/crypto-js/)，不過其中使用的雜湊函數並不是原先的 SHA-3-384 而是 Keccak-384，因為 Keccak 雜湊函數就是以海綿函數的方式建立的。

## 結構

海綿函數由三個部分組成：
* 一個內存狀態 S，包含 b 個位元。其中會分成兩個區塊，R（大小為 r 位元）與 C（大小為 b-r 位元 = c 位元）。r 又叫做轉換率（bitrate），而 c 叫做容量（capacity）。
* 一個能置換或者轉換內存狀態，固定大小的轉換函數 f，IOTA 使用 Keccak-384 作為此轉換函數。
* 一個填充函式（padding function） P，它會增加輸入 M 至足夠的大小，好讓填充後的輸入為轉換率 r 的整數倍，這樣就能將輸入切成 r 的數個分段。

![](https://i.imgur.com/ilAdUxA.png)


接下來海綿函數的運作方式如下，首先從「吸收階段」開始：
* S 先初始化為零
* 輸入經過填充函式處理再切成 r 個位元的分段 M0、M1、M2...等等。
* 填充後輸入的 M0 會與 R 進行 XOR 運算
* S 經過函數 f 轉換成 f(S)
* 更新後的 R 會跟下一個填充輸入 M1 進行 XOR 運算
* 更新後的 S 在經過函數 f 轉換...
* ...

一直重複這樣的步驟直到每個填充輸入分段都使用過為止。

再來海綿函數就能依照以下「擠出階段」輸出結果：
* S 裡面前 r 個位元就是輸出資料 Z0
* 如果需要更多輸出結果的話，再將 S 轉換成 f(S)
* S 的前 r 個位元就是下一個輸出 Z1
* ...

重複這樣的步驟直到擠出所需的長度大小，最後 Z0、Z1、Z2...就會組成輸出結果。

值得注意的地方是，輸入不會與 C 作 XOR 運算也不會被輸出。 C 在這裡僅僅只和轉換函數 f 相關，用來防止撞擊攻擊（Collision attack）或者原像攻擊（preimage attack），它的大小通常會是所希望防止等級的兩倍。

有關更多海綿函數的資訊可以參考：[Keccak Team](https://keccak.team/sponge_duplex.html)
