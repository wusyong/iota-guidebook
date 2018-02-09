# 權重相關術語解釋
## 自身權重
* 每筆交易會有一個初始的權重稱為自身權重，其數值可以是 $3^n$，像是 1、3、9 等等。
* 交易的自身權重與發送這筆交易的節點所投入的工作量成正比

## 累積權重
* 累積權重為交易自身權重加上所有直接和間接驗證此交易的交易權重之和
* 以下圖為例，右下角的數字為自身權重，左上角的數字則為累積權重
* 交易 D 直接或間接受到 A、B 和 C 驗證，所以交易 D 的累積權重會是 1+3+1+1 = 6
* 交易 F 直接或間接受到 A、B、C 和 E 驗證，所以交易 D 的累積權重會是 1+3+1+3+1 = 9

![](https://i.imgur.com/1oeX0hW.png)

* 累積權重在交易網路的確認中扮演中非常重要的衡量單位
* 擁有較大累積權重的交易會比較小的交易還要來的重要
* 每筆加入 tangle 的交易會將自身權重增加到先前交易的累積權重，也就是說存在越久的交易會隨著時間變得越來越重要
* 累積權重的應用能防範濫發交易以及其他攻擊形式，我們預設在一段很短的時間內是無法產生大量而且合理的累積權重交易

## 高度
* 高度定義為自創世交易 (genesis) 至當前這個交易的所有路徑中最長的長度。
* 舉例來說 G 的高度為 1 ，D 的高度為 3。
![](https://i.imgur.com/Oyt3IWK.png)

## 深度
* 深度定義為自這個交易到某個 tip 的最長路徑。
* 舉例來說 G 對於 Tip A 的深度為 4，路徑為 F、D、B 與 A。
![](https://i.imgur.com/Oyt3IWK.png)

## Minimum Weight Magnitude
* Minimum Weight Magnitude（MWM）是工作證明（PoW）的難度設定
* IOTA 的 PoW 演算法類似於 Hashcash
* MWM 指的就是連續串為零的數量
* 以下簡略介紹 Hashcash 運作方式（假設 MWM 為 4）：
hash(transaction_data+counter)=...d83902hfos84f23 （PoW 未完成）
hash(transaction_data+counter)=...fj28068gfdb0000 （PoW 完成）
* 根據 IRI release v1.4.2.1，mainnet 的 MWM 為 14
* 根據 IRI release testnet-v1.4.1.3，testnet 的 MWM 為 9
* 增加 MWM 不會有甚麼問題，不過要注意會增長 PoW 的時間
* MWM 的變更可在以下路徑找到： iri/src/main/java/com/iota/iri/conf/Configuration.java

<script src="https://gist.github.com/wusyong/6f4bc35187d94076be817b728900fc8f.js"></script>
