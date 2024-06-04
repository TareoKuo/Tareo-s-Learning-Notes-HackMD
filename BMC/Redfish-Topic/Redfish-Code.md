<h2>
    Redfish Code
</h2>

```cpp=
inline void requestRoutesTareoService(App& app)
{
    BMCWEB_LOG_ERROR << "Hello GET";
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoHello/TareoService/")
        .privileges(redfish::privileges::getTareoService)
        .methods(boost::beast::http::verb::get)
    (
        [](const crow::Request&,
            const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
        {
            asyncResp->res.jsonValue =
            {
                {"@odata.id", "/redfish/v1/Oem/TareoService/TareoHello"},
                {"@odata.type", "#TareoHello.v1_0_0.TareoHello"}, // TareoService.v1_0_0 內的 Entity Type
                {"Id", "HelloService"},
                {"Name", "Hello Service"},
                {"Description", "Tareo Hello Service"},
                {"Status",
                    {
                        {"Health", "OK"},
                        {"HealthRollup", "OK"},
                        {"State", "Enabled"}
                    }
                },
                {"Hello", {{"@odata.id", "/redfish/v1/Oem/TareoHello/TareoService/Hello"}}},
                {"PropertyBoolean", helloBoolean},
                {"PropertyNumber", helloNumber},
                {"PropertyString", helloString},
                {"PropertyNullNotWritable", nullptr},
                {"PropertyStringArray", helloStringArray}
            };
        }
    );
}
```

<br>

* BMCWEB_ROUTE
```
BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoHello/TareoService/")
```
定義了一個新的路由，對應的 URL 是 `/redfish/v1/Oem/TareoHello/TareoService/`

* privileges
```
.privileges(redfish::privileges::getTareoService)
```
：指定存取這個端點所需的權限

* methods
```
.methods(boost::beast::http::verb::get)
```
指定這個路由處理 GET 請求。

* Lambda 函數
```
[](const crow::Request&, const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
```
這是一個Lambda 表達式，接受兩個參數，一個是請求物件`crow::Request`，另一個是響應對象`bmcweb::AsyncResp` 的共用指標。

<h2>
    Lambda 表達式
</h2>

[Lambda](<https://tjsw.medium.com/%E6%BD%AE-c-14-generic-lambda-%E7%84%A1%E8%85%A6%E5%AF%AB%E4%B8%8D%E7%94%A8%E7%AE%A1%E5%9E%8B%E5%88%A5-%E9%82%84%E8%83%BD%E7%89%B9%E6%8A%80%E8%A1%A8%E6%BC%94-54c3668734a9>)

<h2>
    JSON
</h2>

也可以用這種方法加進去

```cpp=
nlohmann::json hello1 = {
    {"@odata.id", "/redfish/v1/Oem/TareoService/TareoHello/Hello/1"},
    {"Id", "1"},
    {"Name", "HELLO1"},
    {"Description", "TEST HELLO 1"}
};
asyncResp->res.jsonValue["Hello"].push_back(hello1);
```

還有
```cpp=
asyncResp->res.jsonValue["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello/Hello1";
asyncResp->res.jsonValue["@odata.type"] = "#TareoHello.v1_0_0.Hello";
asyncResp->res.jsonValue["Description"] = "TEST HELLO 1";
```

<h2>
    Dbus
</h2>

<h3>
    Use Redfish to access data from dbus code
</h3>

[Reference](<https://wiki.insyde.com/index.php/How_to_get_dbus_data_User_Guide#Use_Redfish_to_access_data_from_dbus_code:~:text=projects%3A%20wrap%20files-,Use%20Redfish%20to%20access%20data%20from%20dbus%20code,-How%20to%20get>)

Use callback function to get dbus property

* **async_method_call**: 使用 `async_method_call` 發送 Dbus 命令
* **Usage Variables**: `asyncResp{std::move(asyncResp)}`
* Declaration of chassisState return value in Dbus
* Get the return value as a string pointer to s
* Capture the Dbus response
* **Use callback function with parameter**: dbus, service name, object path, interface, property name to get the property value

<h3>
    GET
</h3>

```cpp=
inline void getTareoDbusTestData(const std::shared_ptr<bmcweb::AsyncResp>& asyncResp)
{
    BMCWEB_LOG_DEBUG << "Get TareoDbusTest Data";

    // sdbusplus::asio::getProperty<std::string>(
    //     *crow::connections::systemBus, "xyz.openbmc_project.TareoDbusTest",
    //     "/xyz/openbmc_project/TareoDbusTest", "xyz.openbmc_project.TareoDbusTest",
    //     "Status",
    //     [asyncResp](const boost::system::error_code ec, const std::string& status)
    //     {
    //         if(ec) {
    //             BMCWEB_LOG_ERROR << "D-BUS response error " << ec;
    //             return;
    //         }
    //         HelloStatus = status;
    //         BMCWEB_LOG_DEBUG << "HelloStatus value = " << HelloStatus;
    //         asyncResp->res.jsonValue["Hello"]["Status"] = HelloStatus;
    //     }
    // );

    crow::connections::systemBus->async_method_call(
        [asyncResp](const boost::system::error_code ec, std::variant<std::string>& status)
        {
            if(ec) {
                BMCWEB_LOG_ERROR << "D-BUS response error " << ec;
                return;
            }

            const std::string* value = std::get_if<std::string>(&status);

            if (value == nullptr)
            {
                messages::internalError(asyncResp->res);
                return;
            }
            HelloStatus = *value;

            BMCWEB_LOG_DEBUG << "HelloStatus value = " << HelloStatus;
            asyncResp->res.jsonValue["Hello"]["Status"] = HelloStatus;
        },
        "xyz.openbmc_project.TareoDbusTest", "/xyz/openbmc_project/TareoDbusTest",
        "org.freedesktop.DBus.Properties", "Get",
        "xyz.openbmc_project.TareoDbusTest", "Status");
```

有兩種作法可以拿值
* `sdbusplus::asio::getProperty<std::string>`
* `crow::connections::systemBus->async_method_call`

<h3>
    SET
</h3>

```cpp=
inline void setTareoDbusTestData(const std::shared_ptr<bmcweb::AsyncResp>& asyncResp, const std::string& name, const std::string& data)
{
    BMCWEB_LOG_DEBUG << "Set TareoDbusTest Data";
    BMCWEB_LOG_DEBUG << "SETTTTT name value : " << name;
    BMCWEB_LOG_DEBUG << "SETTTTT data value : " << data;

    VariantType dbusPropertyValue(data);

    crow::connections::systemBus->async_method_call(
        [asyncResp](const boost::system::error_code ec)
        {
            if(ec) {
                BMCWEB_LOG_DEBUG << "[Set] Bad D-Bus request error: " << ec;
                messages::internalError(asyncResp->res);
                return;
            }

            messages::success(asyncResp->res);
        },
        "xyz.openbmc_project.TareoDbusTest", "/xyz/openbmc_project/TareoDbusTest",
        "org.freedesktop.DBus.Properties", "Set",
        "xyz.openbmc_project.TareoDbusTest", name, dbusPropertyValue);
}
```

**DATA 的部份要使用 <font color="#f00">`VariantType dbusPropertyValue(data);`</font> 去給值, 不然 Dbus 會出現 ERROR**

<h2>
    Library
</h2>

<h3>
    sdbusplus
</h3>

[Sdbusplus](/s7yxjmdiSuSrxkxSd9mgWg)

是一個用於處理 **D-Bus（Desktop Bus)** 通訊的函式庫，它提供了一個簡單的 C++ 接口，使開發者能夠輕鬆地在應用程式中使用 D-Bus 進行通訊。

**ChatGPT :**
![image](https://hackmd.io/_uploads/HJusXC-ER.png)

<h3>
    C++ boost asio
</h3>

**Boost Asio** 是用於**網路**和**低層 I/O** 程式設計的 C++ 函式庫，屬於 Boost 函式庫的一部分。

1. 非同步 I/O
2. 跨平台
3. 事件驅動

**io_context**

io_context 是 Boost Asio 的核心，負責管理所有的非同步 I/O 操作。

io_context 對象，並將所有的 I/O 操作綁定到這個對象。

```cpp
boost::asio::io_context ioContext;
```

<h3>
    shared_ptr
</h3>

`std::shared_ptr<std::string>` 是一個智慧指針，用於管理動態分配的 std::string 物件的記憶體。

它允許多個指標共享對相同物件的所有權，並在最後一個指標不再需要時自動釋放記憶體。

```cpp
const std::shared_ptr<bmcweb::AsyncResp>& asyncResp
```

<h3>
    make_shared
</h3>

`std::make_shared<std::string>(...)` 是一個函數模板，用於建立一個動態分配的 std::string 對象，並傳回指向該物件的 std::shared_ptr<std::string>。

這個函數模板接受任何類型的參數，並用這些參數來建構一個新的物件。

```cpp
std::shared_ptr<std::string> HelloStatus = std::make_shared<std::string>();
```

<h2>
    Parameter
</h2>

<h3>
    Request
</h3>

```cpp
const crow::Request&
```

`crow::Request` 物件通常用於處理HTTP請求，包含了請求的各種訊息，例如請求方法、路徑、頭部資訊等。

<h3>
    asyncResp
</h3>
    
```cpp
const std::shared_ptr<bmcweb::AsyncResp>& asyncResp
```

他接收一個對非同步回應物件的引用，該引用用於處理 HTTP 回應。

透過該引用，可以設定回應的狀態碼、新增回應頭部資訊等。