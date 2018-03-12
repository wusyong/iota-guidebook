# 快照與附加到 Tangle
## 快照
Snapshot 是種將整體 Tangle 資料庫壓縮節省的技術，主要會將所有 Tangle 上的交易都移除，沒有任何餘額的地址也會在紀錄中移除，只留下擁有餘額的地址。這些地址餘額會在 snapshot 後扮演創始交易的角色。要是有 snapshot 要進行，可以在官方 [Discord](https://discordapp.com/channels/397872799483428865/398069502060789761) #announcements 查看公告。

Snapshot 會產生出兩個檔案 `Snapshot.sig` 和 `Snapshot.txt` 。`Snapshot.sig` 為該 Snapshot 的簽章檢查其完整性，`Snapshot.txt` 就純粹是全部尚有餘額的地址列表。此列表可在 IOTA Reference Implementation (IRI) 中找到：[iri/blob/dev/src/main/resources/Snapshot.txt](https://github.com/iotaledger/iri/blob/dev/src/main/resources/Snapshot.txt)。

以下節錄開頭片段，可以看到基本上就是地址與餘額 pair：
```bash
999999999999999999999999999999999999999999999999999999999999999999999999999999999;22364358
9999UHZHBGAASSILKUXQRIKLJRFUCJLKPVILN9BVKEPWT9KINNZHKSGHMYSOWCUPIIJBYWPRMHDGIEAX9;100000000
999EAAHKCCXRDOCNADIUCPKSZXON9UCHXYGIZAAOIWVEFJFJBRHFVBZMZYSRLUVIJSINIPKHBWNDZKTCB;1611
999GPCVEVCNHLKIZQEZSESMABAJUPEUMRXNPEWFRLHMYTAETPLOPZRHGRDPAQYKWNJAOIOFNSRYWPJHFW;499500000
999LKQV9IOCOOETLRRECUCBBOW9EZTVMBUBIDDJBNAKYQJUXLKNCCQCZVSB9GCKTXQNLYPKI9R9JTABCD;834640574
```

## 確定性（deterministic）錢包
IOTA 屬於確定性錢包，也就是說新的地址是從種子與地址 index 組合後計算產生出來的，地址 index 可以是任何正整數。錢包地址的 index 是從 0 開始，與節點連線後取得相對應地址的交易紀錄列表。

如果該地址沒有查詢到存在的交易的話，錢包會視該地址還沒使用過，錢包會就此停止搜尋地址 index 然後顯示目前為止所有地址的餘額總和。反之，如果該地只有查詢到交易的話，地址 index 就會增加，也就是說會繼續產生新的地址。接下來錢包會重複這樣的步驟，繼續查詢新的地址在 Tangle 上有沒有交易。值得一提的是錢包會跳過已經附加到 Tangle 的地址 index。

我們這邊簡單舉個例子，今天假設有個種子有五個地址，但都還沒有附加到 Tangle 上（有可能是 snapshot 之後，也有可能是就只是新產生的種子）。今天有人發送交易給這個種子，不過是分別發送到 index 1 和 3 的地址，錢包會如下所示：
```bash
0. AAAAA...;0
1. BBBBB...;5
2. CCCCC...;0
3. DDDDD...;1
4. EEEEE...;0
```

因為地址都沒有附加到 Tangle 上，當你用此種子登入時 index 會停留在 index 0。如果這時附加 index 0 的地址，錢包往下就會看到 index 1 的餘額然後停在 index 2，這時錢包顯示的總額會是 5 元。如果繼續把 index 2 地址附加到 Tangle 上的話錢包往下就會看到 index 3，總額就會變成 6 元。


## 附加到 Tangle
附加到 Tangle 的詳細運作方式這邊不會詳細說明，僅約略解釋一下大致上做了什麼步驟：
* 用該地址發送 0 元交易
* 驗證 Tangle 上的兩個交易
* 進行工作證明（PoW）

附加上的地址會出現在錢包的歷史紀錄中，其交易的金額就會是 0 元。這步驟實際上不是強制性的，你可以發送 IOTA 到尚未附加的地址。不過**建議要使用地址前永遠先附加到 Tangle 上**。附加到 Tangle 算是一種保險機制，告訴錢包不要重複使用到相同的地址。要是重複使用地址尤其是發送交易的話，會產生很嚴重的安全衝擊。

## 重複使用地址
要是你使用相同的地址重複發送交易的話，將會指數性降低交易的安全性。因為 IOTA 使用的是 Winternitz one-time signatures，每當你發送一筆交易時都會揭露該地址私鑰的一部份。要是你發送的次數越多，意味著有心人士越容易暴力破解你的私鑰來偷取該地址的餘額。不過你是可以用同個地址無限制地接收任何交易，只要你不拿來發送交易即可。

每當 snapshot 之後因為地址紀錄刷新，錢包不會紀錄交易的歷史資訊，很多人常常會因此重複使用地址，現在官方錢包已經有更新會檢查地址是否可能重複使用。關於 snapshot 回復的方式會在[快照準備事項](snapshot-pre.md)提及，另一種避免的方式是在 snapshot 前把錢包餘額全部轉移到新的種子，因為該種子內的地址都沒有被使用過。

至於後續有關 Snapshot 的開發在 IOTA Roadmap 有兩個項目提及：
* Automated Snapshotting：目前 snapshot 仍為手動進行，之後會交由節點自動進行 snapshot。
* Permanodes: 有鑑於 Snapshot 會讓之前的交易記錄消失，Permanodes 會儲存所有的交易紀錄，即使 snapshot 有發生。
