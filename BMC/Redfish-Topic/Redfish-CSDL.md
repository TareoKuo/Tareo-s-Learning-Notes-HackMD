![image](https://hackmd.io/_uploads/HyJ3cN0X0.png)

<h2>
    CSDL
</h2>

CSDL 文件可以根據標記先分為兩個部分，**Referencing other CSDL files** 和  **CSDL data services**

![image](https://hackmd.io/_uploads/HkEqo82V0.png)

<h2>
    Standard OData Annotations Used by Redfish
</h2>

Core annotations
* **OData.Description**: Informative documentation
* **OData.LongDescription**: Normative documentation
* **OData.Permissions**: Client’s ability to read/write a property
* **OData.AdditionalProperties**: 如果實作可以新增更多屬性
* **OData.AutoExpandReferences**: 擴充導航屬性參考
* **OData.AutoExpand**: 即使沒有請求，服務也會擴展實體

Measures annotations
* Measures.Unit: 記錄值的單位；使用 UCUM 表示法

Capabilities annotations
* **Capabilities.InsertRestrictions**: If a client can **add** to a collection
* **Capabilities.UpdateRestrictions**: If a client can **modify** a resource
* **Capabilities.DeleteRestrictions**: If a client can **remove** from a collection

<h2>
    Base Redfish Schemas
</h2>

RedfishExtensions_v1.xml
* 定義 Redfish 特定註釋

Resource_v1.xml
* Defines base class for all resources
    * Resource and ResourceCollection
* Defines common properties and types
    * **Id**, **Name**, **Description**, **Oem**
    * **Links** object base definition
    * **Status** object definition
    * **Location** object definition
    * Common enumerated lists

<h2>
    Defining Redfish Resources
</h2>

Resources are singular entities, such as Thermal

All Resources inherit from `Resource.v1_0_0.Resource`

Id is used as the key property

Name is mandatory, but Description is optional

許多資源定義了一個 Links 屬性，該屬性繼承自資源的 Links 定義
* Links 是一個僅包含 navigation properties 的對象
* 這些連結是相關資源的參考

通用 Oem 物件可供實現根據需要擴展模式

<h2>
    Defining Redfish Resource Collections
</h2>

Resource Collections 包含一組相同類型的資源, 例如包含不同 Chassis Resources 資源的 Chassis Collection

所有資源集合繼承自 `Resource.v1_0_0.ResourceCollection`

Name is used as the key property

Description is optional

所有 Resource Collections 都將 Members 屬性定義為它們列出的資源類型的陣列

<h2>
    Redfish Schema Versioning
</h2>

所有 Resources 都有特定的命名約定及 namespaces
* 第一個命名空間是「 unversioned 」且不包含任何屬性
* 後續 Namespace 是有版本的，並且相互繼承

Namespaces are named as:
* Unversioned: ResourceName
* Versioned: ResourceName.vX_Y_Z
    * X is the major version
    * Y is the minor version
    * Z is the errata version

**新增屬性**將產生具有新次要版本的 New namespace

**修正現有檔案**將產生具有 **New errata version** 的 New namespace

<h2>
    Relationship to JSON Schema
</h2>

CSDL schemas 用於透過腳本產生 JSON schemas

Different number of files
* CSDL：每個資源類型一個檔案
    * Manager_v1.xml

* JSON：每種資源類型每個版本一個檔案
    * Manager.json
    * Manager.v1_0_0.json
    * Manager.v1_0_1.json
    * Other...

**CSDL 和 JSON schemas 在功能上是等效的**

<h2>
    CSDL
</h2>

<h3>
    How to define Array
</h3>

```xml
<Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="MyNamespace">
  <EntityType Name="MyEntityType">
    <Property Name="MyArrayProperty" Type="Collection(Edm.String)" />
  </EntityType>
</Schema>
```

`Type="Collection(Edm.String)"`：屬性類型為 `Collection(Edm.String)`，表示這是一個字串陣列。 

**Edm.String** 表示基本類型為字串，**`Collection()`表示這是一個陣列（集合）。**

---

**使用自訂類型定義 Array**

```xml
<Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="MyNamespace">
  <ComplexType Name="MyComplexType">
    <Property Name="Property1" Type="Edm.String" />
    <Property Name="Property2" Type="Edm.Int32" />
  </ComplexType>

  <EntityType Name="MyEntityType">
    <Property Name="MyComplexArray" Type="Collection(MyNamespace.MyComplexType)" />
  </EntityType>
</Schema>
```

**ComplexType**：
* `Name="MyComplexType"`：定義一個 Complex Type，其名稱為 `MyComplexType`。
* 包含兩個屬性：**Property1**（字串類型）和 **Property2**（整數類型）。


**EntityType：**
* `Name="MyEntityType"`：定義一個 Entity Type，其名稱為 `MyEntityType`。
* 包含一個屬性：`MyComplexArray`，其類型為 `Collection(MyNamespace.MyComplexType)`，表示這是一個包含 `MyComplexType` 類型元素的陣列。

---

**Redfish 實際例子 (ChatGpt)**

```xml
<Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="Redfish.Example">
  <EntityType Name="ComputerSystem">
    <Property Name="Processors" Type="Collection(Redfish.Example.Processor)" />
  </EntityType>

  <ComplexType Name="Processor">
    <Property Name="Id" Type="Edm.String" />
    <Property Name="Model" Type="Edm.String" />
    <Property Name="SpeedMHz" Type="Edm.Int32" />
  </ComplexType>
</Schema>

```