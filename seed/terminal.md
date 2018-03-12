# 使用終端機產生種子

## 使用終端機產生種子
* 根據官方 [IOTA 知識庫](https://kb.helloiota.com/KnowledgebaseArticle50005.aspx)
可以透過終端機用以下方是產生種子：
* Linux 作業系統：
開啟終端機後輸入以下指令：
**cat /dev/urandom |tr -dc A-Z9|head -c${1:-81}**
* Mac 作業系統：
開啟終端機後輸入以下指令：
**cat /dev/urandom |LC_ALL=C tr -dc 'A-Z9' | fold -w 81 | head -n 1**

## /DEV/URANDOM
* /dev/urandom 函數會透過將裝置驅動程式、網路封包時段與其他來源等環境樣本（entropy）輸入 entropy pool 來產生隨機數值。
* Entropy pool 數據會用為密碼學安全偽亂數生成器（CSPRNG）的輸入值
* 產生器會依此產生隨機數值
* urandom 指的是 unlimited random
* 在 Mac 中 /DEV/URANDOM 和 /DEV/RANDOM 並無差異，兩者行為是一樣的
* 在 Linux 中 /DEV/URANDOM 和 /DEV/RANDOM 則有差異，但這裡將不會討論此議題
