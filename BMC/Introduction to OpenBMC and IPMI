<h2>
    Topic
</h2>

[TOC]

<h2>
    OpenBMC Introduction
</h2>

OpenBMC 是由 Linux 基金會支持與管理的開放原始碼計劃，其目標在於建立一個適合於管理基板管理控制器（Baseboard Management Controller，BMC）的軟體框架。

<h2>
    BMC Introduction
</h2>

BMC（Board Management Controller，基板管理控制器），他是在伺服器主板上的一個小型專用處理器，用於遠端監控和主機系統的管理。通常基於 ARM 的 SoC（System on Chip）。

它的主要功能是監控、管理和控制電腦系統的硬體，即使在操作系統關閉或無法運行時也可以進行操作。

>BMC 是一個獨立的系統，它不依賴與系統上的其它硬體（例如CPU、記憶體等），也不依賴與BIOS、OS等（但是BMC可以與BIOS和OS交互，這樣可以起到更好的平台管理作用，OS下有系統管理軟體可以與BMC協同工作以達到更好的管理效果）。

![image](https://hackmd.io/_uploads/B1g9zzKzC.png)

<h3>
    OpenBMC Interface
</h3>

```
        +----------------+         +----------------+
        | BMC            |         | Host           |
        |                |         |                |
        | Network       -+- LPC ---+-               |
       -+- eth0         -+--PCIe --+-               |
       -+- eth1         -+--UART --+-               |
        |  lo           -+- I2C ---+-               |
        |               -+--I3C ---+-               |
        | USB           -+- SPI ---+-               |
       -+- usb0         -+- PECI --+-               |
        |               -+- GPIOs -+-               |
        | Serial        -+- UTMI --+-               |
       -+- tty0          |         |                |
        |                |         |                |
        +----------------+         +----------------+
```

**Network services provided**

```
        +----------------------------------+
        | BMC                              |
        |                                  |
       -+-+ Network services               |
        | |                                |
        | +-+ TCP ports                    |
        | | +- 22 ssh - shell              |
        | | +- 80 HTTP (no connection)     |
        | | +- 443 HTTPS                   |
        | | +- 2200 ssh - host console     |
        | | +- 5355 mDNS service discovery |
        | |                                |
        | +-+ UDP ports                    |
        |   +- 427 SLP                     |
        |   +- 623 RMCP+ IPMI              |
        |   +- 5355 mDNS service discovery |
        |                                  |
        +----------------------------------+
```

**Host console**

```
* Client access via network IPMI.
* Client access via ssh port 2200.
* The hostlogger facility.

                +---------------------------+    +-----------------+
                | BMC                       |    | Host            |
 ipmitool sol   |                           |    |                 |
 activate       |                           |    |                 |
 UDP port 623 .... netipmid ------------}   |    |                 |
                |                       }   |    |                 |
 ssh -p 2200   ... obmc-console-client -}---+----+- serial UART    |
 TCP port 2200  |                       }   |    |  console        |
                |  hostlogger ----------}   |    |                 |
                |                           |    |                 |
                +---------------------------+    +-----------------+
```

**Web services**

```
OpenBMC provides a custom HTTP/Web server called BMCWeb.

        +--------------------------------------------------+
        | BMC                                              |
        |                                                  |
       -+-+ Network services                               |
        | ++ TCP                                           |
        |  +- 443 HTTPS - BMCWeb -> { static content       |
        |  |                        {   Web app (webui)    |
        |  +- (other ports) <---+   {   Redfish schema     |
        |       |               |   { /login               |
        |       V               |   { Redfish REST APIs    |
       -+- Websockets -+        |   { Phosphor REST APIs   |
        |              |        +<--{-- can set up:        |
        |              |            {     KVM-IP, USB-IP,  |
        |           various         {     Virtual Media    |
        |                                                  |
        +--------------------------------------------------+
```

**Host IPMI services**

```
OpenBMC provides a host IPMI service.

    +---------------+    +-----------------+
    | BMC           |    | Host            |
    |               |    |                 |
    |        ipmid -+----+-                |
    |               |    |                 |
    +---------------+    +-----------------+
```

**D-Bus interfaces**

```
OpenBMC uses D-Bus interfaces as the primary way to communicate (inter-process communication) between OpenBMC applications. 
Note that other methods are used, for example Unix domain sockets.

        +--------------------------------------------------+
        | BMC                                              |
        |                                                  |
        | +-------+                                        |
        | | D-Bus |                                        |
        | |      -+- bmcweb                                |
        | |      -+- ipmid                                 |
        | |      -+- ...                                   |
        | |      -+- many more (not shown here)            |
        | |      -+- ...                                   |
        | |       |                                        |
        | +-------+                                        |
        |                                                  |
        +--------------------------------------------------+
```

<h3>
    BMC Architecture
</h3>

![image](https://hackmd.io/_uploads/H1PJDQFG0.png)

sensor monitor daemon 會去定期讀取 sensor value，使用者可以透過 redfish 來獲取讀值，那 redfish daemon 要去哪裡獲取讀值呢？

OpenBMC 給的答案是 "D-Bus"

![image](https://hackmd.io/_uploads/SkxbDmKG0.png)


<h2>
    IPMI
</h2>

IPMI(Intelligent Platform Management Interface) 智慧平台管理接口

IPMI就是對「平台管理」這個概念的具體的規範定義，該規範定義了「平台管理」的軟硬體架構，互動指令，事件格式，資料記錄，能力集等

然後 BMC 是 IPMI 裡面的一個核心的部分，是 IPMI 硬體架構

![image](https://hackmd.io/_uploads/SJtzMGtzC.png)

灰色的部分是 IPMI 的範圍

<h2>
    IPMI Components
</h2>

IPMI 由下列組件組成 :
![image](https://hackmd.io/_uploads/SJORGGtGR.png)

上面 A micro-controller (BMC) 是 IPMI 架構的核心，以下是 BMC 包含的任務 : 
* 系統管理軟體和所使用的硬體之間的介面（透過它使用 IPMB 和 ICMB 連接 BMC）
* 獨立監控
* 獨立記錄事件
* 控制恢復

<h3>
    MOTHERBOARD
</h3>

![image](https://hackmd.io/_uploads/HkMuLmKGA.png)

<h3>
    Non-volatile Storage
</h3>

我們知道BMC其實是一個獨立的晶片，那麼它肯定也需要運作系統。透過BMC裡面運作的是一個類別Unix系統，而這個系統就存放再Non-volatile Storage中，通常就是SPI Flash裡面。

![image](https://hackmd.io/_uploads/SyBsL7KGA.png)

<h3>
    Sensors & Control Circuitry
</h3>

![image](https://hackmd.io/_uploads/B17p8XYzC.png)

<h2>
    IPMB (Intelligent Platform Management Bus)
</h2>

IPMI 允許透過應用 IPMB 標準透過附加管理控制器 (MC) 來擴充 BMC。

IPMB 是一種基於 I²C 的序列匯流排，可與 one chassis 內的各種 boards 進行連接。

它用於管理控制器 (MC) 之間的通訊以及管理控制器 (MC) 之間的通訊。額外的 MC 通常被指定為 Satellite Controllers。

![image](https://hackmd.io/_uploads/SyILLQFfR.png)

<h2>
    ICMB (Intelligent Chassis Management Bus)
</h2>

ICMB 為 chasses 之間的通訊和控制提供標準化介面。

<h2>
    IPMI Memory Areas
</h2>

IPMI 將資訊儲存到 
* System Event Log (SEL)
* Sensor Data Record (SDR) 
* Repository
* Field Replaceable Units (FRU)
* PEF (Platform Event Filtering)

<h2>
    System Event Log (SEL)
</h2>

BMC 包含一個中央 **non-volatile** System Event Log (SEL)。由於此 SEL **由 BMC 管理**，因此即使在伺服器上發生 CPU 故障後也可以對其進行訪問，例如透過 IPMI LAN 存取。

一系列 IPMI 命令 permits reading 和 deleting SEL。

由於 SEL 的記憶體有限，因此必須定期檢查和刪除它，以便記錄其他事件。

<h2>
    Sensor Data Record (SDR) Repository
</h2>

Sensor Data Records 是包含感測器類型和數量資訊的記錄。
因此，感測器數據記錄描述了特定的感測器。

Sensor Data Records 儲存在中央 **non-volatile** storage area，**由 BMC 管理**。
此儲存區域稱為 Sensor Data Record Repository（SDR Repository

<h2>
    Field Replaceable Unit (FRU) Information
</h2>

IPMI supports 儲存系統中 various modules 的 Field Replaceable Unit (FRU) information。 
FRU 資料包含下列等資訊 : 
* serial numbers、
* part numbers、
* models numbers and inventory numbers 
(sometimes called “asset tags”)

![image](https://hackmd.io/_uploads/ByJRLQtGA.png)

<h2>
    PEF (Platform Event Filtering)
</h2>

平台事件過濾提供了一種機制，用於配置 BMC 以對其接收或內部產生的事件訊息採取選定的操作

<h2>
    Communication Interfaces
</h2>

IPMI 允許透過下列介面進行 IPMI 訊息傳遞：
* system interfaces (local access)
* serial (or modem) interface (access through a serial port or a modem)
* LAN interface
* (ICMB and PCI Management Bus)

<h2>
    System Interfaces
</h2>

IPMI 定義了多個 system interfaces，用於系統軟體本地存取 BMC。
有多種介面可支援最廣泛的微控制器。
系統介面可以透過 IO 或記憶體映射存取來存取。

The IPMI system interfaces are:
* keyboard controller style (KCS)
* system management interface chip (SMIC)
* block transfer (BT)
* SMBus system interface (SSIF)

<h2>
    Serial/Modem Interface
</h2>

serial 或 modem interface 規格定義瞭如何透過直接串列或外部數據機連接在 BMC 之間傳輸 IPMI 訊息。

Three Connection Modes are supported for this:
* basic mode
* PPP mode
* terminal mode

<h2>
    LAN Interface
</h2>

LAN interface specifications 定義如何在封裝的 Remote Management Control Protocol (RMCP) UDP 資料封包（UDP target port 623 for asf-rmcp）中將 IPMI 訊息傳送至 BMC 或從 BMC 傳送。此功能也稱為 “IPMI-over-LAN”。 

IPMI 也定義了特定於 LAN 的配置設置，有點類似於 IP 位址的配置設定。

RMCP 是由 DMTF(Distributed Management Task Force) 定義的
除了 IPMI 之外，此封包格式也用於 DMTF 警報標準論壇 (ASF) 規格。

IPMI 2.0 中也定義了附加資料包格式 (RCMP+)。 
RMCP+ 除了支援 various extensions for **authentication** 功能之外，還支援 **encrypted data transmission**。

>IPMI Specification 中有提到如果有 support IPMI v2.0 就要 support IPMI v1.5

<h3>
    Payloads
</h3>

Payloads 是除了 IPMI messages 之外還透過 RMCP+ IPMI session 傳輸更多資料的功能。
這種附加資料類型的一個範例 serial-over-LAN (SOL)。

<h2>
    Channel Model
</h2>

IPMI 使用 channel model 在 communication interface 和 BMC 之間進行直接通訊。
因此，每個通道都有自己的 properties 和 configuration：
* a unique channel number
* the type of communication interface (such as LAN interface)
* users and passwords (使用者不是為整個 BMC 建立的，而是始終為通道單獨建立的。)
* individually supported authentication modes (such as MD5)
* an individually configurable Channel Privilege Limit
* IMPI messaging and alerting can be also be activated or deactivated individually for each channel

<h3>
    Channel Numbers
</h3>

每個頻道都有一個獨立的頻道號碼。
僅預先定義了主 IPMB 的 channel number（channel number 0）和 system interface（channel number 0x0F or 15）。
其餘通道數取決於各自的實作。

![image](https://hackmd.io/_uploads/rkRMxQYfC.png)

<h3>
    Session
</h3>

channel 可以是 session-based 的或 session-less 的。
在此，session 實現以下兩個目的
* a session provides a framework for user authentication
* a session makes processing of several IPMI messaging streams possible on a single channel

**session-based** channels : 
* LAN
* serial/modem channels

**session-less** channels : 
* system interface
* IPMB channels

然後與 BMC 的 IPMI Messange 傳遞連線符合以下三種分類之一：
* session-less (無會話)
    * 不用認證, 不用建 session
* single-session (單一會話)
    * 一次只有一個人建 session
* multi-session (多重會話)
    * 可以有多個人 session

>然後沒有建 Session 就不用檢查 User, Password

![image](https://hackmd.io/_uploads/HJG1SiYf0.png)


<h3>
    Channel Privilege Levels
</h3>

可以配置 Channel，以便可以以特定的最大權限等級使用它們。各種權限等級是：
* None, Callback, User, Operator, Administrator, OEM
    * OEM 預期權限最大
    * Lan Channel 連上去 Permission limite user
        * 需要使用 Set Session Privilege Level command 提升 Permission 才能提升 Permission

![image](https://hackmd.io/_uploads/B15kfmYMR.png)

Channel Privilege Levels
* 某個 Channel 可以使用的最大權限
* Set Channel Access command (NetFn=App, CMD=40h)
* 還有另外一個 Get channel command

Session Privilege Levels
* Set Session Privilege Level command (NetFn=App, CMD=3Bh)

User Privilege Levels
* Set User Access command (NetFn=App, CMD=43h)
    * Enable/disable user access by channel

>Channel Privilege 如果被調小 remote 就沒救了, 因為被 default 在 user privilege
>
>如果有 user name, password 都一樣會使用第一個, 如果要改 password 要使用 user id 去指定

<h2>
    Serial Over LAN (SOL)
</h2>

Serial-over-LAN (SOL) 表示透過 IPMI session 將資料流量重新導向至 based board’s (motherboard’s) 的 serial port。

在某種程度上，這使得可以存取 BIOS interface（if serial redirection has been configured），以及訪問 Grub 等 bootloaders，甚至可以訪問 Linux command console（if a serial console has been configured there）。

SOL 已定義為 IPMI v2.0 (RMCP+) 中的 payload type。

<h2>
    Changes to IPMI
</h2>

<h3>
    Changes in IPMI v1.5
</h3>

IPMI 2.0 specification 的第 5.1 節列出了以下變更：
* Serial/Modem Messaging and Alerting
* Serial Port Sharing
* Boot Options
* LAN Messaging and Alerting
* Extended BMC Messaging ‘Channel Model’
* Additional Sensor and Event Types
* Platform Event Filtering (PEF)
* Alert Policies

<h3>
    Changes in IPMI v2.0
</h3>

IPMI 2.0 specification 的第 1.6 節列出了以下變更：
* Enhanced Authentication (addition for IPMI-over-IP: RMCP+)
* VLAN Support
* Serial Over LAN (SOL, 已在 RMCP+ 的 new payload 中定義為 custom payload type)
* Payloads 
( RMCP+ 可以透過 IPMI-over-IP 會話傳輸 IMPI messages 以外的資料。
這包括為 SOL 指定的標準化 payload types 或其他 OEM 特定的「value-added」payload types )
* Encryption Support (可對可透過 RMCP+ 傳輸的 IPMI 訊息和其他有效負載類型進行加密)
* Extended User Login Options
* Firmware Firewall
* SMBus System Interface (SSIF 是硬體介面的新選項，允許透過 SMBus 主機控制器本機存取 BMC。 SSIF 應該有助於支援低成本的 BMC 實施)


<h2>
    Reference
</h2>

https://www.thomas-krenn.com/en/wiki/IPMI_Basics
https://github.com/openbmc/docs/blob/master/architecture/interface-overview.md
