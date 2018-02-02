# IOTA Seed 產生器

在IOTA的世界裡，只要有Seed就能存取錢包，因此使用者有責任保護好自己的Seed。

官方對於安全產生Seed的建議：[https://blog.iota.org/the-secret-to-security-is-secrecy-d32b5b7f25ef](https://blog.iota.org/the-secret-to-security-is-secrecy-d32b5b7f25ef)

首先，**不要**使用線上種子生成器。

一個簡單的方法：
在一張紙上隨機記下大寫字母（A-Z）和數字9，直到你寫出了81個字符。
就是這樣，你完成了！

不幸的是，人類隨意挑選東西通常不好，所以我們可以使用一些工具來增加我們種子的隨機性。 

## Linux
開啟終端機，輸入以下指令：
```shell=
cat /dev/urandom |tr -dc A-Z9|head -c${1:-81}
```

## Mac OS
開啟終端機，輸入以下指令：
```shell=
cat /dev/urandom |LC_ALL=C tr -dc 'A-Z9' | fold -w 81 | head -n 1
```

## Windows
使用 KeePass，KeePass是一款開源的密碼管理器
KeePass Wiki：https://zh.wikipedia.org/wiki/KeePass
KeePass 官網(下載)：https://keepass.info

![](https://i.imgur.com/XcppCix.png)

## 安全至上
請好好保存你的Seed！

###### tags: `iota` 