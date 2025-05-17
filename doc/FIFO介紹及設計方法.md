# FIFO介紹及設計方法

## 1.FIFO (First Input First Output) 架構
在數位電路中，FIFO(First In, First Out)是一種資料緩衝區(buffer)，用來暫存資料並確保輸入資料的順序在輸出時保持不變，此記憶體的資料先進先出(後進後出)。
根據FIFO工作的時脈域分為兩種:

- **同步FIFO:** 指<ins>讀時脈</ins>和<ins>寫入時脈</ins>為同一個時脈在時脈沿來臨時同時發生讀寫。  
同步FIFO之所以稱為“同步”，是因為它採用同步時脈來控制讀寫操作。  
FIFO的讀寫指標與時脈同步更新，資料與時脈同步在FIFO與外部電路之間傳輸。  
主要用於在資料傳輸速率超過資料處理速率時緩衝資料。這在高速系統中特別重要，因為時間差異可能導致資料遺失或損壞。  
- **非同步FIFO:** 讀寫時脈不一致，讀寫相互獨立。所以存在跨時脈域的處理。

- 一個硬體 FIFO 通常包括以下部分：
  - 儲存陣列（Memory Array）&ensp;&ensp;&ensp;&ensp;&nbsp;：暫存資料。
  - 寫指標（Write Pointer）&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&nbsp;：指出下一筆資料要寫入的位置。
  - 讀指標（Read Pointer）&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&ensp;&nbsp;：指出下一筆資料要讀出的資料位置。
  - 空/滿 訊號（Empty / Full Flags）：判斷 FIFO 是否已滿或已空。


<!-- 這句看不見，一句話的註解 --> 
<!-- 
| 注意部分                         	| 作用                             	|
|----------------------------------	|----------------------------------	|
| 儲存陣列（Memory Array）          | 暫存資料。                       	|
| 寫指標（Write Pointer）          	| 指出下一筆資料要寫入的位置。     	|
| 讀指標（Read Pointer）           	| 指出下一筆資料要讀出的資料位置。 	|
| 空/滿 訊號（Empty / Full Flags） 	| 判斷 FIFO 是否已滿或已空。       	|
 --> 
  

## 同步FIFO(Synchronous FIFO)設計

- 深度和寬度  
位寬 WIDTH：表示FIFO裡面每個資料的大小  
深度 DEPTH：表示FIFO能存多少個數據

**🧠 問題：**  
同步 FIFO 中，在同一個時鐘邊緣，如果：  
寫入資料到地址 A（write pointer 指向 A），同時讀出資料（read pointer 也指向 A）  
那麼，讀出來的資料是什麼？是「原本存在 A 的值」，還是「剛寫進 A 的新值」？

**📌 答案：**  
視 FIFO 記憶體的寫入／讀出時序行為而定！  
也就是說：這取決於底層 memory 的行為（通常是 RTL 中的 RAM module 的行為）。  

**📚 常見兩種情況：**  
1️⃣ 寫在前，讀到新資料（Write-first behavior）
- 寫入與讀出同一地址，在同一個時鐘邊緣。  
- 結果：讀出剛寫進去的新資料。

📌 適合有些同步 FIFO 的需求，尤其是 latency 敏感的應用。

2️⃣ 讀在前，讀到舊資料（Read-first behavior）
- 同樣寫入與讀出同一地址。
- 結果：讀出的是原本存在的舊資料（還沒寫進去的）。

📌 在某些記憶體 IP 或是合成工具中的 RAM，預設是這種行為。  

**✅ 設計建議：**  
為了避免這種模糊狀況，通常會強制在 write 和 read 指標不同時才允許操作，或是加一層保護邏輯，例如：
- 只有當 FIFO 中有資料才允許讀（避免空讀）。
- 只有當 FIFO 還沒滿才允許寫（避免覆蓋）。  

若要支援同步寫讀同地址，記憶體模組必須設計成 write-first 或 read-first，並在文件中說明。


## 異步FIFO(Asynchronous FIFO)設計






## Reference
1. [同步FIFO的兩種Verilog設計方法] (https://blog.csdn.net/wuzhikaidetb/article/details/121136040)
