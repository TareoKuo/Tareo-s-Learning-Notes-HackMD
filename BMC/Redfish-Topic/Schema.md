<h2>
    Schema definition languages
</h2>

![image](https://hackmd.io/_uploads/BJjwpvh7R.png)

* [DMTF Redfish Standard Library](<https://redfish.dmtf.org/schemas/>)
    * 可以從這邊看有哪些要 Reference 的

補充：
* `</Schema >` 中能使用 `</EntityType>` 和 `</ComplexType>` 標籤分別定義 **entity type** 和 **complex type** 元素。
* 標籤內可以用 `</Property>` 或是 `</NavigationProperty>`

<h2>
    OData Common Schema Definition Language
</h2>

OData **C**ommon **S**chema **D**efinition **L**anguage (CSDL) is an XML schema format defined by the OData CSDL specification. 

The following clause describes how Redfish uses CSDL to describe Resources and Resource Collections.

<h3>
    File naming conventions for CSDL
</h3>

Redfish CSDL 模式檔案應使用 TypeName 值命名，後面跟著「v」和模式的主要版本。

由於單一 CSDL 架構檔案包含架構的所有次要修訂，因此檔案名稱中僅使用主要版本。

檔案名稱的格式應為：
```
TypeName_vMajorVersion.xml
```

For example, version 1.3.0 of the Chassis schema would be named "`Chassis_v1.xml`".

<h3>
    Core CSDL files
</h3>

Resource_v1.xml 包含資源、資源集合和常見屬性（such as Status）的所有基本定義。

RedfishExtensions_v1.xml 包含所有 Redfish types and annotations.

<h3>
    CSDL format
</h3>

[Introduction to CSDL](<https://www.dmtf.org/sites/default/files/Redfish_School-Introduction_to_CSDL.pdf>)

OData 模式表示文件的外部元素應為 `Edmx` 元素，並且**應具有值為「4.0」的 Version 屬性**。

![image](https://hackmd.io/_uploads/By_mkuhX0.png)

<h3>
    EDMX（Entity Data Model XML）
</h3>

EDMX 是用於描述實體資料模型的 XML 格式。

它是 OData（Open Data Protocol）規範的一部分，用於定義資料模型和資料服務的元資料。

EDMX 檔案中一些關鍵且必須包含的元素和結構：
* edmx:Edmx
* edmx:Reference
* edmx:DataServices
* Schema
* EntityType

**Example :**
```xml=
<edmx:Edmx xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" Version="4.0">
    <edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/cs02/vocabularies/Org.OData.Core.V1.xml">
        <edmx:Include Namespace="Org.OData.Core.V1" Alias="Core"/>
    </edmx:Reference>
    <edmx:DataServices>
        <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="ExampleNamespace">
            <EntityType Name="Product">
                <Key>
                    <PropertyRef Name="ID"/>
                </Key>
                <Property Name="ID" Type="Edm.Int32" Nullable="false"/>
                <Property Name="Name" Type="Edm.String"/>
            </EntityType>
            <EntityContainer Name="ExampleContainer">
                <EntitySet Name="Products" EntityType="ExampleNamespace.Product"/>
            </EntityContainer>
        </Schema>
    </edmx:DataServices>
</edmx:Edmx>

```

<h3>
    Referencing other CSDL files
</h3>

類型定義通常會引用 OData 和 Redfish 命名空間來取得常見類型註釋術語。 

Redfish CSDL 檔案始終在以下命名空間上使用 Alias 屬性：
* `Org.OData.Core.V1` is aliased as **OData**.
* `Org.OData.Measures.V1` is aliased as **Measures**.
* `RedfishExtensions.v1_0_0` is aliased as **Redfish**.
* `Validation.v1_0_0` is aliased as **Validation**.

```xml=
<edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/cs01/vocabularies/Org.OData.Core.V1.xml">
  <edmx:Include Namespace="Org.OData.Core.V1" Alias="OData"/>
</edmx:Reference>
<edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/os/vocabularies/Org.OData.Measures.V1.xml">
  <edmx:Include Namespace="Org.OData.Measures.V1" Alias="Measures"/>
</edmx:Reference>
<edmx:Reference Uri="http://redfish.dmtf.org/schemas/v1/RedfishExtensions_v1.xml">
  <edmx:Include Namespace="RedfishExtensions.v1_0_0" Alias="Redfish"/>
  <edmx:Include Namespace="Validation.v1_0_0" Alias="Validation"/>
</edmx:Reference>
<edmx:Reference Uri="http://redfish.dmtf.org/schemas/v1/Resource_v1.xml">
  <edmx:Include Namespace="Resource"/>
  <edmx:Include Namespace="Resource.v1_0_0"/>
</edmx:Reference>
```

<h3>
    CSDL Data Services
</h3>

*結構、枚舉和其他定義在 CSDL 的命名空間內定義。*

命名空間是透過 Schema 標籤定義的，並使用 Namespace 屬性來宣告命名空間的名稱。 

Redfish 使用命名空間來區分不同版本的模式。 

CSDL 允許結構從其他結構繼承，這允許較新的命名空間僅定義附加定義。 

CSDL 命名空間的元素子句中進一步描述了此行為

Schema 元素是 `DataServices` 元素的子元素，而 `DataServices` 元素也是 Edmx 元素的子元素。

![image](https://hackmd.io/_uploads/BkHQndnQR.png)

<h3>
    Elements of CSDL namespaces
</h3>

<h4>
    Qualified names
</h4>

CSDL 中的許多定義都使用限定名稱的參考。 

CSDL 將其定義為「`Namespace.TypeName`」形式的字串，其中：
* *Namespace* 是命名空間的名稱
* *TypeName* 是命名空間中包含的元素的名稱

例如，如果引用 `MyType.v1_0_0.MyDefinition`，則表示可以在 `MyType.v1_0_0` 命名空間中找到具有名稱為 `MyDefinition` 的元素的定義。

<h4>
    Entity Type and Complex Type elements
</h4>

|   Elements   |    Tags     |
|:------------:|:-----------:|
| Entity Type  | EntityType  |
| Complex Type | ComplexType |

這是透過在 EntityType 或 ComplexType 標記內定義 Property 元素和 Navigation Property 元素來完成的。

所有 **Entity Type** 和 **Complex Type** 都包含**名稱屬性**，該屬性指定定義的名稱

**Entity Type** 和 **Complex Type** 可能具有 BaseType 屬性，該屬性指定限定名稱。

當使用 BaseType 屬性時，所引用的 BaseType 的所有定義都可供定義的 **Entity Type** 或 **Complex Type** 使用。

All Resources and Resource Collections are defined with the Entity Type element.

資源繼承自 **Resource.v1_0_0.Resource**，資源集合繼承自
**Resource.v1_0_0.ResourceCollection**。

**大多數結構化屬性都是使用複雜類型元素定義的**。

有些是使用繼承自 `Resource.v1_0_0.ReferenceableMember` 的實體類型元素定義的。

實體類型元素允許使用導航屬性元素進行引用，而複雜類型元素不允許這種用法。

Example **Entity Type** and **Complex Type element**:

![image](https://hackmd.io/_uploads/H1FkJK2XC.png)

<h4>
    Actions element
</h4>

Actions 應該要包含
* **一個 Name 屬性**，該屬性指定動作的名稱。
* 所有 Action 元素都包含 IsBound 屬性, 並且始終設定為 true。這用於指示該操作顯示為給定結構化類型的成員。

此元素用於定義可以對資源執行的操作。

此操作**應在有效負載中**表示為該操作的限定名稱，前面有「#」。

在 Redfish 中，這始終是以下複雜類型元素之一：
* 對於 Standard Actions，資源的操作複雜型別。
* 對於 OEM Actions，資源的 OemActions 複雜型別。

其餘的 `Parameter` 元素描述要傳遞給操作的附加參數。

需要在操作請求中提供包含術語 `Nullable="false"` 的參數。

![image](https://hackmd.io/_uploads/HkPQgKhm0.png)

**Action element for OEM actions**

![image](https://hackmd.io/_uploads/SkpIgFnXR.png)

<h4>
    Property element
</h4>

所有 Property 元素**都包含一個 Name 屬性**，該屬性指定屬性的名稱。

所有Property元素都包含一個 Type屬性指定資料類型。 
Type 屬性應為下列之一：
* 引用 Enum Type element 的限定名稱
* 引用 Complex Type element 的限定名稱
* 原始資料類型
* 使用集合術語的上述任何一個數組

原始資料類型應為以下類型之一：

![image](https://hackmd.io/_uploads/ryNNWKhmR.png)

屬性元素可以指定 Nullable 屬性。

如果值為 false，則不允許 null 值作為該屬性的值。

如果該屬性為 true 或未指定，則允許值為 null。

Example Property element:

![image](https://hackmd.io/_uploads/Hk5CZY2mR.png)

<h4>
     Navigation Property element
</h4>

他用來表示資源之間關聯的一個重要概念。

這些屬性可以用來導航到其他資源，從而形成一個可探索的 API 結構

使用 `Navigation Property element` 定義的 `Reference properties`
* Resources
* <font color="#f00">**Resource Collections**</font>
* Structured properties

NavigationProperty Tags
* **用於定義** **Entity Type** 和 **Complex Type** elements 內部的導航屬性元素。

Navigation Property elements 需要包含
* 一個名稱屬性，該屬性指定屬性的名稱。
* 指定資料類型的 Type 屬性，Type 屬性是 References an Entity Type element 的 Qualified name(限定名稱)。
* Nullable (可要可不要)

**除非要擴展引用屬性**，否則 Redfish 中的所有導航屬性都使用 `OData.AutoExpandReferences` Annotation 元素來表示參考始終可用

Example Navigation Property element:
```xml
<NavigationProperty Name="RelatedType" Type="MyTypes.TypeB">
    <Annotation Term="OData.Description" String="This property references a related resource."/>
    <Annotation Term="OData.LongDescription" String="This is the specification of the related property."/>
    <Annotation Term="OData.AutoExpandReferences"/>
</NavigationProperty>
```

<h4>
    Enum Type element
</h4>

Enum Type 元素是使用 EnumType 標籤定義的。

此元素用於定義一組 enumeration values，這些值可以應用於一個或多個屬性。

Enum Types 需要包含
* **Name** attribute
* Member elements, Member 包含一個 Name attribute, 該屬性指定成員名稱的字串值。

```xml
<EnumType Name="EnumTypeA">
  <Annotation Term="OData.Description" String="This is the EnumTypeA enumeration."/>
  <Annotation Term="OData.LongDescription" String="This is used to describe the EnumTypeA enumeration."/>
  <Member Name="MemberA">
    <Annotation Term="OData.Description" String="Description of MemberA"/>
  </Member>
  <Member Name="MemberB">
    <Annotation Term="OData.Description" String="Description of MemberB"/>
  </Member>
</EnumType>
```

<h4>
    Annotation element
</h4>

CSDL 中的註釋是使用 Annotation 元素來表達的。 

Annotation 元素可以應用於 CSDL 中的任何模式元素。

以下範例顯示了 Redfish 使用的每個不同模式註解如何在 CSDL 中表達。

* 帶有 **OData** 前綴的術語在 OData 核心架構中定義。
* 帶有 **Measures** 前綴的術語在 OData Measures Schema 中定義。
* 帶有 **Redfish** 前綴的術語在 RedfishExtensions Schema 中定義。

**Example Description annotation:**

![image](https://hackmd.io/_uploads/BkyHQY2Q0.png)


**Example Long Description annotation:**

![image](https://hackmd.io/_uploads/H1LE7t3X0.png)

**Example Additional Properties annotation:**

![image](https://hackmd.io/_uploads/SkbImFhQC.png)

**Example Permissions annotation (Read Only):**

![image](https://hackmd.io/_uploads/SkIvXF2Q0.png)

**Example Permissions annotation (Read/Write):**

![image](https://hackmd.io/_uploads/rkD_XK27C.png)

**Example Required annotation:**

![image](https://hackmd.io/_uploads/Sy5KmK2mC.png)

**Example Required on Create annotation:**

![image](https://hackmd.io/_uploads/HJRqXYnQC.png)

**Example Units of Measure annotation:**

![image](https://hackmd.io/_uploads/H1en7FnmR.png)

**Example Expanded Resource annotation:**

![image](https://hackmd.io/_uploads/H1g6QFnXC.png)

**Example Insert Capabilities Annotation (showing POST is not allowed):**

![image](https://hackmd.io/_uploads/Hkl0XKnXC.png)

**Example Update Capabilities Annotation (showing PATCH and PUT are allowed):**

![image](https://hackmd.io/_uploads/ryQ1VKh7C.png)

**Example Delete Capabilities Annotation (showing DELETE is allowed):**

![image](https://hackmd.io/_uploads/rkGl4t2mC.png)

**Example Resource URI Patterns Annotation:**

![image](https://hackmd.io/_uploads/Skxb4Fh7R.png)

**Example Owning Entity Annotation:**

![image](https://hackmd.io/_uploads/rJA-NK2QC.png)

<h2>
    JSON Schema
</h2>

<h3>
    File naming conventions for JSON Schema
</h3>

Versioned：
```
ResourceTypeName.vMajorVersion_MinorVersion_Errata.json
```
```
Chassis.v1_3_0.json
```
Unversioned (應使用 TypeName 命名)：
```
ResourceTypeName.json
```
```
Chassis.json
```


後續待寫