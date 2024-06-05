<h2>
    Redfish Service Details
</h2>

Redfish Service 需要 client 或 administrator **建立訂閱來接收事件**。

There are two methods of creating a subscription：
* 直接透過向訂閱集合發送 HTTP POST 來建立訂閱
* 為事件服務開啟伺服器發送事件 (SSE) 連線時間接建立訂閱。

<h2>
    Redfish Event
</h2>

<h3>
    Event Service Model Map
</h3>

![image](https://hackmd.io/_uploads/BJUrhoIER.png)

<h3>
    Event Service
</h3>

![image](https://hackmd.io/_uploads/ByX_niLNC.png)

![image](https://hackmd.io/_uploads/H1fK2i8VA.png)

<h3>
    Event
</h3>

![image](https://hackmd.io/_uploads/H1ah3j8EA.png)

<h3>
    Message Registry (example entry, not official)
</h3>

![image](https://hackmd.io/_uploads/rkb1aiUVC.png)

<h3>
    How a client finds the message in a registry a forms a message
</h3>

* Use the MessageId to find the registry, version and message
* Use that information to find the registry and the message in it
    * The ServiceRoot has a link to the MessageRegistry collection.
* Populate the message for the end user
    * 取得 Message 屬性值並開始替換事件本身的訊息參數

![image](https://hackmd.io/_uploads/ryLH0oU4A.png)


<h2>
    Redfish Log Mechanism
</h2>

有兩種
* LogService
    * 負責紀錄 System Event
* EventService

<h3>
    OpenBMC Rsyslog
</h3>

1. Check if the log has the `REDFISH_MESSAGE_ID` filed
2. Store in `/var/log/redfish` using the redfishtemplate format
3. Redfish will retrieve data form `/var/log/redfish`
4. Add new Redfish log using the `sd_journal_send` function

<h3>
    Prefix
</h3>

這些 Prefix 用於標識不同類型的事件

目前 OpenBMC 支援的 prefix 四種：
* Base
* TaskEvents
* ResourceEvent
* OpenBMC

message_registeries.hpp : 
![image](https://hackmd.io/_uploads/rJqcpcc4R.png)

<h3>
    LogService
</h3>

![image](https://hackmd.io/_uploads/BkSWBs8VA.png)

<h3>
    EventService
</h3>

[Redfish-EventService](<https://github.com/openbmc/docs/blob/master/designs/redfish-eventservice.md>)

* Architecture

![image](https://hackmd.io/_uploads/BkCWHoL4A.png)

`EventService` 可以偵測有沒有sel 新增，並即時的通知使用者
他有兩種訂閱方式：
* **RedfishEvent**
    * 當服務偵測到需要傳送事件時，它會使用 HTTP POST 將事件訊息推送到客戶端。
    * 客戶端可以透過在 `EventService` 中建立訂閱條目來啟用事件接收。
* **SSE (Server-Sent Events)**
    * 用戶端透過對事件服務中「`ServerSentEventUri`」指定的 URI 執行 GET 來開啟與服務的 SSE 連接

EventService 也有對 Subscriptions 提供一些filter

`[Get] /redfish/v1/EventService` 中的SSEFilterPropertiesSupported

![image](https://hackmd.io/_uploads/Hyqc8iINC.png)

>ServicesRoot 就是 /redfish/v1

```
/redfish/v1/EventService/Subscriptions/SSE?$filter=RegistryPrefix eq OpenBMC
```

<h4>
    RedfishEvent
</h4>

訂閱方式是透過
`[POST]/redfish/v1/EventService/Subscriptions` with body : 

![image](https://hackmd.io/_uploads/rye1LjLNR.png)

<h4>
    SSE(Server-Sent Events)
</h4>

它算是一種比較新的網路溝通協定，當 server-client 建立連線後，server 端就可以一直 push data 給 client 端，這邊 bmc 就是做為 server 端

因此我們只需要瀏覽器中輸入 SSE URI (`/redfish/v1/EventService/Subscriptions/SSE`)，連上BMC之後，就可以不定時收到新增的sel

![image](https://hackmd.io/_uploads/B1mIUoUNC.png)

<h2>
    EventService
</h2>

Redfish 擁有三大資源來支援 EventService。
* Event Service
* Subscription Collections
* Subscriptions

<h3>
    EventService
</h3>

包含服務的屬性，例如**服務啟用**、**狀態**、**傳送重試嘗試**、**事件類型**、**訂閱**、**操作 URI** 等。

* URI : `/redfish/v1/EventService`
* SCHEMA : EventService
* METHODS : GET, PATCH

<h3>
    Subscription Collections
</h3>

此資源用於取得所有訂閱的集合，也用於配置自動事件的新訂閱。

* URI : ``/redfish/v1/EventService/Subscriptions
* SCHEMA : EventDestinationCollection
* METHODS : GET, POST

<h3>
    Subscriptions
</h3>

此資源包含訂閱**詳細資訊**。

用於**獲取**每個人的訂閱信息，**更新**和**刪除**現有的訂閱信息。

* URI : `/redfish/v1/EventService/Subscriptions/<ID>`
* SCHEMA : EventDestination
* METHODS : GET, PATCH, DELETE

<h2>
    SSE (Server-Sent Events)
</h2>

EventService 的資源包含一個名為「`ServerSentEventUri`」的屬性。

如果用戶端對「`ServerSentEventUri`」指定的 URI 執行 GET, bmcweb 伺服器將保持連線開啟並符合 HTML5 規範，直到用戶端關閉套接字。

bmcweb 服務應將回應傳送至具有以下標頭的 GET URI。

```
'Content-Type': 'text/event-stream',
'Cache-Control': 'no-cache',
'Connection': 'keep-alive'
```

<h3>
    Open connection
</h3>

當用戶端使用所需的過濾器對「`ServerSentEventUri`」執行 get 時，bmcweb 在訂閱集合中建立 EventDestination 實例。

一旦 bmcweb 收到 SSE 串流請求，開啟的連線將透過檢查請求 Content-Type 標頭（與「`text/event-stream`」相符）升級為 WebSocket，並保持連線處於活動狀態，直到客戶端/伺服器關閉連線。

>因為他是在現有的 GET 上進行操作, 所以**會正常經過身份驗證和授權**, 所以不存在 security threat

<h3>
    Close connection
</h3>

client 或 Servier 可以隨時終止事件流並關閉連線。 

bmcweb 伺服器可以在連線關閉時刪除對應的 EventDestination 實例。

連接可以透過以下方式關閉
* Client 透過在訂閱的 id URI 上傳送 DELETE 方法。
* Service（bmcweb）可以關閉連線並向客戶端發送帶有「event: close」請求正文的通知。

>**由於 SSE 是單向的**，如果客戶端突然關閉連線/瀏覽器，bmcweb服務不知道客戶端連線狀態。這是 SSE 連線的限制。
>
>在這種情況下，如果傳送錯誤的數量超過預先配置的閾值，bmcweb 服務可能會刪除訂閱。

**The SSE subscription data is not persistent in nature.**

所以 BMC 重新啟動時, Client 端要去重新建立連線

Supported filters are:
* RegistryPrefix
* ResourceType
* EventFormatType
* MessageId
* OriginResource

<h2>
    Push style events (HTTP/HTTPS)
</h2>

BMC 充當 HTTP 用戶端並將資料傳送到外部系統上執行的 Web 伺服器。

在 SSE 的情況下，連線將保持活動狀態，直到串流關閉，但在推送事件中，連線根據需要打開，並在推送事件/報告資料後立即關閉。

透過在「`/redfish/v1/EventService/Subscriptions`」URI 上使用帶有所需請求正文（必填欄位：目的地、協定）的 POST 方法，使用者可以訂閱事件。

當有事件日誌或遙測資料可用時，監視器和格式化程式資源（事件日誌和 MetricReport）將訂閱的資料用於匹配規則。 

「事件服務發送者」使用訂閱資料中的目標欄位來開啟連線並將資料推送到外部運行的網路伺服器。

<h3>
    SubmitTestEvent
</h3>

使用 action parameters 指定的 Event Data, 將 Test Event 新增到 Event Service, 然後根據 Subscription data 將 message 傳送到所有目標

* URI: `/redfish/v1/EventService/Actions/EventService.SubmitTestEvent`
* SCHEMA: EventService
* METHODS : POST

Example POST Request body : 

![image](https://hackmd.io/_uploads/Sk39Mp8V0.png)

<h2>
    Implement
</h2>

可以使用下面指令發送 Test Event

```
curl -k -u root:0penBmc -X POST https://192.168.100.124/redfish/v1/EventService/Actions/EventService.SubmitTestEvent
```

<h3>
    Send Event
</h3>

* event_service.hpp
```cpp
inline void requestRoutesSubmitTestEvent(App& app)
{

    BMCWEB_ROUTE(
        app, "/redfish/v1/EventService/Actions/EventService.SubmitTestEvent/")
        .privileges(redfish::privileges::postEventService)
        .methods(boost::beast::http::verb::post)(
            [](const crow::Request&,
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) {
                EventServiceManager::getInstance().sendTestEventLog();
                asyncResp->res.result(boost::beast::http::status::no_content);
            });
}
```

* event_service_manager.hpp
```cpp
void sendTestEventLog()
    {
        nlohmann::json logEntryArray;
        logEntryArray.push_back({});
        nlohmann::json& logEntryJson = logEntryArray.back();

        logEntryJson = {
            {"EventId", "TestID"},
            {"EventType", "Event"},
            {"Severity", "OK"},
            {"Message", "Generated test event"},
            {"MessageId", "OpenBMC.0.2.TestEventLog"},
            {"MessageArgs", nlohmann::json::array()},
            {"EventTimestamp", crow::utility::getDateTimeOffsetNow().first},
            {"Context", customText}};

        nlohmann::json msg = {{"@odata.type", "#Event.v1_4_0.Event"},
                              {"Id", std::to_string(eventSeqNum)},
                              {"Name", "Event Log"},
                              {"Events", logEntryArray}};

        this->sendEvent(
            msg.dump(2, ' ', true, nlohmann::json::error_handler_t::replace));
    }
```