<h2>
    Data model
</h2>

Redfish 的關鍵原則之一是將 **Protocol** 與 **Data Model** 分開。

這種分離使得資料與傳輸和協定無關。

<h2>
    Common Properties
</h2>

在 Redfish API 中，資源是指一個獨立的、具體的實體。

Redfish 服務會以 JSON 格式傳回這些資源。

當 Redfish 服務傳回資源時，會在 HTTP 回應頭中指定 `Content-Type: application/json`，表示傳回的資料是 JSON 格式。

對單一資源的回應應包含下列屬性： 

**Common Properties**

|  Property   |                                              Description                                              |
|:-----------:|:-----------------------------------------------------------------------------------------------------:|
|  @odata.id  | Register resource are not requested to provide `@odata.id` <br> 該屬性的值是資源的 URI。 （Required） |
| @odata.type |                                       資源的類型。 （Required）                                       |
|     Id      |                          資源的 Id 屬性唯一標識包含該資源的資源集合中的資源                           |
|    Name     |                          Name 屬性用於傳達資源的人類可讀的名字 （Required）                           |

>**application/json** 是 **MIME** 類型, **JSON** 的官方類型是 **application/json**
>
>網路上每個資源都應該要有一個 **MIME** 類型, 讓瀏覽器知道才有法去處理他

**Status (Object property)**



| Property     |                                Description                                 |
| ------------ |:--------------------------------------------------------------------------:|
| Health       | 這表示在缺乏依賴資源的情況下該資源的健康狀態。如果值存在，則將顯示該屬性。 |
| HealthRollup | 這代表了從該資源的角度來看的整體健康狀況。<br>如果值存在，則將顯示該屬性。 |
| State        |                   這指示資源的已知狀態，例如是否已啟用。                   |


<h2>
    Resource Collections
</h2>

**Object property**

資源收集 Responses **應包含**以下屬性：



|      Property       |                     Description                      |
|:-------------------:|:----------------------------------------------------:|
|      @odata.id      |        該屬性的值是資源的 URI。 （Required）         |
|     @odata.type     |              資源的類型。 （Required）               |
|        Name         | Name 屬性用於傳達資源的人類可讀的名字。 （Required） |
|       Members       |           包含該集合的成員。 （Required）            |
| Members@odata.count |             集合中的項目數。（Required）             |


資源集合的 Responses **可能包含**以下屬性：

|        Property        | Description |
|:----------------------:|:-----------:|
|     @odata.content     |             |
|      @odata.etag       |             |
|      Description       |             |
| Members@odata.nextLink |             |
|          Oem           |             |

資源集合的 Responses 不應包含除有效負載註解之外的任何其他屬性。

<h2>
    Message object
</h2>

Message 物件提供有關物件、屬性或錯誤回應的附加資訊。

Message 物件是具有下列屬性的 JSON 物件：



|     Property      |           Type            | Required |                                         Defines                                          |
|:-----------------:|:-------------------------:|:--------:|:----------------------------------------------------------------------------------------:|
|     MessageId     |          String           |   Yes    |  錯誤或訊息，請勿與 HTTP 狀態碼混淆。<br>用戶端可以使用此程式碼從訊息註冊表存取詳細訊息  |
|      Message      |          String           |    No    |  人類可讀的錯誤訊息，指示與錯誤關聯的語意。<br>這應該是完整的訊息，而不是依賴替換變數。  |
| RelatedProperties | An array of JSON pointers |    No    |                              訊息描述的 JSON 負載中的屬性。                              |
|    MessageArgs    |    An array of string     |    No    | 訊息的替換參數值。<br>如果參數化訊息定義了 MessageId，則服務應在回應中包含 MessageArgs。 |
|     Severity      |          String           |    No    |                                       錯誤的嚴重性                                       |
|    Resolution     |          String           |    No    |                               解決錯誤所需採取的建議操作。                               |

Message 物件的每個實例應**至少包含一個** `MessageId` 以及任何適用的 `MessageArgs` 或定義完整的人類可讀錯誤訊息的 `Message` 屬性

The MessageId property value shall be in the format:
```
RegistryName.MajorVersion.MinorVersion.MessageKey
```

![image](https://hackmd.io/_uploads/BydPtD2Q0.png)

若要在訊息註冊表中搜尋對應的訊息，用戶端可以使用 `MessageId`。

<h2>
    Properties
</h2>

<h3>
    Resource identifier (<font color="#f00">@odata.id</font>) property
</h3>

Response 中的登錄資源**可能包含** `@odata.id` 屬性。

Response 中的所有其他資源**應包含** `@odata.id` 屬性。

標識符屬性的值應為資源 URI。

<h3>
    Resource type (<font color="#f00">@odata.type</font>) property
</h3>

Response 中的所有資源**應包含** `@odata.type` 類型屬性。

為了支援通用 OData 用戶端，回應中的所有 structured properties 都**應包含** @odata.type 類型屬性。

該值應為指定資源類型的 URL 片段，格式如下：
```
#Namespace.TypeName
```

| Variable  |                                      Description                                       |
| --------- |:--------------------------------------------------------------------------------------:|
| Namespace | 定義類型的 Redfish 模式的完整命名空間名稱。<br>對於 Redfish 資源，版本化命名空間名稱。 |
| TypeName  |                             The name of the Resource type                              |

資源類型值的範例：
```
#ComputerSystem.v1_0_0.ComputerSystem

omputerSystem.v1_0_0 表示 ComputerSystem 的版本 1.0.0 命名空間
類型本身就是 ComputerSystem。
```

<h3>
    Resource ETag (<font color="#f00">@odata.etag</font>) property
</h3>

ETag 使客戶端能夠有條件地檢索或更新資源。

資源應包含 `@odata.etag` 屬性。對於資源，該值應為 ETag。

<h4>
    Etag
</h4>

ETags（Entity Tags） 是 HTTP 協定中的一個機制，用於實作 Web 快取和條件請求。

ETag 是一個由伺服器產生的標識符，用於表示特定資源的特定版本。

它可以幫助客戶端和伺服器驗證資源是否發生了變化，從而優化網路頻寬和效能。

* Implementations should support the return of ETag properties for each resource.
* Implementations should support the return of ETag headers for each single-resource response.
* Implementations shall support the return of ETag headers for GET requests of `ManagerAccount` resources.

An ETag can be:
* A hash
* A generation ID
* A time stamp
* 當底層物件發生變化時，其他一些值也會發生變化

The format of the ETag header is:
```
ETag: "<string>"
```

<h3>
    Resource context (<font color="#f00">@odata.context</font>) property
</h3>

單一資源的 Responses 可能包含描述有效負載來源的 `@odata.context` 屬性。

如果存在 `@odata.context` 屬性，則根據 OData-Protocol，它應是描述資源的上下文 URL。

The context URL for a resource should be in the format:
```
/redfish/v1/$metadata#ResourceType
```

For example, the following context URL specifies that the results show a single ComputerSystem
Resource:
```
"@odata.context": "/redfish/v1/$metadata#ComputerSystem.ComputerSystem"
```

<h3>
    <font color="#f00">Id</font>
</h3>

資源的 Id 屬性唯一標識包含該資源的資源集合中的資源。 

Id 的值在資源集合中應該是**唯一的**。 

<h3>
    <font color="#f00">Name</font>
</h3>

Name 屬性用於傳達資源的人類可讀的名字。 

Name 屬性的類型應為**字串**。

Name 的值**不需要是唯一的**。 

<h3>
    <font color="#f00">MemberId</font>
</h3>

MemberId 屬性唯一標識數組中的元素，其中可以透過引用屬性來引用該元素。 

MemberId 的值在陣列中應**是唯一的**。 

<h3>
    Count (<font color="#f00">Members@odata.count</font>) property
</h3>

count 屬性定義資源集合中可用的資源或成員的總數。 

count 屬性應命名為 `Members@odata.count`，其值應為資源集合中可用成員的總數。 

$top 或 $skip 查詢參數不應影響此計數。

<h3>
    <font color="#f00">Members</font>
</h3>

資源集合的 Members 屬性標識集合的成員。 

Members 屬性是**必需的**，並且應在任何資源集合的回應中傳回。 

Members 屬性應是名為 Members 的 JSON 物件陣列。

會員財產**不得為空**。空集合應為空 JSON 陣列。

<h3>
    Next link (<font color="#f00">Members@odata.nextLink</font>) property
</h3>

Next Link 屬性的值應為資源的不透明 URL，具有相同的 `@odata.type`，其中包含來自原始操作的下一組部分成員。

只有當資源集合中的**成員數大於傳回的成員數**，且有效負載不代表所要求的資源集合的結尾時，才應出現下一個連結屬性。

<h3>
    <font color="#f00">Links</font>
</h3>

Links 屬性表示與資源關聯的超鏈接，如該資源的架構定義所定義。

為資源定義的所有關聯引用屬性應嵌套在 Links 屬性下。

為資源定義的所有直接（從屬）所引用的屬性應位於資源的根。

![image](https://hackmd.io/_uploads/SkqiqUnQ0.png)

![image](https://hackmd.io/_uploads/HyVpqUnXA.png)

<h3>
    <font color="#f00">Actions</font>
</h3>

每個支援的操作都表示為嵌套在操作下的屬性。

屬性名稱是使用識別操作的唯一 URI 建構的。

This property name shall be in the format : 
```
#ResourceType.ActionName
```


| Variable     |                Description                |
| ------------ |:-----------------------------------------:|
| ResourceType | The Resource where the action is defined. |
| ActionName   |          The name of the action.          |

用戶端可以使用此片段來識別引用的 Redfish Schema 文件中的操作定義。

The property for the action is a JSON object and contains the following properties : 
* <font color="#f00">target</font> 屬性**應存在**，並定義呼叫操作的相對或絕對 URL。
* title 屬性**可能存在**，並定義操作的名稱。

For example, the following property defines the Reset action for a ComputerSystem : 

![image](https://hackmd.io/_uploads/SJOxhL2X0.png)

有鑑於此，用戶端可以使用以下正文呼叫對 /redfish/v1/Systems/1/Actions/ ComputerSystem.Reset 的 POST 要求：

![image](https://hackmd.io/_uploads/ryHN3LnQC.png)

<h2>
    OEM
</h2>

<h3>
    OEM Property
</h3>

為了避免與其他 OEM 資源和未來 Redfish 標準資源發生 URI 衝突，Redfish 資源樹中 OEM 資源的 URI 應採用以下形式：
```
*BaseUri*/Oem/*OemName*/*ResourceName*

Example : 
/redfish/v1/AccountService/Oem/Contoso/AccountServiceMetrics
```

例如，如果 Contoso 定義了一個名為 ContosoAccountServiceMetrics 的新資源，透過在 URI `/redfish/v1/AccountService` 中找到的 Oem 屬性進行鏈接，則 OEM 資源將具有 URI `/redfish/v1/AccountService/Oem/Contoso/AccountServiceMetrics`


Example : 

![image](https://hackmd.io/_uploads/r1nHlD3XR.png)

<h3>
    OEM actions
</h3>

![image](https://hackmd.io/_uploads/HkStlP3mC.png)

目標屬性中 OEM 操作的 URI 應採用以下形式：
```
*ResourceUri*/Actions/Oem/*QualifiedActionName*
```

* ResourceUri 是支援呼叫操作的資源的 URI。
* `Actions` 是包含資源操作的屬性的名稱。
* `Oem` 是 `Actions` 屬性中 OEM 屬性的名稱。
* `QualifiedActionName` 是操作的限定名稱，包含命名空間。

<h2>
    Payload annotations
</h2>

Resources, objects within a Resource,和 Properties 可以包含附加註解作為屬性，其名稱格式如下：
```
[PropertyName]@Namespace.TermName
```



|   Variable   |                                      Description                                       |
|:------------:|:--------------------------------------------------------------------------------------:|
| PropertyName | 要註釋的屬性的名稱。<br>如果不存在，則註釋適用於整個 JSON 對象，該對象可能是整個資源。 |
|  Namespace   |                             定義註釋術語的命名空間的名稱。                             |
|   TermName   |                          應用於資源或資源屬性的註釋術語的名稱                          |

<h2>
    Allowable values
</h2>

要**指定參數的允許值**列表，用戶端可以對**屬性**或操**作參數**使用 `@Redfish.AllowableValues` 註解。

若要指定允許的值集，請包含具有屬性或參數名稱的屬性，後面接著 `@Redfish.AllowableValues`。

屬性值是一個 JSON 字串數組，用於定義參數的允許值。

<h2>
    Extended information
</h2>

Redfish Sepcification 有提供兩種方法進行 Extended information

* Extended object information
* Extended property information

<h3>
    Extended object information
</h3>

若要指定物件級狀態訊息，服務可以使用 `@Message.ExtendedInfo` 註解來註解 JSON 物件。

![image](https://hackmd.io/_uploads/rJlkSwhQ0.png)

<h3>
     Extended property information
</h3>

Service 可以使用 `@Message.ExtendedInfo`，並在前面加上屬性名稱，以使用擴充資訊來註解 JSON 物件中的單一屬性：

![image](https://hackmd.io/_uploads/SkWo8w3mA.png)

<h2>
    Action Info annotation
</h2>

Action Info annotation 用於傳達操作的參數要求和參數允許值。

這是透過在操作表示中使用 `@Redfish.ActionInfo` 術語來完成的。

This term contains a URI to the ActionInfo Resource.

帶有 `@Redfish.ActionInfo` annotation and
Resource 的 `#ComputerSystem.Reset` 操作範例：

![image](https://hackmd.io/_uploads/SJLEqPhmC.png)

ResetActionInfo 資源包含參數和支援的值的更詳細描述。

此資源遵循 ActionInfo 架構定義。

![image](https://hackmd.io/_uploads/HJ9Oqw2QC.png)

<h2>
    Settings Resource
</h2>

對於支援未來狀態和配置的資源，回應應包含帶有 `@Redfish.Settings` 註釋的屬性。使用設定註解時，應滿足以下條件：

* 使用 `@Redfish.Settings` 註解連結到目前資源的 Settings Resource 應具有相同的架構定義。
* Settings Resource 應該是可以更新的屬性的子集
* Settings Resource **不應包含** `@Redfish.Settings` 註釋
* Settings Resource **可能包含** @Redfish.SettingsApplyTime annotation

下面是支援設定資源的資源的範例主體。

用戶端可以使用 `SettingsObject` 屬性來找到設定資源的 URI。

![image](https://hackmd.io/_uploads/BJL13DhmR.png)

更新設定資源時，用戶端可以透過在請求正文中包含  `@Redfish.SettingsApplyTime` annotation 來表明其對何時套用未來配置的偏好。

* 如果服務支援配置何時套用未來設置，則 Settings Resource **應包含**帶有 `@Redfish.SettingsApplyTime` 註解的屬性。
* **只有設定資源應包含** `@Redfish.SettingsApplyTime` 註解。

下面是一個範例請求正文，顯示客戶端配置何時應用設定資源中的值。

在這種情況下，它要么處於重置狀態，要么處於指定的維護時段期間：

![image](https://hackmd.io/_uploads/ryoFnv3QA.png)
