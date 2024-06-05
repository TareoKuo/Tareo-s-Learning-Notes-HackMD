<h2>
    Security details
</h2>

<h3>
    Protocols
</h3>

實作應支援 TLS v1.1 或更高版本。

<h3>
    Cipher suites
</h3>

實作應支援 TLS 套件中基於 AES-256 的密碼。

<h3>
    Certificates
</h3>


Redfish 實作應支援替換預設憑證（如果提供）。

Redfish 實作應使用符合 RFC5280 中定義的 X.509 v3 憑證格式的憑證。

<h3>
    Authentication
</h3>

Service 應該要 support
* Basic Authentication
* Redfish Session Login Authentication

使用 Basic Authentication 時，服務不應要求客戶端建立會話

<h4>
    HTTP header security
</h4>

所有對 Redfish 物件的寫入請求都應經過身份驗證，即 *POST*、*PUT/PATCH* 和 *DELETE*

*Sessions 服務的特例*：
* `POST operation to the Sessions service/object needed for authentication`
    * 當用戶 *首次訪問* 系統時，他們需要創建一個會話（Session），這需要提供憑據以進行身份驗證。
    * 由於用戶在這個過程中還未被認證，所以這個請求不需要事先的身份驗證。

*Redfish API 中哪些對象在未經身份驗證的情況下可以被訪問：*
* 根對象（root object）：用於識別設備和服務位置
* `$metadata` 對象：用於檢索資源類型
* OData 服務文檔（OData Service Document）：用於與 OData 客戶端的兼容性
* 版本對象（version object）：位於 `/redfish`

>大多數 Redfish API 需要進行身份驗證才能被訪問，以確保安全性和防止未經授權的訪問。
>
>通過**外部引用**鏈接的外部服務不在此規範的範疇內，並且這些外部服務可能有其他的安全需求。

<h4>
    Request/Message level authentication
</h4>

每個建立安全通道的請求都應附上**身分驗證標頭**。

<h2>
    Redfish and IPMI difference of Security
</h2>

<h3>
    Redfish
</h3>

基於現代的 RESTful API
* Redfish 使用 HTTPS 作為傳輸層，這意味著所有的數據傳輸都是加密的
* 基於 HTTPS 的安全性比 IPMI 的傳統傳輸層安全協議要強

細密的訪問控制
* Redfish 支持基於角色的訪問控制（RBAC），可以對不同的用戶分配不同的權限，實現更精細的安全控制

OAuth 和其他身份驗證機制
* Redfish 支持多種現代身份驗證機制，包括基本身份驗證、OAuth 等，提供了靈活的身份驗證選項

擴展性和未來支持
* Redfish 的設計考慮了擴展性，易於添加新功能和增強安全性

<h3>
    IPMI
</h3>

使用較老的傳輸層協議
* IPMI 主要依賴於平面文本的傳輸協議，如 RMCP（Remote Management Control Protocol）和 RMCP+，這些協議的安全性較低
* 即使 IPMI 2.0 引入了加密（RMCP+），但整體設計較為陳舊，安全性相對較弱

基於密碼的身份驗證
* IPMI 使用基於密碼的身份驗證，並且許多實現中，密碼是硬編碼的或者難以更改。
* 過於簡單的身份驗證機制容易受到暴力破解和其他攻擊

缺乏細密的訪問控制
* IPMI 的訪問控制不如 Redfish 細密度，無法針對不同的用戶角色設置詳細的權限

已知的安全漏洞
* IPMI 有多個已知的安全漏洞，如惡意軟件可以通過未經授權的訪問來利用 IPMI 接口進行攻擊

<h3>
    Conclusion
</h3>

Redfish 的安全性明顯優於 IPMI。

Redfish 的現代化設計，包括 HTTPS 加密、多種身份驗證機制和基於角色的訪問控制，使其在安全性方面有著顯著的優勢。

相較之下，IPMI 的安全性較弱，尤其是在**傳輸層加密**和**身份驗證機制**上存在明顯劣勢。

考慮到這些因素，Redfish 在**遠程管理**和**監控環境**中的安全性更高，更適合現代數據中心的需求。