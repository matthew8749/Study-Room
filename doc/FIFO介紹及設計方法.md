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

- 儲存陣列通常使用 RAM (Random Access Memory) 實現。 根據 FIFO 的深度和寬度，RAM 可以是單端口 (single-port) 或雙端口 (dual-port)。 雙端口 RAM 允許同時進行讀寫操作，可以提高 FIFO 的性能。
- RAM 的選擇會影響 FIFO 的性能和複雜度。

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

- 重要參數  
`深度`、`寬度`、`空白標誌`、`滿標誌`、`讀取時鐘`、`讀時針`、`寫入時鐘`和`寫入時針`

- 深度和寬度  
`位寬 WIDTH`：表示FIFO裡面每個資料的大小  
`深度 DEPTH`：表示FIFO能存多少個數據

- 讀取時鐘 and 寫入時鐘  
在同步FIFO中這兩個是相同的
clk dege來時，會讀取和寫入data


- 讀時針(rd_ptr) and 寫入時針(wr_ptr)  
  - 指向下一個即將要讀取或寫入的位置。讀寫操作的執行前提是讀寫使能信號 (rd_en, wr_en) 有效，並且 FIFO 沒有滿 (對於寫入) 或空 (對於讀取)。 在每次成功的讀寫操作 之後，相應的指針會 遞增，指向下一個可用的位置。  
  - 它們是用來指示RAM位址的索引，透過指標值的比較可以判斷FIFO的空滿狀態。

- 滿/空標誌邏輯:  
    - **滿:** 當寫入指針追上讀取指針時，FIFO 為滿。 更精確地說，如果下一次寫入將使 wr_ptr 等於 rd_ptr，並且當前正在執行寫入操作，則 FIFO 為滿。  

    - **空:** 當讀取指針追上寫入指針時，FIFO 為空。 更精確地說，如果下一次讀取將使 rd_ptr 等於 wr_ptr，並且當前正在執行讀取操作，則 FIFO 為空。

    - **初始狀態:** 當 wr_ptr 等於 rd_ptr 時，FIFO 初始狀態為空。  
    > 注意: 要考慮到重頭開始的狀況  
    > 意思是指 **繞回 (Wrap-around)** 的考慮: 當指針繞回時，簡單的比較 wr_ptr == rd_ptr 無法區分滿和空。 因此，通常使用額外的一位 (extra bit) 來擴展指針，以區分這些情況。   


- 定義電路介面
``` verilog
module sync_fifo
#(
  parameter                       ADR_BIT =  6,
  parameter                       DAT_BIT = 32,
  parameter                       WEN_BIT =  1
)(
  input  logic                    clk,
  input  logic                    rst_n,

  //當 cs_en 為高且 wr_en 為高時，表示一個寫入請求。
  //當 cs_en 為高且 wr_en 為低時，表示一個讀取請求。
  //當 cs_en 為低時，表示沒有任何操作請求。
  input  logic [WEN_BIT-1: 0]     cs_en,          // higi activ  
  input  logic [WEN_BIT-1: 0]     wr_en,          // higi activ
  input  logic [DAT_BIT-1: 0]     wr_dat,


  output logic [DAT_BIT-1: 0]     rd_dat,
  output logic                    fifo_full,
  output logic                    fifo_empty,
  output logic [ADR_BIT : 0]      fifo_count      // for testbench display
);
```

## 同步FIFO完整程式碼
```verilog
// +FHDR--------------------------------------------------------------------------------------------------------- //
// Project ____________                                                                                           //
// File name __________ sync_fifo.sv                                                                              //
// Creator ____________ Yan, Wei-Ting                                                                             //
// Built Date _________ MMM-DD-YYYY                                                                               //
// Function ___________                                                                                           //
// Hierarchy __________                                                                                           //
//   Parent ___________                                                                                           //
//   Children _________                                                                                           //
// Revision history ___ Date        Author            Description                                                 //
//                  ___                                                                                           //
// -FHDR--------------------------------------------------------------------------------------------------------- //
//+...........+...................+.............................................................................. //
//3...........15..................35............................................................................. //
`timescale 1ns/10ps

module sync_fifo
#(
  parameter                       ADR_BIT =  6,
  parameter                       DAT_BIT = 32,
  parameter                       WEN_BIT =  1
)(
  input  logic                    clk,
  input  logic                    rst_n,

  input  logic [WEN_BIT-1: 0]     cs_en,          // higi activ
  input  logic [WEN_BIT-1: 0]     wr_en,          // higi activ
  input  logic [DAT_BIT-1: 0]     wr_dat,


  output logic [DAT_BIT-1: 0]     rd_dat,
  output logic                    fifo_full,
  output logic                    fifo_empty,
  output logic [ADR_BIT : 0]      fifo_count      // for testbench display
);

// tag COMPONENTs and SIGNALs declaration --------------------------------------------------------------------------

  //
  logic    [ADR_BIT     : 0]      wr_ptr;
  logic    [ADR_BIT     : 0]      rd_ptr;

  // RAM 控制信號
  logic    [WEN_BIT-1   : 0]      ram_cen;
  logic    [WEN_BIT-1   : 0]      ram_wen;
  logic    [ADR_BIT-1   : 0]      ram_addr;
  logic    [DAT_BIT-1   : 0]      ram_wdata;
  logic    [DAT_BIT-1   : 0]      ram_rdata;

  // base on cs_en & wr_en
  logic                           rd_req;
  logic                           wr_req;
  logic                           wr_actual;
  logic                           rd_actual;

// tag OUTs assignment ---------------------------------------------------------------------------------------------
assign rd_dat      = ram_rdata;

// tag INs assignment ----------------------------------------------------------------------------------------------


// tag COMBINATIONAL LOGIC -----------------------------------------------------------------------------------------

//當 cs_en 為高且 wr_en 為高時，表示一個寫入請求。
//當 cs_en 為高且 wr_en 為低時，表示一個讀取請求。
//當 cs_en 為低時，表示沒有任何操作請求。
assign        rd_req              = (cs_en == 1'b1 && wr_en == 1'b0 );
assign        wr_req              = (cs_en == 1'b1 && wr_en == 1'b1 );

assign        wr_actual           = wr_req && !fifo_full;
assign        rd_actual           = !wr_actual && rd_req && !fifo_empty;

assign fifo_full   = ( wr_ptr[ADR_BIT] != rd_ptr[ADR_BIT]) && ( wr_ptr[ADR_BIT-1 : 0] == rd_ptr[ADR_BIT-1 : 0] );
assign fifo_empty  = ( wr_ptr == rd_ptr);
assign fifo_count  = wr_ptr - rd_ptr;                 // for testbench display

// tag COMBINATIONAL PROCESS ---------------------------------------------------------------------------------------
always_comb begin
  ram_cen                         = {WEN_BIT{1'b1}};
  ram_wen                         = {WEN_BIT{1'b1}};
  ram_addr                        = 'd0;
  ram_wdata                       = 'd0;

  if ( wr_actual ) begin
  ram_cen                         = {WEN_BIT{1'b0}};
  ram_wen                         = {WEN_BIT{1'b0}};
  ram_addr                        = wr_ptr[ADR_BIT-1:0];
  ram_wdata                       = wr_dat;
  end else if ( rd_actual ) begin
  ram_cen                         = {WEN_BIT{1'b0}};
  ram_wen                         = {WEN_BIT{1'b1}};
  ram_addr                        = rd_ptr[ADR_BIT-1:0];
  end

end

// tag SEQUENTIAL LOGIC --------------------------------------------------------------------------------------------
// ***********************/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**
//                       /**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/***
// *********************/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/****
always @ (posedge clk or negedge rst_n) begin
  if (~rst_n) begin

  end else begin

  end
end

 sim_ram_top #(
  .ADR_BIT ( ADR_BIT       ),
  .DAT_BIT ( DAT_BIT       ),
  .WEN_BIT ( WEN_BIT       )
) u_ram (
  .clk     ( clk       ),
  .rst_n   ( rst_n     ),
  .CEN     ( ram_cen   ),  //low activ
  .WEN     ( ram_wen   ),  //low activ
  .addr    ( ram_addr  ),
  .w_data  ( ram_wdata ),
  .r_data  ( ram_rdata )
);

// ***********************/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**
// 讀寫指針更新            /**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/***
// *********************/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/****
always_ff @ ( posedge clk or negedge rst_n) begin
  if ( !rst_n ) begin
    wr_ptr <= {ADR_BIT{1'b0}};
  end else begin
    if ( wr_actual ) begin
      wr_ptr <= wr_ptr + 1'b1;
    end else begin
      wr_ptr <= wr_ptr;
    end


  end
end

always_ff @ ( posedge clk or negedge rst_n) begin
  if ( !rst_n ) begin
    rd_ptr <= {ADR_BIT{1'b0}};
  end else begin
    if ( rd_actual ) begin
      rd_ptr <= rd_ptr + 1'b1;
    end else begin
      rd_ptr <= rd_ptr;
    end


  end
end
// ***********************/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**
//                       /**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/***
// *********************/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/****


endmodule
```

## 同步FIFO testbench程式碼
```verilog
// +FHDR--------------------------------------------------------------------------------------------------------- //
// Project ____________                                                                                           //
// File name __________ sim_sync_fifo.v                                                                              //
// Creator ____________ Yan, Wei-Ting                                                                             //
// Built Date _________ MMM-DD-YYYY                                                                               //
// Function ___________                                                                                           //
// Hierarchy __________                                                                                           //
//   Parent ___________                                                                                           //
//   Children _________                                                                                           //
// Revision history ___ Date        Author            Description                                                 //
//                  ___                                                                                           //
// -FHDR--------------------------------------------------------------------------------------------------------- //
//+...........+...................+.............................................................................. //
//3...........15..................35............................................................................. //
//`timescale 1ns/10ps

// test pattern
//1. 寫入測試：連續寫入數據直到 FIFO 滿
//2. 讀取測試：連續讀取直到 FIFO 空
//3. 讀寫交替測試


module sim_sync_fifo;

// tag COMPONENTs and SIGNALs declaration --------------------------------------------------------------------------
  //localparam STIM = "./vectors/stim_zero.txt";
  parameter                       ADR_BIT =  6;
  parameter                       DAT_BIT = 32;
  parameter                       WEN_BIT =  1;

  logic                           ref_clk_i;
  logic                           slow_clk_i;
  logic                           test_clk_i;
  logic                           rstn_glob_i;

// tag OUTs assignment ---------------------------------------------------------------------------------------------
// tag INs assignment ----------------------------------------------------------------------------------------------
// tag COMBINATIONAL LOGIC -----------------------------------------------------------------------------------------
// tag COMBINATIONAL PROCESS ---------------------------------------------------------------------------------------
// tag SEQUENTIAL LOGIC --------------------------------------------------------------------------------------------
// ***********************/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**
//                       /**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/***
// *********************/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/****
// clock gen
tb_clk_gen #(
    //.CLK_PERIOD(REF_CLK_PERIOD)
    .CLK_PERIOD(3.33)   // 5 --> 200MHz
  ) i_ref_clk_gen (
    .clk_o(ref_clk_i)
  );
// ***********************/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**
//                       /**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/***
// *********************/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/****

  //fufo input
  logic       [WEN_BIT-1   : 0]   cs_en;
  logic       [WEN_BIT-1   : 0]   wr_en;
  logic       [DAT_BIT-1   : 0]   wr_dat;
  logic       [DAT_BIT-1   : 0]   rd_dat;

  //fifo optput
  logic                           fifo_full;
  logic                           fifo_empty;
  logic       [ADR_BIT : 0]       fifo_count;

sync_fifo #(
  .ADR_BIT    (   ADR_BIT         ),
  .DAT_BIT    (   DAT_BIT         ),
  .WEN_BIT    (   WEN_BIT         )
) i_sync_fifo (
  .clk        (   ref_clk_i       ),
  .rst_n      (   rstn_glob_i     ),
  .cs_en      (   cs_en           ), // high activ
  .wr_en      (   wr_en           ), // high activ
  .wr_dat     (   wr_dat          ),
  .rd_dat     (   rd_dat          ),
  .fifo_full  (   fifo_full       ),
  .fifo_empty (   fifo_empty      ),
  .fifo_count (   fifo_count      )
);

// ***********************/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**\**\****/**/**
//                       /**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/****\**\**/**/***
// *********************/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/******\**\/**/****
initial begin
    $fsdbDumpfile("./sim_sync_fifo.fsdb");
    $fsdbDumpvars(0, sim_sync_fifo, "+all");
    $fsdbDumpMDA;
  end

initial begin
  rstn_glob_i   = 1'b0;
  #15
  rstn_glob_i   = 1'b1;
end


// test pattern
//1. 寫入測試：連續寫入數據直到 FIFO 滿
//2. 讀取測試：連續讀取直到 FIFO 空
//3. 讀寫交替測試

initial begin
  cs_en        = 1'b0;
  wr_en        = 1'b0;
  wr_dat       = 'd0;
  #20


  for (int i = 0; i <66 ; i++)begin
    @(posedge ref_clk_i);
    if ( ~fifo_full ) begin
      cs_en  = 1'b1;
      wr_en  = 1'b1;
      wr_dat = i;
      $display("Time=%0t, -----  FIFO Not Full at %0d", $time, wr_dat);
    end else begin
      wr_en  = 1'b0;
      wr_dat = 0;
      $display("Time=%0t, -----  FIFO full at %0d", $time, wr_dat);
      break;
    end
  end


  wr_en  = 1'b0;
  for (int i = 0; i <66 ; i++)begin
    @(posedge ref_clk_i);
    if ( ~fifo_empty ) begin
      cs_en  = 1'b1;
      //@(posedge ref_clk_i);
      $display("Time=%0t, -----  Read data: %0d", $time, rd_dat);
    end else begin
      $display("Time=%0t, -----  Read data: %0d", $time, rd_dat);
      $display("Time=%0t, -----  FIFO empty at %0d", $time, i);
      break;
    end
  end

  #30
  $finish;

end


  // 監控 FIFO 狀態
  initial begin
    forever begin
      @(posedge ref_clk_i);
      $display("Time=%0t, Full=%0d, Empty=%0d, Count=%0d",
               $time, fifo_full, fifo_empty, fifo_count);
    end
  end

endmodule
```


##
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
