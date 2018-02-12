# 在 Google Cloud Platform 上建立 IRI Full Node

GCP 會提供 $300 美元的金額以及一年的時間給新的帳號試用，對於架設節點來說可以免費維持約 3 到 6 個月。

**目錄：**  

[建立 VM 執行個體](#建立-VM-執行個體)  
[安裝 IRI](#安裝-IRI)  
[監測 IRI](#監測-IRI)  
[安裝 Nelson](#安裝-Nelson)  
[其他補充](#其他補充)

## 建立 VM 執行個體

### 1. 建立 GCP 帳號

[https://console.cloud.google.com/](https://console.cloud.google.com/)

### 2. 建立新的專案

按下「+」然後這邊我們將專案命名為「IOTA」  
![](https://i.imgur.com/lkJZbwN.png)

### 3. 建立執行個體（Instance）

* 在搜尋列輸入 `Compute Engine` 然後選擇 `執行個體, Compute Engine` 的選項
* 按下「建立」然後名稱可以取作「iota-full-node」
* 選擇區域，要是想架在台灣的話可以選「asia-east1」，其他位置可以[參考此連結](https://cloud.google.com/compute/docs/regions-zones/)
* 規格最少為 2vCPU/4gb RAM，建議的話則為 4vCPU/6gb+（筆者自己測試 2core 6gb 應該就足夠）
* 開機磁碟選擇「Ubuntu 16.04 LTS」，大小建議為 96gb。HD 通常來說就足夠，不過你希望平順且更快的話建議就選 SSD。
* **重要：** 在「身分及 API 存取權」中，請記得選擇「允許所有 Cloud API 的完整存取權」。
* **重要：** 在「防火牆」選項中，請確定「允許 HTTP 流量」和「允許 HTTPS 流量」均有勾選。
* 都設定好後就按下「建立」。

### 4. 防火牆設定

* 我們得開啟一些 ports 才能讓 IRI 與 Nelson 妥善運作。在搜尋列輸入 `外部 IP 位址`，接著將你伺服器的 IP 從 「臨時」改為「靜態」。
* 現在我們要啟用必要的 ports，按下左列的「防火牆規則」，然後「建立防火牆規則」。我們有 4 個 port 要設定，這邊會用預設作為舉例，分別是 14265\(IRI\)、14600\(udp\)、15600\(tcp\)、16600\(nelson\)。
* 先用 14265 為例，名稱可以取作像是「iota-14265-in」
  1.流量方向選「輸入」
  2.相符時執行的動作選「允許」
  3.目標選「網路中的所有執行個體」
  4.來源 IP 範圍輸入 `0.0.0.0/0`
  5.指定的通訊協定和通訊埠輸入 `tcp:14265; udp:14265`
  ![](https://i.imgur.com/lI31sRL.png)
* 建立另一個新的取作「iota-14265-out」，不過流量方向選「輸出」。
* 重複上面兩個步驟於 port 16600。
* 至於 udp 14600 和 tcp 15600 一樣是重複上面的步驟，但是在指定的通訊協定和通訊埠輸入只須分別輸入 `udp:14600;` 和 `tcp:15600`。
* 確認以上 port 都有建立「輸入」與「輸出」的規則，都設定好後用搜尋列回到「執行個體」，按下 `SSH` 就能建立 SSH 連線，往下我們就可以開始安裝 IRI。
  ![](https://i.imgur.com/n70XMrl.png)

## 安裝 IRI

安裝 IRI 之前，我們得為伺服器更新並升級我們需要的東西。

### 1. 更新

首先更新的是 OS ，kernel 更新會需要重啟，在終端機輸入以下指令。  
如果會需要重啟的話輸入 `sudo reboot`。

```bash
sudo apt update -qqy --fix-missing && sudo apt-get upgrade -y && sudo apt-get clean -y && sudo apt-get autoremove -y --purge
```

### 2. PACKAGES

接下來會需要安裝 Java，這邊會使用 Oracle，因為按照討論區所說的 OpenJDK 可能遇到的問題會比較多，而且 Oracle 所耗的資源也較少。另外我們用的版本是較舊的 8，因為 9 似乎也有一些已知問題。不過 Oracle 會需要同意 license，所以到時候在安裝時你會需要同意彈出視窗的訊息。jq package 讓我們更方便閱讀 JSON 輸出的訊息。

```bash
sudo apt install software-properties-common -y && sudo add-apt-repository ppa:webupd8team/java -y && sudo apt update && sudo apt install oracle-java8-installer curl wget jq git -y && sudo apt install oracle-java8-set-default -y
```

### 3. 設定

我們要告訴 OS 預設的 java 是哪個

```bash
sudo sh -c 'echo JAVA_HOME="/usr/lib/jvm/java-8-oracle" >> /etc/environment' && source /etc/environment
```

### 4. USERS

這邊新增兩個 2 使用者以便安全，一個用於 IOTA 而另一個用於 nelson。

```bash
sudo useradd -s /usr/sbin/nologin -m iota
sudo useradd -s /usr/sbin/nologin -m nelson
```

### 5. 目錄

建立節點（IRI）應用程式的目錄架構。

```bash
sudo -u iota mkdir -p /home/iota/node /home/iota/node/ixi /home/iota/node/mainnetdb
```

### 6. 安裝 IRI

從 github 下載官方最新的版本\(1.4.2.1\)到節點目錄。

```bash
sudo -u iota wget -O /home/iota/node/iri-1.4.2.1.jar https://github.com/iotaledger/iri/releases/download/v1.4.2.1/iri-1.4.2.1.jar
```

### 7. SYSTEMD SERVICE

我們會希望節點能在重啟或是當機後自動啟動，所以這邊建立個 systemd service 給它。

_**注意事項：**_ 我們會限制 java 能使用的記憶體並留一些給 Ubuntu 以及 IRI-DB。請按照你有的 RAM 大小選擇所需的 Xmx 參數，這邊的選擇是約略 75%，你也可以自己設定。將所選的參數替換到以下的`< Xmx 參數 >`。

* 4gb: `-Xmx3g`
* 6gb: `-Xmx4500m` 或 `-Xmx4g`
* 8gb: `-Xmx6144m` 或 `-Xmx6g`
* 10gb: `-Xmx8192m` 或 `-Xmx8g`
* 12gb: `-Xmx10240m` 或 `-Xmx10g`
* 16gb: `-Xmx14336m` 或 `-Xmx14g`  

複製並貼上到終端機，請確認你有設定好 Xmx 參數：

```bash
cat << "EOF" | sudo tee /lib/systemd/system/iota.service
[Unit]
Description=IOTA (IRI) full node
After=network.target

[Service]
WorkingDirectory=/home/iota/node
User=iota
PrivateDevices=yes
ProtectSystem=full
Type=simple
ExecReload=/bin/kill -HUP $MAINPID
KillMode=mixed
KillSignal=SIGTERM
TimeoutStopSec=60
ExecStart=/usr/bin/java < Xmx 參數 > -Djava.net.preferIPv4Stack=true -jar iri-1.4.2.1.jar -c iota.ini
SyslogIdentifier=IRI
Restart=on-failure
RestartSec=30

[Install]
WantedBy=multi-user.target
Alias=iota.service

EOF
```

再來輸入以下指令便能設定好，如果之後有調整到檔案的話記得輸入`sudo systemctl daemon-reload`

```bash
sudo systemctl daemon-reload && sudo systemctl enable iota.service
```

如此一來，以後輸入下面的指令就能啟動或是暫停節點（_**進行到目前這步時先別啟動 IRI 伺服器**_）： `sudo service iota start|stop|restart|status`

### 8. IRI \(IOTA REFERENCE IMPLEMENTATION\)

接下來需要設定一些 ports 以及鄰居。`PORT` 是用來與 IRI 的 API 溝通，錢包就會使用此 port。`UDP_RECEIVER_PORT` 與 `TCP_RECEIVER_PORT` 用來與我們的鄰居 `NEIGHBORS` 溝通，以下已經有預設 port，你也可以自行更改。至於鄰居你可以前往官方 Discord \#nodesharing 或是到 [helloiota](https://forum.helloiota.com/Community/Node-Sharing) 討論區尋找有意願的人。  
如果到時候有同步問題的話，可以考慮看看加入 LiQio 或是 CFB 的 Swarm Node：udp://94.156.128.15:14600  
udp://88.99.249.250:41041  
udp://185.181.8.149:14600

```bash
cat << "EOF" | sudo -u iota tee /home/iota/node/iota.ini
[IRI]
PORT = 14265
UDP_RECEIVER_PORT = 14600
TCP_RECEIVER_PORT = 15600
API_HOST = 0.0.0.0
IXI_DIR = ixi
HEADLESS = true
DEBUG = false
TESTNET = false
DB_PATH = mainnetdb
RESCAN_DB = false

REMOTE_LIMIT_API = "removeNeighbors, addNeighbors, interruptAttachingToTangle, attachToTangle, getNeighbors, setApiRateLimit"

NEIGHBORS = < 在此輸入鄰居 >
EOF
```

如果你有要使用 Nelson 的話，這邊可以先往下跳到安裝 Nelson 的步驟，之後再回來繼續設定。

### 9. 下載 DATABASE

加入 Database 能讓 IRI 更快同步（Sync），如果你有好鄰居的話通常你只需要再同步前一個小時的 milestone，也不需要再 rescan。整個 DB 大約有 8gb，_**這會花費一些時間來下載**_。

```bash
cd /tmp/ && curl -LO http://db.iota.partners/IOTA.partners-mainnetdb.tar.gz && sudo -u iota tar xzfv /tmp/IOTA.partners-mainnetdb.tar.gz -C /home/iota/node/mainnetdb && rm /tmp/IOTA.partners-mainnetdb.tar.gz
```

### 10. 啟動節點

當你有至少一位鄰居後就能開始了，第一次啟動節點請輸入以下指令。`restart` 的話雷同，若是到時候節點沒有正常運作，可以用同樣方式重啟。

```bash
sudo service iota start
```

我們希望能夠自動更新 IRI，以下指令每 15 分鐘檢查有無新的 IRI 版本並自動安裝

```bash
echo '*/15 * * * * root bash -c "bash <(curl -s https://gist.githubusercontent.com/zoran/48482038deda9ce5898c00f78d42f801/raw)"' | sudo tee /etc/cron.d/iri_updater > /dev/null
```

## 監測 IRI

以下提供 IRI API 來觀測你的節點，而要離開程式的話通常按 `q`、`ESc`、`strg+c` 即可。

### 1. 同步（SYNCHRONISATION）

當 `latestMilestoneIndex` == `latestSolidSubtangleMilestoneIndex` 就代表你的節點同步了，目前的 milestone 可以在官方 Discord \#botbox 頻道找到。通常當你重啟時 latestSolidSubtangleMilestoneIndex 會顯示為 `999999999...`，不過經過數分鐘不等之後就會顯示 \#botbox 上的數值了，通常落後 \#botbox 一點點的話算正常現象。

輸入以下指令可以看到 logfile 來確認是否有同步成功：  
`[Solid Milestone Tracker] INFO com.iota.iri.Milestone - Latest SOLID SUBTANGLE milestone has changed from #123456 to #123457`

瀏覽 logfile

```bash
journalctl -u iota
```

顯示目前最新的 logfile

```bash
journalctl -u iota -f
```

### 2. 顯示所有鄰居

一個可靠的鄰居可以依據 `numberOfNewTransactions` 來判斷，要是長時間沒有增加或是維持在 0 的話就會需要注意該鄰居狀況。

```bash
curl http://localhost:14265 -X POST -H 'Content-Type: application/json' -H 'X-IOTA-API-Version: 1.4' -d '{"command": "getNeighbors"}' | jq
```

### 3. 顯示 IRI 狀態

_**注意事項：**_ milestone 從 243000 更新到最新數值會花一些時間，通常這和你的鄰居有關。要是 IRI 卡在某個 milestone 非常久的話，請停止 service、刪掉 database 和 database logs，然後重新下載並安裝 database，最後重啟 iota service。

```bash
curl http://localhost:14265 -X POST -H 'Content-Type: application/json' -H 'X-IOTA-API-Version: 1.4' -d '{"command": "getNodeInfo"}' | jq
```

### 4. 管理鄰居

以下 API 能及時增減鄰居來方便管理，不過最後請記得要在 `/home/iota/node/iota.ini` 調整你的鄰居，不然下次重啟節點的話，是不會記住這些設定的。

* 新增鄰居：
  用分號（,）分隔開來
  ```bash
  curl -H 'X-IOTA-API-VERSION: 1.4' -d '{"command":"addNeighbors", "uris":[
  "tcp://ip-of-the-new-neighbor:12345", "udp://ip-of-the-new-neighbor:54321"
  ]}' http://localhost:14265
  ```
* 移除鄰居：
  和新增的方式差不多
  ```bash
  curl -H 'X-IOTA-API-VERSION: 1.4' -d '{"command":"removeNeighbors", "uris":[
  "tcp://ip-of-the-new-neighbor:12345", "udp://ip-of-the-new-neighbor:54321"
  ]}' http://localhost:14265
  ```

要是你看到有鄰居如下 `numberOfNewTransactions` 卡著一直沒有產生新的交易，或者是 `numberOfInvalidTransactions` 不為 0 的話，可能就要考慮移除它了。

```bash
{
  "address": "example.neighbor:15600",
  "numberOfAllTransactions": 5487,
  "numberOfRandomTransactionRequests": 0,
  "numberOfNewTransactions": 0,
  "numberOfInvalidTransactions": 1,
  "numberOfSentTransactions": 9527,
  "connectionType": "tcp"
}
```

## 安裝 Nelson

### 1. 安裝 nodejs 8+ 與 npm:

```bash
curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -
sudo apt-get install -y nodejs
```

### 2. 到 home 目錄然後建立 nelson 的目錄

```bash
sudo -u nelson mkdir /home/nelson
```

### 3. 安裝 Nelson

```bash
sudo npm install -g nelson.cli
```

### 4. 調整設定 config

```bash
cat << "EOF" | sudo -u iota tee /home/nelson/config.ini
[nelson]
cycleInterval = 60
epochInterval = 300
apiPort = 18600
apiHostname = 127.0.0.1
port = 16600
IRIHostname = localhost
IRIPort = 14265
TCPPort = 14600
UDPPort = 15600
dataPath = data/neighbors.db
incomingMax = 5
outgoingMax = 4
isMaster = false
silent = false
gui = false
getNeighbors = https://raw.githubusercontent.com/SemkoDev/nelson.cli/master/ENTRYNODES
; add as many initial Nelson neighbors, as you like
neighbors[] = mainnet.deviota.com/16600
neighbors[] = mainnet2.deviota.com/16600
neighbors[] = mainnet3.deviota.com/16600
neighbors[] = iotairi.tt-tec.net/16600
EOF
```

### 5. 啟動 Nelson

如果你還沒有開始你的 IRI service 的話，先回去剛剛上面的步驟來啟動節點。

接下來我們想和 IRI 一樣設為 service，請輸入以下設定：

```bash
cat << "EOF" | sudo tee /lib/systemd/system/nelson.service
[Unit]
Description=Nelson
After=network.target

[Service]
User=root
Type=simple
ProtectSystem=full
TimeoutStopSec=60
KillMode=mixed
ExecStart=/usr/bin/nelson --config /home/nelson/config.ini
SyslogIdentifier=NELSON
Restart=on-failure
RestartSec=5s

[Install]
WantedBy=multi-user.target
EOF
```

然後輸入：

```bash
sudo systemctl daemon-reload && sudo systemctl enable nelson.service
```

這樣就能啟用 Nelson：

```bash
sudo service nelson start
```

### 6. 更新 Nelson

有鑑於 Nelson 算是另外幫你找鄰居的服務，正常狀況下還是建議自己手動更新。要更新時先停止 Nelson：

```bash
sudo service nelson stop
```

再到 [nelson.cli NPM](https://www.npmjs.com/package/nelson.cli) 頁面查看最新的版本（以下用 0.3.22 舉例）

```bash
sudo npm install -g nelson.cli@0.3.22
```

最後就能重開 Nelson 了。

## 其他補充

### 鄰居

相信大家很快就會發現鄰居可靠的重要性，所以建議在 `iota.ini` 上加上註解隨時注意鄰居的健康狀況，要是 `numberOfNewTransactions` 為 0 或長時間卡住的話，請在移除之前通知他。你想要鄰居友善對待你，相對地你也需要妥善地對待鄰居。如果出現 `NumberOfInvalidTransactions` 則建議改快移除在通知，這對你的節點是有傷害性的。

### 錢包

架設完整節點的另一項好處是你可以自己拿來使用，就不必再和其他人一樣擠外面的公開節點了，你可以在錢包工具上設定節點為你所自己架設的。注意 milestone 是否有同步再繼續使用。如果發現餘額為 0 的話，處理步驟和 snapshot 一樣持續產生地址即可。

`ETH Donation: 0xdda02772C1C5b48B72aDe8Be67d203CF4AEa8f7F`  
`IOTA Donation: ZDB9IS9WFPQQVLLQPREF9BSGNZUCWN9IDGOBXTGJEFIYEIJZYNHREMPAVYIDNNZUYHRBHICSXSVWZVSECTGBPNPRKB`
