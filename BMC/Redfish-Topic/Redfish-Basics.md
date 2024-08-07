<h2>
    Redfish Conformance Test Suite Guidance
</h2>

[Redfish Conformance Test Suite Guidance](<https://www.dmtf.org/sites/default/files/standards/documents/DSP-IS0018_1.0.0.pdf>)

<h2>
    Introduction to Redfish
</h2>

[Redfish Specification](<https://www.dmtf.org/sites/default/files/standards/documents/DSP0266_1.8.0.pdf>)
[Redfish White Paper](<https://www.dmtf.org/sites/default/files/DSP2044%20Redfish%20%E7%99%BD%E7%9A%AE%E4%B9%A6%201.0.0.pdf>)

用於 IT 基礎架構的行業標準 RESTful API

基於 Odata v4 的 JSON 格式的 HTTPS

應用程式、GUI 和腳本同樣可用

Schema-backed 但可讀性高

>Redfish API 提供資料中心資源管理、事件處理、長時間任務以及 發現等機制。

<h2>
    Redfish Architecture
</h2>

![image](https://hackmd.io/_uploads/S1xguSe4C.png)

Redfish 將 Protocol 和 Data Model 分開

Protocol -> Basic CRUD RESTful Protocol
* POST
* GET
* PUT (replace), PATCH (update)
* DELETE
* HEAD

Schema 有兩種
* CSDL (Common Schema Definition Language)
* JSON

<h2>
    Universal Resource Identifiers
</h2>

A Universal Resource Identifier (URI) identifies a resource, including the service root and all Redfish Resources.

For example, the `https://mgmt.vendor.com/redfish/v1/Systems/1` URI contains these component parts:

![image](https://hackmd.io/_uploads/ryfUpKom0.png)

<h2>
    HTTP methods
</h2>

![image](https://hackmd.io/_uploads/rkZDlqo70.png)

<h2>
    Redfish Version
</h2>

DMTF 將Redfish 協議（protocol）的定義與數據模型（data model）分開，同時允許獨立修改schema中定義的每個Resource，所以Redfish的版本可以分為下面**三種** : 

* Protocol 版本 (RedfishVersion: v1.xx)
* Schema 版本 (2022.01) (Redfish 的 mini world)
* 每個 ResourceType 版本 (AccoutService v1.10, ManagerService v2.xx...)

<h3>
    (1) Protocol Version
</h3>

* Protocol Version 指 Redfish Service 遵循 ***DSP0266*** 的版本

<font color="#005AB5">**DSP0266 Redfish Specification**</font> 主要內容是規範了 Redfish 需要符合哪些協定，例如OpenAPI， OData，Security 等，透過Redfish 指令（redfish/v1）可以得到 Redfish Version

<h3>
    (2) Schema Version 
</h3>

* Schema Version 指 Redfish Service 遵循 ***DSP0268*** 的版本

Redfish 將所有Resource 組成的mini world 稱作Schema

<h3>
    (3) ResourceType Version 
</h3>

* ResourceType version 需要額外註記，因為ResourceType  Version 很少完全符合 ***DSP0268*** 內註記的最新版本

![image](https://hackmd.io/_uploads/rknU8fhQC.png)

ResourceType 和版本可以從 `@odata.type` 獲得
```
@odata.type 格式：#<ResourceType>.<Version>.<TermName>

其中 <Version> 是 v<MajorVersion>_<MinorVersion>_<ErrataVersion>
```

例如：
```
@odata.type"： "#ComputerSystem.v1_8_0.ComputerSystem
``` 
可以知道 ComputerSystem 的版本是v1.8.0，透過查表得到他是在2019.2的時候被 released 的

![image](https://hackmd.io/_uploads/Hk1GDM27C.png)

<h2>
    Basic concepts
</h2>

每個 URL 代表一個資源。

一個資源可以是一項服務、一個集合、一個實體或其他結構。

但在 RESTful 術語中，URI 指向資源和與資源互動的用戶端。

所以，當您看到“**資源**”這個術語時，可以把它想像為訪問某個 URI 時返回的 內容。

<h2>
    Operations
</h2>

實做時要看 Redfish Specification 第七章 「[Service requests](<https://www.dmtf.org/sites/default/files/standards/documents/DSP0266_1.20.1.html#service-requests:~:text=see%20RFC3986.-,7%20Service%20requests,-This%20clause%20describes>)」


**GET** : (Common)
* 獲取，例如獲取系統帳戶訊息GET

**POST** : (Common)
* 用於建立資源或使用動作

**PATCH** : (Common)
* PATCH 用於改變一個或多個資源的屬性

**DELETE** : (Common)
* 用於刪除資源，但目前只有少數資源可刪除。

**PUT** : 
* 用於完全替換資源(只有少數資源可被完全替換)

**HEAD** : 
* HEAD 與 GET 類似，只是不傳回主體數據，存取Redfish實作的程式可使用 HEAD 來取得 URI 結構。

<h2>
    Status-Code
</h2>

![image](https://hackmd.io/_uploads/H1J-uMnQR.png)

<h3>
    Debug messages
</h3>

Add different levels of debug messages
* definitions in http/logging.hpp
* BMCWEB_LOG_CRITICAL
* BMCWEB_LOG_ERROR
* BMCWEB_LOG_WARNING
* BMCWEB_LOG_INFO
* BMCWEB_LOG_DEBUG
```
BMCWEB_LOG_DEBUG << "output message " << output_value;

BMCWEB_LOG_DEBUG("output message {}", output_value);
```

Use journal to print debug messages to the console
```
journalctl -f -u bmcweb
```

<h2>
    Redfish Session
</h2>

Access Service 的 Login and Log out 通常是耗時的, 如果去 Program 可能會有額外開銷

所以要 Create Session, 而不是使用 X-Auth-Token

1. GET the /redfish/v1
2. POST to the Session Collection

Create Session
```bash
POST /redfish/v1/SessionService/Sessions
Content-Type: application/json

{
  "UserName": "admin",
  "Password": "password"
}
```

建立 Session 後會取得

![image](https://hackmd.io/_uploads/SJ46NCbrA.png)

之後要使用那個 Location 去 Delete

Using the X-Auth-Token method
```
curl -k -H "X-Auth-Token: MI1VM7WGC35JG2O4" -X GET https:///redfish/v1/
```

DELETE
```
curl -k -H "X-Auth-Token: MI1VM7WGC35JG2O4" -X DELETE https:///redfish/v1/SessionService/Sessions/0;
```

<h2>
    Redfish Command
</h2>

<h3>
    Curl
</h3>

查看 Redfish Version
```
curl -k -u root:0penBmc -H "Content-Type: application/octet-stream" -X GET https://192.168.9.227/redfish/v1 --silent | jq -r ".RedfishVersion"
```

GET
```
curl -k -u <username>:<password> -X GET https://<IP>/redfish/v1
```
PATCH
```
curl -k -u <username>:<password> -X PATCH https://<IP>/redfish/v1/<URL> -d  '{"key": "value"}'
```
POST
```
curl -k -u <username>:<password> -X POST https://<IP>/redfish/v1/<URL> -d  '{"key": "value"}'
```
DELETE
```
curl -k -u <username>:<password> -X DELETE https://<IP>/redfish/v1/<URL> -d ''
```

一般 Get 出來會很亂

![image](https://hackmd.io/_uploads/SkQV7QnmR.png)

可以使用 `jq .` 讓他整齊一點
```
curl -k -u root:0penBmc -X GET https://192.168.9.227/redfish/v1/Systems/system/EthernetInterfaces | jq .
```
![image](https://hackmd.io/_uploads/H14DXXnXC.png)

>**jq** 是一個強大的命令列工具，用於處理和操作 JSON 資料。
>
>它類似於 sed、awk 或 grep，但專門用於 JSON 格式。 
>
>jq 可以用於解析、過濾、修改和格式化 JSON 資料。

<h3>
    Redfishtool
</h3>

Install redfishtool deb package:
```
sudo apt-get install redfishtool
```
![image](https://hackmd.io/_uploads/Hkxfh_VsXA.png)

![image](https://hackmd.io/_uploads/Hy8idEjXA.png)

```
redfishtool -r 192.168.9.227 -S Always -u root -p 0penBmc root
```

![image](https://hackmd.io/_uploads/HyWadEiXA.png)

<h2>
    Redfish Service Validator
</h2>

`Redfish Service Validator` 主要是 parse 你給定的 Redfish service 整個 Redfish tree 裡面的各個 resource 是不是有符合 Schema file 裡的標準
大部份 attributes 是 optional, 若有出現會檢查 type

```
python3 RedfishServiceValidator.py -i https://{bmcip} -u {id} -p {password}
```

<h2>
    Redfish Mockup Creator and Redfish Mockup Server
</h2>

[介紹網址](<https://iris123321.blogspot.com/2022/02/redfish-mockup-creator-redfish-mockup.html>)

Redfish模型工具：**Redfish Mockup Creator** 和 **Redfish Mockup Server**

以上兩個 tool 都是由DMTF開發的，Redfish Mockup Creator 可以去對Redfish Services 作 mockup，而Redfish Mockup Server 我對它的定位在模擬器(emulator)，可以餵給它指定的 Redfish mock，Server 跑起來之後可以對它請求 GET method

對於想要開發 redfish unity，但沒有 BMC 的開發者來說挺方便的

<h2>
    忘記為甚麼貼在這的圖
</h2>

![image](https://hackmd.io/_uploads/rkJNsNlBR.png)