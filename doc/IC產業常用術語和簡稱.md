# **IC產業常用術語**

(S)IP: (Silicon) Intellectual Property

就是智慧財產權那個IP沒錯，半導體業界裡也習慣簡稱IP，意指Silicon IP，叫做矽智財；IP到底是什麼？最常見的解釋就是樂高積木譬喻法，把設計IC想成用樂高積木蓋房子，會有一塊基地，上面要用各種尺寸大小的樂高積木來蓋，用幾個黃色的先組好一些柱子，用幾個咖啡色的組好幾片屋頂，再把他們相互拼接，IP就像那些預先組好的柱子或是屋頂，是設計好並經過驗證的一些功能組塊，在設計IC時可以直接把這些現有組塊拿來用，不用再從頭開始重新設計電路。

EDA: Electronic Design Automation

設計IC用的各種電腦軟體，由三巨頭Synopsys、Cadence、Siemens把持主要市場(原本是Mentor Graphics，2016已被Siemens收購)，高階製程幾乎壟斷，在整段設計流程中各有各的強項，各別流程中使用的工具也差不多是贏者全拿，比如做邏輯合成的幾乎都用Synopsys的Design Compiler，驗證DRC和LVS的都是用Siemens的Calibre。

## **DRC: Design Rule Check**

驗證畫出來的IC layout符合各製程下的設計規範，最基本的三種類型：最小線寬(minimum width)、最小線距(minimum spacing)、最小覆蓋面積(minimum enclosure)；稍微用蓋房子想像，各種大小不一的木條搭成結構體，最細的木條不能太小以免強度不夠，木條間隔若太寬也可能無法承重，木條互相卡榫的地方要有一定的接觸面積。最先遇到的術語之一，可能因為一進公司就剛好在趕tapeout，整天都在問DRC剩多少，DRC clean是IC要可以下線生產的最基本底線，實在改不了的地方要請晶圓廠同意接受(waive)。

## **LVS: Layout Versus Schematic**

驗證畫出來的IC layout和原始的電路(schematic)設計之間的差異，畫完的layout經工具轉成netlist，拿來和當初原始設計的電路圖的netlist比較，兩者須一致(功能等價)即滿足驗證。最先遇到的術語之一，LVS clean後開始進行DRC clean，LVS clean也是IC要可以下線生產的最基本底線。

## **PDK: Process Design Kit**

就是晶圓廠打包好的工具包，設計IC最終要讓它可以被生產製造出來，所以是由晶圓廠把製程相關的知識，包括設計規則文件、電學規則文件、光罩各層定義文件、SPICE模型…等等，讓工程師在設計過程中更順利；比如我有學過水電知識，但我要去別人家修理時可能還是會忘記帶這個帶那個，若公司會幫我配置好工具包，我就是帶這一包出門所有問題都可以解決。

## **EC: Equivalence Check**

等價性檢查，用數學模型證明以兩種不同形式呈現的設計表現出一樣的行為，通常是指synthesis過後的netlist和源頭RTL source code比較，但在整個IC design flow中，每一層級經過處理後的輸出產物都要回去和上一層級做比較驗證，也都是相似的概念。

## **SDC: Synopsys Design Constraints**

是一種檔案格式，內容定義對電路的時序、面積、功耗的限制，簡單理解它就是前端的電路設計工程師替這個電路規範的一些限制，後端在進行實際的physical implementation的時候要遵守，很粗糙的舉例比如：訊號從這裡進入到出來不能超過0.000000000000000001秒，那可能走線就不能走這麼長，線要畫短一點。

## **SPICE model: Simulation Program with Integrated Circuit Emphasis**

是用來模擬電子電路內的類比訊號的軟體，由晶圓廠(FAB)提供給設計端，因為是晶圓廠透過蒐集實際製程做出來的silicon data，抽取裡面各器件(比如電晶體、電容、電阻...等)的參數，形成對應的數學模型，讓設計端可以透過SPICE model來模擬自己設計的電路表現行為是不是符合預期，也就是工程師設計了一個電路，把SPICE model套進去模擬，看看電路跑出來的結果(波形什麼的)是不是跟預想的一樣。

## **SPEF: Standard Parasitic Exchange Format**

由工具產生，是紀錄接線的RC值。只要講到RC這兩個字就只能是電阻Resistance和電容Capacitance，不是recycle，畫完的layout圖可以經由工具把裡面的器件及接線相互影響產生的效應：比如信號延遲、雜訊等，計算轉換成RC值抽取出來(SPEF)，建立數學模型，以便進行後續的時序、功耗、信號完整性等等分析，看是不是符合預期。

## **SI/PI: Signal Integrity/Power Integrity**

中文翻譯就很好理解：訊號完整性/電源完整性(我本來不知道integrity還有這個含義呢~)，電子電路設計就是都在講signal/power/ground，到底要輸出輸入什麼就是訊號signal，那也要給它動力power讓它可以運行，然後接地線ground意指相對的低電位，作為基準當參考點；做SI/PI模擬就是在對設計的電路進行各種分析，常常討論的問題主要有串擾(crosstalk，英文比較好理解，想成兩條信號線彼此互相講話干擾)、訊號損失(loss)、抖動(jitter)、雜訊(noise)、電磁干擾(EMI)等；在整個IC design flow中都會一直做SI/PI模擬分析，甚至到封裝、電路板設計都是很重要的參考數據。