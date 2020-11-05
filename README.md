Project Name: BIG DATA  
======================  
  
  
Target Release: 1.1  
  
  
Version Number:           1.1	  
Version Date:		2019/12/19  
Author:			Mack.Chuang	  
Total pages:	  
  
* * *  
  
Contents  
--------  
[TOC]  
  
* * *  
# 1. 文件資訊(History)  
## 1.1 文件制／修訂履歷(Table of Changes)  
### Modification History  
  
|    Data    |    Author   | Version | Description |  
|:----------:|:-----------:|:-------:|:-----------:|  
| 2019/02/26 | Mack Chuang |    1.0    |     初版    |  
| 2019/12/19 | Mack Chuang |    1.0    |     工作流說明    |  
  
## 1.2 文件存放位置(Document Location)  
路徑 :  
  
## 1.3 文件發佈資訊(Distribution)  
本文件提供以下的人員閱讀:    
>系統開發人員    
>維運人員    
>資訊部人員    
  
  
* * *  
# 2. 何謂Apache NiFi?   
    即時ETL工具，專為提供企業進行自動化系統間的資料流而構建的，由NSA[National Security Agency]貢獻給Apache基金會，  
    後來由Hortonworks提供軟體開發及支持。  
  
* * *  
# 3. 資料流設計痛點  
    + 系統故障  
    + 數據處理瓶頸  
    + 異常數據處理  
    + 業務快速演進  
    + 多系統升級不同步引入的前後相容  
    + 相容性及安全性  
    + 正式環境無痛升級  
  
## 3.1	NiFi基本單位  
|    NiFi 名詞    |    FBP 名詞   | 說明 |  
|:----------:|:-----------:|:-------:|  
|FlowFile|Information Packet|NiFi系統間傳輸的資料流單位|  
|FlowFile Processor|Black Box|NiFi處理器，負責定義資料流中的動作|  
| Connection | Bounded Buffer |NiFi處理器間的隊列定義|  
| Flow Controller | Scheduler |維護資料流之間連接和控制器設定|  
| Process Group | subnet |   資料流群組化，允許群組間的數據交換    |  
  
## 3.2 NiFi架構  
主要在OS主機系統內的JVM執行  
![NiFi_Architecture](https://i.imgur.com/pWm9Fy6.png)  
  
## 3.3 NiFi操作介面  
資料流設計一律透過Web UI進行規劃與設定。  
  
首先透過瀏覽器進入登入畫面URL資訊如下：  
> https://nifi.settour.com.tw/nifi/login  
  
與公司的AD綁定，輸入ERP的帳號密碼：  
>![NiFi登入頁](https://i.imgur.com/O32VgsO.png)  
  (沒畫面請聯繫NiFi管理員給予權限)  
  
### 3.3.1 用戶介面  
透過瀏覽器登入NiFi之後便可以開始設計資料流，畫面如下：  
>![NiFi介面](https://i.imgur.com/Ox7XiGn.png)  
![其他功能](https://i.imgur.com/9tc21Ul.png)  
  
大致分為五個部分，分別為：  
>Operate Palette：操作面板  
Status Bar：資料流狀態欄位  
Components Toolbar：物件工具列  
Global menu：全域目錄  
Search：利用搜尋查找案件名  
Navigate Palette：導航面板  
Bird's Eye View：瀏覽視圖  
Breadcrumbs：資料流程組路徑  
  
### 3.3.2 物件工具說明  
要建構資料流一律使用拖拉的方式，使用需要的組件組合成資料流。  
>![工具列](https://i.imgur.com/e2twhfB.png)  
  
>![處理器](https://i.imgur.com/3pf1thI.png)  
處理器：負責資料流所有的ETL操作，透過拖拉產生各種不同的處理器。  
  
>![處理器選項](https://i.imgur.com/4TwRKgN.png)  
處理器選單，對處理器點擊滑鼠右鍵，可以進行處理器的設定配置。  
  
>![處理器群組](https://i.imgur.com/vEwT7zx.png)  
處理器群組：可將物件進行邏輯分組，以閱讀跟維護資料流。 

### 3.3.3 NiFi本身參數說明
NiFi部份物件再產生資料/處理資料的同時會產生辨識值屬性(Attribute)

fragment.identifier : 來自同一結果集FlowFile會有相同的數值，用來關聯相同數據集

fragment.index : 來自同一結果集FlowFile內的索引位置

fragment.count : 來自同一結果集生成的FlowFile筆數

record.count : 來自同一結果集生成的資料筆數

### 3.3.4 NiFi正則表達式

URL : https://nifi.apache.org/docs/nifi-docs/html/expression-language-guide.html#minus

# 4. 工作流說明  
  
## 4.1 新增ELK資料  
> ELK流程圖  
![ELK](https://i.imgur.com/igLBSDv.png)    
  
使用到的NiFi物件如下所示  
>QueryDatabaseTableRecord : 連線到資料庫並執行SQL語法撈資料  
>ReplaceText : 使用正則表示式，針對撈出的資料做資料清理  
>PutElasticsearchHttp : 將資料新增至ElasticSearch  
  
每個元件的設定參數解說如下  
>QueryDatabaseTableRecord Configure 設定如下  
>Database Connection Pooling Service : 連線至資料庫的JDBC連線  
>Database Type : 哪種類型資料庫，通常為Generic  
>Table Name : 查詢的表格名稱，如果使用自定義SQL會變成查詢的別名  
>Columns to Return : 想要回傳哪些欄位，如果使用自定義SQL請為空  
>Additional WHERE clause : SQL查詢的條件，如果使用自定義SQL請為空  
>Custom Query : 自定義SQL，會將語法拼成子查詢的方式傳至目標資料庫查詢。  
>Record Writer : 查詢的資料要用什麼格式寫入至FlowFile (NiFi最小單位) 內  
  
>ReplaceText Configure 設定如下  
>Search Value : 正規表示式  
>Replacement Value : 如果正則匹配到置換的值  
>Replacement Strategy : 用哪種方式來做匹配查找  
>Evaluation Mode : 使用哪種方式來讀取資料，分成整個FlowFile讀進記憶體再處理或是每筆處理  
  
>PutElasticsearchHttp Configure 設定如下  
>Elasticsearch URL : Elasticsearch的網址  
>Index : 於Elasticsearch的哪個索引下新增資料  
>Type : 於Elasticsearch的索引下的資料序號名稱  
>Batch Size : 一次批次新增多少筆資料  
>Index Operation : 對Elasticsearch進行的CRUD動作  
  
## 4.2 發簡訊-Kettle  
>Kettle 轉 NiFi  
![Kettle 轉 NiFi](https://i.imgur.com/fph1Kox.png)  
  
使用到的NiFi物件有八個如下所示  
>QueryDatabaseTableRecord : 連線到資料庫並執行SQL語法撈資料  
>EvaluateJsonPath : 將FlowFile(NiFi最小單位)內的資料新增至Attribute Values中方便異動或將其他資訊寫入至FlowFile(NiFi最小單位)內  
>PutEmail : 將資料拼成郵件格式寄出  
>InferAvroSchema : 替Attribute Values定義Schema  
>UpdateAttribute : 新增資料到Attribute Values方便之後取用  
>UpdateRecord : 根據資料格式更新FlowFile內的資料  
>ConvertJSONToSQL : 將JSON資料格式轉換成SQL語法  
>PutSQL : 將SQL語法丟到資料庫去執行  
  
每個元件的設定參數解說如下  
>QueryDatabaseTableRecord Configure 設定如下  
>Database Connection Pooling Service : 連線至資料庫的JDBC連線  
>Database Type : 哪種類型資料庫，通常為Generic  
>Table Name : 查詢的表格名稱，如果使用自定義SQL會變成查詢的別名  
>Columns to Return : 想要回傳哪些欄位，如果使用自定義SQL請為空  
>Additional WHERE clause : SQL查詢的條件，如果使用自定義SQL請為空  
>Custom Query : 自定義SQL，會將語法拼成子查詢的方式傳至目標資料庫查詢。  
>Record Writer : 查詢的資料要用什麼格式寫入至FlowFile (NiFi最小單位) 內  
>Maximum-value Columns : 記錄指定欄位每次查詢的最大值，下次異動資料的值必須大於指定欄位才執行  
  
>EvaluateJsonPath Configure 設定如下  
>Destination : 要將新增的資料寫至FlowFile (NiFi最小單位)內或是將FlowFile (NiFi最小單位)內的資料寫至Attribute Values中方便處理  
>Return Type : 指定Destination要寫入至哪裡，通常建議使用auto-detect讓他自動判斷  
>右上角+號 : 自定義FlowFile (NiFi最小單位)內的哪些資料欄位寫入Attribute Values或將什麼樣的資料欄位寫回至FlowFile (NiFi最小單位)內  
  
>PutEmail Configure 設定如下  
>SMTP Hostname : MailServer位址  
>SMTP Port : MailServer阜號  
>Content Type : Mail內的郵件格式  
  
>InferAvroSchema Configure 設定如下  
>Schema Output Destination : 將Schema定義在Attribute Values中或將Schema寫回至FlowFile (NiFi最小單位)內會覆蓋掉本來FlowFile(NiFi最小單位)內的資料  
>Input Content Type : 傳入FlowFile(NiFi最小單位)的資料格式  
>Avro Record Name : 定義Schema的名稱  
  
>UpdateAttribute Configure 設定如下  
>右上角+號 : 自定義哪些數值寫入到Attribute Values中方便之後取用  
  
>UpdateRecord Configure 設定如下  
>Record Reader : FlowFile(NiFi最小單位)內的資料格式  
>Record Writer : 處理完輸出後FlowFile(NiFi最小單位)內的資料格式  
>Replacement Value Strategy : 指定使用何種方式替換FlowFile內的值  
>右上角+號 : 自定義哪些欄位以及數值替換掉FlowFile(NiFi最小單位)內的欄位資料  
  
>ConvertJSONToSQL Configure 設定如下  
>JDBC Connection Pool : JDBC連線設定  
>Statement Type : 指定要做哪些SQL類型的DML動作  
>Table Name : 要執行SQL類型中DML語言的表格名稱  
>Update Keys: 執行哪些欄位更新的判斷條件  
  
>PutSQL Configure 設定如下  
>JDBC Connection Pool : JDBC連線設定  
>Batch Size : FlowFile(NiFi最小單位)數量達到指定數值才執行批次更新  
  
## 4.3 poi更新  
>於ERP修改POI/國外團體行程觸發API  
![poi更新](https://i.imgur.com/2tHZV4y.png)    
  
使用到的NiFi物件有四個如下所示  
>ListenHTTP : 監聽指定通訊阜  
>RouteOnContent : 根據正則表示式將FlowFile(NiFi最小單位)內符合判斷的資料路由到不同的處理器  
>UpdateAttribute : 新增資料到Attribute Values方便之後取用  
>ExecuteStreamCommand : 將FlowFile(NiFi最小單位)傳入至自刻的程式碼中做運算在輸出  
  
每個元件的設定參數解說如下  
>ListenHTTP Configure 設定如下  
>Listening Port : 監聽哪個通訊阜  
  
>RouteOnContent Configure 設定如下  
>Match Requirement : 內容部分吻合正則表示或全部吻合正則表示才匹配成功  
>右上角+號 : 定義哪些標籤用哪種正則表示式，之後要用於路由  
  
>UpdateAttribute Configure 設定如下  
>什麼事情都不做，開發者認爲這樣比較美  
  
>ExecuteStreamCommand Configure 設定如下  
>Command Path : 自刻程式碼擺放的位置


>RouteOnAttribute 設定如下
相同結果集flowfile檔案數，當檔案的索引值與檔案數相同時走自定義的隊列[Queued]
>finish : \${fragment.count:minus(1):equals(${fragment.index})}

# 5. NiFi Upgrade  
  https://settour01.atlassian.net/wiki/spaces/DataAnalysis/pages/595198035/Apache+NiFi+Upgrade
