<h2>
    Topic
</h2>

[TOC]

<h2>
    IPMI Sensors Introduction
</h2>

The BMC can provide sensors

典型的伺服器 BMC 將提供用於基板溫度、電壓和機殼入侵監控的感測器
透過 IPMI 感測器模型可以存取受監控的信息，例如溫度和電壓、風扇狀態等。


IPMI 不是提供對監控硬體的直接訪問，而是透過抽象感測器命令提供訪問
>例如透過管理控制器 (BMC) 實現的「獲取感測器讀數」命令

他可以分成兩種

**Analog (Threshold)**
    * Analog 類比, 線性數字, 溫度

**Discrete**
    * Digit 數位, 0 or 1

<h2>
    Sensor Type
</h2>

Threshold
* Temperature
* Voltage
* Current
* Fan

Discrete
* Button
* Device present
* Status

其他的可以去 ipmi spec 找 Keyword (Sensor Type Codes)

<h2>
    Sensor Architecture
</h2>

一個 Service 可以提供多個 Sensor

![image](https://hackmd.io/_uploads/SJk_6fRf0.png)

Service 當主控者跟 Linux Kernel 溝通, 再把資料 export DBus 上

![image](https://hackmd.io/_uploads/rktKpzRG0.png)

<h2>
    Sensor Service Architecture
</h2>

為了存取 export hardware data to D-Bus

<h3>
    Dbus Sensor
</h3>

他是一個 recipe，提供了 11 種 Sensor service，然後他大部分都是 Threshold section

```
Service     xyz.openbmc_project.Hwmon.Hwmon[x]
Path        /xyz/openbmc_project/Sensors/<type>/<label>
Interfaces  xyz.openbmc_project.Sensor.[*]

Signals: All properties for an interface will broadcast signal changed
```

**Path definitions**

`<type>` : [HWMon class] (https://www.kernel.org/doc/Documentation/hwmon/sysfs-interface) 名稱小寫。例如溫度、風扇、電壓。

`<label>` : 使用者定義的感測器名稱。例如環境、cpu0、fan5

[Sensor Support for OpenBMC](<https://gerrit.openbmc.org/plugins/gitiles/openbmc/docs/+/a33c90e81c3af3488e64dcaff1619b7da86bb3d4/sensor-architecture.md>)

<h2>
    Entity Manager
</h2>

Entity Manager 是一種用於管理實體系統元件並將它們對應到 BMC 內的軟體資源的設計。

有很多 Sensor 之後,就要來設定 server 應該有哪些 sensor, 這邊就是用 Entity Manager 來去設定

![image](https://hackmd.io/_uploads/B1G2y7AzR.png)

<h3>
    Entity Manager Config
</h3>

![image](https://hackmd.io/_uploads/B1pnxQ0fR.png)

可以 define Threshold, 這個是 option 的

左邊是 EntityManager 原本就有, 在 OpenBMC 原本說 Sensor number 不重要

>"type": "TMP75"
>
>LM75, I2C 的 IC, I2C bus -> 11

<h3>
    Add Sensor
</h3>

通常在增加 Sensor 會先在 DTS 去註冊他, 註冊之後在 entity-manager 的 configuartion 添加我們所支援的單板的json文件


<h2>
    IPMI Provides the Standard Messange Interface
</h2>

以下概念與此訊息介面相關：

* **Channel type** (頻道類型)：通信頻道類型（SMS / KCS，IPMB，LAN）
* **Channel number** (頻道號碼)：頻道描述符(descriptor)
* **Requester** (請求者)：請求者的地址address
* **Responder** (響應者)：響應者的地址address
* **NetFN**：請求/響應的邏輯功能(logical function)。
* **Command** (命令)：命令編號
* **Sequence** (序列)：標識請求/響應對(pair) 的ID
* **Message tracking** (訊息追踪)：匹配請求/響應對(pair)的能力(ability)。

當通過任何通道發出通信時，應用程序會格式化一個請求(request)，並期望一個響應(response)。

該應用程序還包括API的 “human readable (人類可讀)” Instance 實例：

```
ipmitool mc selftest

Selftest: passed
```

<h3>
    Direct Command
</h3>

最簡單的通信形式是使用 **SMS**/**KCS** 的 “Direct Command”

Example : 
```
ipmitool raw 6 1
```

這將 NetFn 6（Application）的 raw command 4（selftest自我測試）發送到 KCS，driver 負責 “**Message Tracking** (訊息追踪)” 並提供答案。

<h2>
    BMC IPMI and Events
</h2>

![image](https://hackmd.io/_uploads/SJJyywZkye.png)

PM 現在比較常見是 ME, 或是另一顆 BMC

BMC 他是一個 Event Generator, Event Receiver

IPMB 透過 Event Command 的方法進去

<h3>
    IPMI Event Structure Dirgram
</h3>

![image](https://hackmd.io/_uploads/r1Uw1P-1Je.png)

**第一步**：Event 收到後一定會 Log 起來 ⟶ SEL, 一定會寫進去 SEL
* 一定要存在 Non-Volatile Storage

**第二步**：可能會有送到 PEF(Platform Event filter) Optional
* 會幫你 filter 你感興趣的東西進來

<h3>
    Sensor Classes and Event/Reading Types
</h3>

Table 42-1 Event/Reading Type Code Ranges
* Event Reading Type and Sensor Reading Type are different

大部分都是 Threshold based, 數字以外都是 Discrete
>Threshold based(a.k.a Analog Sensor)

![image](https://hackmd.io/_uploads/Sy9A1v-J1x.png)

<h3>
    Event Condition
</h3>

**event = something happens = event condition holds (occurs)**

<br/>

for threshold based sensors：
* reading triggers threshold(s)
* reading is back to normal
* analog class, threshold(門檻)
* 有 6個 Threshold 去發 event

for discrete sensors：
* state is **on**
* state is **gone**


<h2>
    Reference
</h2>