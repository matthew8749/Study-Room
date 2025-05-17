# FIFO介紹及設計方法

## 1.FIFO (First Input First Output) 架構
在數位電路中，FIFO(First In, First Out)是一種資料緩衝區(buffer)，用來暫存資料並確保輸入資料的順序在輸出時保持不變，此記憶體的資料先進先出(後進後出)。
根據FIFO工作的時脈域分為兩種:
- 同步FIFO: 指<u>讀時脈</u>和寫入時脈為同一個時脈在時脈沿來臨時同時發生讀寫。
- 非同步FIFO: 讀寫時脈不一致，讀寫相互獨立。所以存在跨時脈域的處理。

一個硬體 FIFO 通常包括以下部分：
儲存陣列（Memory Array）：暫存資料。
寫指標（Write Pointer）：指出下一筆資料要寫入的位置。
讀指標（Read Pointer）：指出下一筆資料要讀出的資料位置。
空 / 滿 訊號（Empty / Full Flags）：判斷 FIFO 是否已滿或已空。

## 同步FIFO(Synchronous FIFO)設計
### - 深度和寬度 ()


## 異步FIFO(Asynchronous FIFO)設計






## Reference
1. [同步FIFO的兩種Verilog設計方法] (https://blog.csdn.net/wuzhikaidetb/article/details/121136040)
