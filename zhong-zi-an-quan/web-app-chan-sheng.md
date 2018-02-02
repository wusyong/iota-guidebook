# 使用 Web App 安全產生種子的方法
考量到網頁產生器仍然是多數人在接觸新的貨幣錢包，產生私鑰最常用的手段（像以太通常大家應該會最先使用 MEW）。以下提供由 IOTA knowledge base 推薦的 Web App，並提供簡單步驟就能確保產生種子時基本上是安全的。最下方是檢查此 Web App 的安全與完整性，可以選擇略過。

使用瀏覽器產生種子
---
如果真的想用 Web App 來產生 IOTA 種子的話可以參考以下步驟：
- 前往 https://ipfs.io/ipfs/QmdqTgEdyKVQAVnfT5iV4ULzTbkV4hhkDkMqGBuot8egfA 然後右鍵儲存網頁到你的電腦裡。
- **將你的電腦斷線，像是關閉 WiFi 或是拔掉網路線等等。**
- 開啟下載的網頁然後開始移動滑鼠直到 100%
- 將 IOTA 種子儲存在安全的地方（可以參考看看 [KeePass](https://hackmd.io/s/BJu5FZVBf)）

Web App 安全性驗證
---
- IOTA knowledge base 所推薦能夠安全產生種子的 Web App 為： https://ipfs.io/ipfs/QmdqTgEdyKVQAVnfT5iV4ULzTbkV4hhkDkMqGBuot8egfA
- 此種子產生器的開源碼在此：https://github.com/knarz/seedgen
- knarz/seedgen 使用的是 Stanford Javascript Crypto Library，此 Library 連結在此：https://github.com/bitwiseshiftleft/sjcl
- 關於該 Library 更多訊息可以參考以下連結：
http://bitwiseshiftleft.github.io/sjcl/ 
http://bitwiseshiftleft.github.io/sjcl/doc/

> The Stanford Javascript Crypto Library is a project by the Stanford Computer Security Lab to build a secure, powerful, fast, small, easy-to-use, cross-browser library for cryptography in Javascript.

以下是主要的程式碼並附上一些註解：
```
//Random number generator.
var gen = new sjcl.prng(10);

//start the built-in entropy collectors
gen.startCollectors();

//Called when generator is done.
sjcl.random.addEventListener("seeded", function() { document.getElementById("seed").innerText = genSeed(); });

//Called each time entropy is added.
sjcl.random.addEventListener("progress", function(p) { 
    if(p != 1) {
        document.getElementById("seed").innerText = "Collecting entropy, please move your mouse/device\nProgress: " + p * 100 + "%"
    }
});
			
//sjcl.random.randomWords(nwords, paranoia) generates several random words, and return them in an array.
//sjcl.codec.base64.fromBits() converts from a bitArray to a base64 string.
//Remove all characters not belong to "A-Z9"
//Return the seed which is 81 characters long
function genSeed() {
    var seed = "";
    for(;seed.length < 81;seed += sjcl.codec.base64.fromBits(sjcl.random.randomWords(33, 10)).replace(/[^A-Z9]+/g, '')) {};
    return seed.substring(0,81);
}
```

`IOTA Donation:
ZDB9IS9WFPQQVLLQPREF9BSGNZUCWN9IDGOBXTGJEFIYEIJZYNHREMPAVYIDNNZUYHRBHICSXSVWZVSECTGBPNPRKB`