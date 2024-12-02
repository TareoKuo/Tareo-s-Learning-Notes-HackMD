<h2>
    Topic
</h2>

[TOC]

<h2>
    SMBIOS
</h2>

[Reference <- 大部分資料來源都在這](<https://iris123321.blogspot.com/2022/02/smbios.html>)
[Reference2 <- 大部分資料來源都在這](<https://iris123321.blogspot.com/2022/02/openbmc-smbios-mdr.html>)

SMBIOS (System Management BIOS) 是一個標準 由 DMTF所開發，這個標準的目的是允許作業系統去獲取電腦的資訊。

在設定 SMBIOS 方面，我們會在 Memory 的某個地方建立一個 Table. 經由解析這個 Table，它是可以存取電腦資訊和電腦屬性。

SMBIOS 在 BIOS 會設定之，在 POST 階段於各硬體 Module 會依據不同的 type 存於 Memory 中。

<h3>
    SMBIOS Entry Point Table
</h3>

SMBIOS Entry Point Table 的位置在 **Memory 0xF0000 和 0xFFFFF**, 且必需為 16-byte boundary，去搜尋其表格的開始於特定的位置，是需要在 Memory 搜尋字串 **"_SM_"**, 且核對結構的 checksum.

![image](https://hackmd.io/_uploads/HkZlldEUA.png)

如果是 **UEFI-based** 系統可以直接搜尋GUID找到
（SMBIOS_TABLE_GUID， **{EB9D2D31-2D88-11D3-9A16-0090273FC14D}**）

![image](https://hackmd.io/_uploads/rkLJg_4L0.png)

<h3>
    Entry point table structure
</h3>

```
 struct SMBIOSEntryPoint {
  char EntryPointString[4];    //This is _SM_
  uchar Checksum;              //This value summed with all the values of the table, should be 0 (overflow)
  uchar Length;                //Length of the Entry Point Table. Since version 2.1 of SMBIOS, this is 0x1F
  uchar MajorVersion;          //Major Version of SMBIOS
  uchar MinorVersion;          //Minor Version of SMBIOS
  ushort MaxStructureSize;     //Maximum size of a SMBIOS Structure (we will se later)
  uchar EntryPointRevision;    //...
  char FormattedArea[5];       //...
  char EntryPointString2[5];   //This is _DMI_
  uchar Checksum2;             //Checksum for values from EntryPointString2 to the end of table
  ushort TableLength;          //Length of the Table containing all the structures
  uint TableAddress;      //Address of the Table
  ushort NumberOfStructures;   //Number of structures in the table
  uchar BCDRevision;           //Unused
 };
```

<h3>
    Header Types
</h3>

```
 struct SMBIOSHeader {
  uchar Type;
  uchar Length;
  ushort Handle;
 };
```

<h2>
    OpenBMC SMBIOS MDR
</h2>

MDR（Managed Data Region） 是 OpenBMC 中取得 SMBIOS Table 並解析其內容的一個功能

MDR 是一種通用機制，用於管理韌體實體之間的數據傳輸並在 BMC 中維護伺服器數據。 
目前，該數據由 `BCT/SMBIOS` 數據組成。 但將來可能會添加其他數據實體。

[openbmc/smbios-mdr](https://github.com/openbmc/smbios-mdr)

<h3>
    Use
</h3>

確定好結構類型後，找到相對應的表，填入相對應的值，如果值的類型是STRING，就填入是未格式化區域中的第幾個字串，如果沒有字串，就填0

![image](https://hackmd.io/_uploads/B1OjUd48A.png)

可以定義結構，套上去就能處理的值

![image](https://hackmd.io/_uploads/HkzALdV8R.png)

<h3>
    Work Flow
</h3>

BIOS 將 smbios table 透過一些方式傳給 BMC 後，BMC 呼叫 dbus method "AgentSynchronizeData" ，smbios-mdr 會去將 smbios 資料 expose 到 dbus 上，供使用者讀取和操作

>類似放在 `/var/lib/smbios/smbios2` 的位子

```c=
// github.com/openbmc/intel-ipmi-oem/blob/master/src/smbiosmdrv2handler.cpp
ipmi::RspType<> cmd_mdr2_data_done(uint16_t agentId, uint16_t lockHandle)
{
    // ...
    sdbusplus::message::message method = bus->new_method_call(
        service.c_str(), mdrv2Path, mdrv2Interface, "AgentSynchronizeData");
    // ...
}
 
// github.com/openbmc/smbios-mdr/blob/master/src/smbios-ipmi-blobs/handler.cpp
bool syncSmbiosData()
{
    // ...
    sdbusplus::message::message method =
        bus.new_method_call(mdrV2Service, phosphor::smbios::mdrV2Path,
                            mdrV2Interface, "AgentSynchronizeData");
    // ...
}
```

![image](https://hackmd.io/_uploads/SkLHwONUA.png)

**CMM**：Chassis Management Module
**BMC**：Baseboard Management Controller
**BIOS**：Basic Input/Output System

<h3>
    Transmission Method
</h3>

* **MDRv1** intel-ipmi-oem/smbioshandler.cpp
* **MDRv2** (**Pull**)   intel-ipmi-oem/smbiosmdrv2handler.cpp
* **MDRv2** (**Push**)   intel-ipmi-oem/smbiosmdrv2handler.cpp
* **IPMI blobs** smbios-mdr/handler.cpp

這四種方法最後都是將 SMBIOS 放到 /var/lib/smbios 底下，呼叫 smbios-mdr 去做解析並將資料放到 dbus 上

MDR V1 是 BIOS 透過指令將 SMBIOS 慢慢送過去給 BMC

MDR V2 是 BIOS 將 SMBIOS 放到 **VGA share memory**，BMC再去讀出來
只是*會根據主動發送指令的是 BMC 還是 BIOS 分成 Pull 和 Push 兩種*

IPMI Blobs 的方式就是將 SMBIOS 視為一個 Blobs 傳送給 BMC

![image](https://hackmd.io/_uploads/SJCNddNLA.png)


>smbios-mdr 中還有另一個服務 **"xyz.openbmc_project.cpuinfo.service"**。 
>
>它的作用是根據 SMBIOS 中的 Processor Type 和 Entity manager 的 config 文件，
>通過 PIROM 訪問 CPU，確認 CPU ID 合法 ，如果沒有問題，才將 CPU 資訊曝光到 dbus 上

```
# Entity Manager config / baseboard.json
{
	"Address": "0x30",
	"Bus": 0,
	"CpuID": 1,
	"Name": "CPU 1",
	"PiromI2cAddress": "0x50",
	"PiromI2cBus": 12,
	"PresenceGpio": [
	    {
	        "Name": "CPU1_PRESENCE",
	        "Polarity": "Low"
	    }
	],
},
```

<h2>
    SMBIOS MDRv2 傳輸方式
</h2>

![image](https://hackmd.io/_uploads/Hyey-anD0.png)

在 linux-dts 的部分需要填好, 如果使用 MDRv2 的話

```
vga-shared-memory {
    compatible = "aspeed,ast2500-vga-sharedmem";
    reg = <0x9f700000 0x100000>;
};
```

`reg = <0x9f700000 0x100000>`

`0x9f700000` 代表起始位址, `0x100000` 代表內存區域大小

>起始位址 `0x9f700000` 是裝置暫存器或記憶體對映的起點。
>`0x100000` 表示這段記憶體區域的大小是 1 MB。
>這個裝置佔用了從 `0x9f700000` 開始的 1 MB 記憶體區域。設備樹將此訊息傳遞給內核

<h2>
    openbmc/smbios-mdr
</h2>

[openbmc/smbios-mdr](https://github.com/openbmc/smbios-mdr)

SMBIOS Parser

此儲存庫中的主要應用程式是 `smbiosmdrv2app`，能夠解析二進位 SMBIOS 表並在 D-Bus 上發布系統信息，以供其他 OpenBMC 應用程式使用。

SMBIOS 表通常由主機韌體 (BIOS) 傳送到 BMC。

理論上，系統設計者可以選擇任何傳輸方式和機制來傳送 SMBIOS 數據

目前至少有兩種實作方式：

**MDRv2**

主要 API 是一組稱為託管資料區域版本 2 (MDRv2) 的 Intel OEM IPMI 命令，它為主機韌體提供了一種**透過 VGA 共享記憶體區域發送資料**的方法。 

MDRv2 有多個代理的概念，每個代理程式維護一個包含目錄條目（也稱為資料集）的「目錄」。

主機可以查詢目錄是否存在及其版本，以確定何時需要發送更新的 SMBIOS 表。

`intel-ipmi-oem` 實作 IPMI 命令處理程序，將命令和資料路由到正確的代理
（例如 `smbios-mdr`）。

IPMI 處理程序和 smbios-mdr 之間的 D-Bus 介面很大程度上是 IPMI 指令的鏡像。

**phosphor-ipmi-blobs**

`Phosphor-ipmi-blobs` 是通用 IPMI blob 傳輸 API 的替代實作。

與 MDRv2 相比，它更簡單、更易於使用，但也使用 IPMI 命令在帶內傳輸數據，因此比使用共享記憶體區域慢（這可能是也可能不是問題）。

Phosphor-ipmi-blobs 為 ipmid 提供了一個 Blob 管理器共用函式庫，它實作了 IPMI 指令。

反過來，它會載入 Blob 處理程序庫，每個庫都實作對特定 Blob 的支援。

在 `smbios-mdr` 中，我們為 /smbios blob 提供了這樣的 blob 處理程序。
它的工作原理是將資料寫入 `/var/lib/smbios/smbios2` （SMBIOS 表的本地持久性快取）並呼叫 AgentSynchronizeData D-Bus 方法來觸發 smbios-mdr 從該檔案重新載入和解析該表。

<h3>
    Intel CPU Info
</h3>

`cpuinfoapp` 是一個特定於 Intel 的應用程序，它使用 I2C 和 PECI 來收集有關因某種原因未包含在 SMBIOS 表中的 Xeon CPU 的更多詳細資訊。

它還實現了英特爾速度選擇技術 (SST) 的發現和控制。