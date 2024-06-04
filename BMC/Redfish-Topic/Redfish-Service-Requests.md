<h2>
    Redfish Service Requests
</h2>

Redfish Services 和 Clients 在處理 HTTP 請求時，需要依照 HTTP 1.1 規格來處理請求頭

[7.1. Request headers table](<https://www.dmtf.org/sites/default/files/standards/documents/DSP0266_1.20.1.html#request-headers:~:text=to%20Redfish%20services.-,7.1%20Request%20headers,-Table%206%20lists>)

<h3>
    For Redfish Services
</h3>

如果 Service Requirement 為
* Yes
    * 服務端必須依照 HTTP 1.1 規格**處理**這些請求頭
* Conditional
    * 需要在描述列中註明的條件下**處理**這些請求頭
* No
    * 服務端應依照 HTTP 1.1 規範**處理**這些請求頭，但不是必須的


<h3>
    For Redfish Clients (sending the HTTP requests)
</h3>

如果 Client Requirement 為
* Yes
    * 服務端必須依照 HTTP 1.1 規格**包含**這些請求頭
* Conditional
    * 需要在描述列中註明的條件下**包含**這些請求頭
* No
    * 服務端應依照 HTTP 1.1 規範**包含**這些請求頭，但不是必須的

<h3>
    GET (read requests)
</h3>

GET 操作用於從 Redfish 服務中檢索資源。

客戶端向單個資源的 URI 發送 GET 請求。

1. 不會改變資源的狀態或服務的行為
2. 服務應忽略 GET 請求中的正文內容
3. 多次執行相同的 GET 請求應返回相同的結果
（除非資源在此期間發生了外部變更）

<h3>
    HEAD
</h3>

HEAD 方法 與 GET方法 的不同之處在於**它不會傳回 Message body**。

HEAD 方法完成相同的授權檢查，並在 HTTP 標頭中返回與 GET 方法相同的 meta information 和 status codes。

Services may support the HEAD method to:
* Return meta information in the form of HTTP response headers.
* Verify hyperlink validity

<h3>
    Data modification requests
</h3>

若要建立、修改和刪除資源，用戶端發出以下操作：
* **POST** (create)
* **PATCH** (update)
* **PUT** (replace)
* **DELETE** (delete)
* **POST** (action) on the resource

<h3>
    PATCH (update)
</h3>

若要**更新資源的屬性**，服務應支援 PATCH 方法。

* PATCH請求：用來部分更新資源，只會更改請求正文中指定的屬性。
* 請求正文：定義了要更新的屬性，服務應忽略OData註解。
* 空JSON對象：表示不對資源做任何更改。
* ETag保護：使用If-Match或If-None-Match標頭來實現。
* 拒絕更新：服務可以根據策略拒絕更新，並返回相應的HTTP狀態碼。

再沒有外部變更的情況下, 重複的 PATCH requests 不會改變資源的最終狀態

但是就算資源內容不變, **ETag 值可能會在每次PATCH操作後發生變化**，這反映了資源的更新狀態

<h4>
    PATCH on array properties
</h4>

資源中的陣列屬性有三種可能的樣式
* 固定長度 (**Fixed Length**) 和 可變長度 (**Variable Length**)

    * 接受null來移除元素，並接受空對象{}來保持元素不變
* 剛性風格 (**Rigid Style**)

    * 移除的元素將被null元素取代，這樣保持陣列的大小不變

處理 PATCH 請求時，操作順序應為：
1. Modifications
2. Deletions
3. Additions

**Example :**

給定一個固定長度的陣列 `Flavors`，支持最多六個元素（以null元素填充），其中有四個元素已經填充：
```
"Flavors": [ "Chocolate", "Vanilla", "Mango", "Strawberry", null, null ]
```

客戶端可以發出如下的PATCH請求來移除 `Vanilla`，用 `Cherry` 替換 `Strawberry` ，並添加 `Coffee` 和 `Banana` 到陣列，同時保持其他元素不變：
```
"Flavors": [ {}, null, {}, "Cherry", "Coffee", "Banana" ]
這邊的 1, 3 {} 是 Chocolate 和 Mango
```

處理此 PATCH 請求後，結果陣列應該是：
```
"Flavors": [ "Chocolate", "Mango", "Cherry", "Coffee", "Banana", null ]
```

<h3>
    PUT (replace)
</h3>

PUT 方法用於完全替換一個資源

服務器**可以在響應中添加客戶端省略的屬性**，以滿足資源定義所需或服務器通常提供的屬性

在不受外部變更影響的情況下，PUT 操作應該是幂等的。但可能會改變 ETag 值。

當替換操作成功時，響應可以包含替換後的資源表示。參見修改成功的響應

PUT 方法的例外情況包括服務**未實現此方法時**返回 HTTP 405 狀態碼，服務可以拒絕不包括資源定義（架構）所需屬性的請求，以及客戶端對資源集合進行 PUT 請求時返回 HTTP 405 狀態碼。

<h3>
    POST (create)
</h3>

若要建立新資源，服務應支援資源集合上的 POST 方法。

* 服務應在對新資源的 URI 的回應中設定 Location 標頭。
* **不是幂等操作**： POST 操作不是幂等操作，即重複執行同一個 POST 請求不會產生相同的結果。

向資源集合提交 POST 請求相當於向該資源集合的 Members 屬性提交相同的請求。

For example, let's say the SessionCollection is found at `/redfish/v1/SessionService/Sessions`. 

A client can POST to either `/redfish/v1/SessionService/Sessions` or `/redfish/v1/SessionService/Sessions/Members`, and **the effect would be the same**.

![image](https://hackmd.io/_uploads/BygVm82NR.png)

<h3>
    DELETE (delete)
</h3>

若要刪除資源，服務應支援 DELETE 方法。

當刪除操作成功時，回應可能會包含刪除後的資源表示。

如果該資源**永遠不能被刪除**，服務應返回 HTTP 405 狀態碼。

如果該資源**已經被刪除**，服務可以返回 HTTP 狀態碼 404 或成功的代碼。

服務可以允許在請求主體中包含 `@Redfish.OperationApplyTime` 屬性。

<h3>
    POST (Action)
</h3>

服務應支援 POST 方法將操作傳送至資源。

The POST operation may not be idempotent.

服務可能允許在請求正文中包含 `@Redfish.OperationApplyTime` 屬性。

資源的 Actions 屬性中的 target 屬性應包含操作的 URI。

操作的 URI 應採用以下格式：
```
ResourceUri/Actions/QualifiedActionName
```

>QualifiedActionName：
>The qualified name of the action. Includes the namespace.

如果 Action 不需要接受任何參數, 他必須要可以接受 {}

<h3>
    Operation apply time
</h3>

服務可以接受 POST（建立）、DELETE（刪除）或 POST（操作）請求正文中的 `@Redfish.OperationApplyTime` 註解。

此註釋使客戶端能夠控制何時執行操作。

<h3>
    ETag
</h3>

ChatGPT : 

![image](https://hackmd.io/_uploads/Hy8z4H3VR.png)
