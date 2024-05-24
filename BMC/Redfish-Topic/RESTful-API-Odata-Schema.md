<h2>
    RESTful API
</h2>

[關於 RESTful API](<https://aws.amazon.com/tw/what-is/restful-api/>)

RESTful API 是一種能讓兩個電腦系統用來安全地透過網際網路交換資訊的介面。

大多數商業應用程式必須與其它內部和第三方應用程式溝通，以執行各種任務。

>舉例來說，為了產生每月薪資單，您的內部帳戶系統必須與您客戶的銀行系統共用資料，以自動化提供薪資單的過程，並且與內部出勤卡應用程式進行通訊。
>
>多種 RESTful API 支援此類資訊交換，因為它們遵守安全、可靠、且有效率的通訊標準。

它的約束（constraints）有以下幾項：

* 主從架構 `Client-server`
* 無狀態 `Statelessness`
* 可暫存 `Cacheable`
* 統一介面 `Uniform interface`
* 階層式系統 `Layered systems`： 每一個資源有對應至少一個的URI
* [Optional] 依需求提供程式 Code on demand

<h3>
    REST
</h3>

Representational State Transfer (REST) 是一種軟體架構，它對 API 的運作方式施加了條件。

---

<h2>
    Odata
</h2>

OData（The Open Data Protocol） 是一種基於REST的數據訪問方式，該標準由微軟發起，前三個版本1.0、2.0、3.0是微軟開放標準，第4.0版於2014年在OASIS投票通過成為 開放工業標準。

<h3>
    Design Principles of Odata
</h3>

1. 傾向機器能儲存各種資料來源
2. 服務能支援拓展性的功能
3. 以REST設計原則為導向
4. 容易地建立兼容的服務
5. 保持簡單

OData允許不同的客戶端能存取不同的資料來源。

![image](https://hackmd.io/_uploads/ryw6f-oXR.png)

<h3>
    Annotation attributes
</h3>

**OData註解**
* @odata.id：包含資源的URL
* @odata.type：用於尋找定義此資源的模式


**DMTF註解**
* @DMTF.Settings：表示資源設定

**還有種常見的註釋 (@odata.context)**
這是通用 OData v4 客戶端的專用註釋
* 提供描述有效負載的元資料位置
* 提供用於解析相對引用的根 URL
>**@odata.context** 的結構為帶有資料(通常根路徑為頂層單一實例或集合)描述片段的元資料文檔URL

<h3>
    Public attributes
</h3>

“Name” 和 “Id” 是必要屬性。

您還會看到嵌入式物件 “Status”，它在所有使用中都有相同的定義。

實際上，這些屬性屬於 Schema 的公共部分，可以被其他Schema引用。

公共屬性透過模式中的OData 「引用」元素被各資源的模式引用，確保各處使用的定義一致。

這類屬性包括：
* “**Actions**”，告知客户可以调用哪些操作。
* “**Oem**”，具體廠商針對標準資源定義的擴展。

---

<h2>
    Schema
</h2>

Redfish Schema 是用來**描述**和**定義** Redfish API 的規格和結構。 

Redfish 是一種基於標準化 RESTful API 的開放式標準，旨在簡化和標準化伺服器和資料中心管理。 

Redfish Schema 提供了用於**建立和存取 Redfish API** 的標準化方法，使得不同廠商和實作都能夠以統一的方式與 Redfish 相容。

**Redfish Schema 以兩種格式定義：**
* OData-Schema 格式 
    * OData Schema 格式(CSDL)定義，是為了方便通用 OData 工具和應用程式解析
* JSON Schema 格式
    * JSON Schema 格式定義，是為了應用於其他環境，如 Python 腳本、 JavaScript 程式碼和視覺化