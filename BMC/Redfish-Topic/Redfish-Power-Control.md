![Redfish_Power_Control.drawio](https://hackmd.io/_uploads/HyCvg0DS0.png)

![image](https://hackmd.io/_uploads/rJTRIRDHA.png)

<h2>
    Power control manager
</h2>

![image](https://hackmd.io/_uploads/H1b2dKtH0.png)

路徑應該是在這, 還沒有試過

```
meta-openbmc-mods/conf/machine/include/intel.inc
```

<h2>
    Meson
</h2>

Meson 旨在開發最具可用性和快速的建置系統。

提供簡單但強大的聲明式語言用來描述建置。

原生支援最新的工具和框架，如 Qt5 、程式碼覆蓋率、單元測試和預編譯頭檔等。

利用一組最佳化技術來快速編譯程式碼，包括增量編譯和完全編譯。

<h3>
    meson configure
</h3>

```
meson configure
```
![image](https://hackmd.io/_uploads/rkiGcFIB0.png)

![image](https://hackmd.io/_uploads/SJXJKKUBC.png)

![image](https://hackmd.io/_uploads/Hyl6utUSA.png)

![image](https://hackmd.io/_uploads/HkxGYFIH0.png)

<h2>
    phosphor-state-manager
</h2>

`phosphor-state-manager` 使用 D-Bus 介面來公開其功能，這些介面定義在 `phosphor-dbus-interfaces` 中。

And it provides unified interface to control and monitor the state of the BMC, HOST, and Chassis

`phosphor-state-manager` 的目標是控制和追蹤系統中不同軟體實體的狀態。

在基於 BMC 的伺服器中，**有三種狀態**需要追踪和控制。

這些狀態在 
`/xyz/openbmc_project/state/`+`/bmcX`, `/hostY`, `/chassisZ` 路徑下可以找到，其中 X, Y, Z 是實例（在大多數情況下為 0）。

對於這三種狀態，軟體會追踪當前狀態和請求的轉換。

**BMC**

The BMC would provide interfaces at `/xyz/openbmc_project/state/bmc<instance>`

**Host**

The Host would provide interfaces at `/xyz/openbmc_project/state/host<instance>`

**Chassis**

The Chassis would provide interfaces at `/xyz/openbmc_project/state/chassis<instance>`

![image](https://hackmd.io/_uploads/SJPeG9UBR.png)

<h2>
    /org/freedesktop/systemd1
</h2>

是 D-Bus 系統中的一個標準目錄，通常用來表示 systemd 的 DBus 接口。

這個目錄是 systemd 的 D-Bus 接口的一部分，允許其他應用程序通過 D-Bus 協議與 systemd 進行交互。

`/org/freedesktop/systemd1` 是 systemd 的主對象，包含了 systemd 服務的主要接口和方法。

這個目錄下的接口包含許多與系統狀態相關的信號（signals）和方法（methods）。

例如，**JobRemoved** 和 **JobNew** 信號是用來通知服務或目標狀態變化的。

<h3>
    Main interface
</h3>

* `org.freedesktop.systemd1.Manager`

這是最重要的接口，提供了管理 systemd 守護進程的各種方法

* `org.freedesktop.systemd1.Unit`

這個接口表示 systemd 中的單元（unit），如服務（service）、目標（target）等

* `org.freedesktop.systemd1.Service`

這是 Unit 的子接口，專門用於表示 systemd 中的服

<h3>
    JobNew 和 JobRemoved
</h3>

systemd 中，JobNew 和 JobRemoved 是兩個訊號，用來通知管理任務的狀態變更。

**信號發送情況：**

JobNew
* 當一個新的任務（job）被建立或提交到 systemd 時

JobRemoved
* 當一個任務（job）完成、停止或被刪除時

**典型用途：**

JobNew
* 在 systemd 啟動或停止服務時，JobNew 訊號用於通知 systemd 監視器或其他相關組件一個新的任務已被新增
* 例如，當你啟動一個服務時，systemd 會建立一個新的任務，並發送 JobNew 訊號。

JobRemoved
* 在任務完成或停止時，JobRemoved 訊號用於通知 systemd 監視器或其他元件任務已結束。
* 例如，當一個服務停止或失敗時，systemd 會傳送 JobRemoved 訊號。

**訊號內容：**

JobNew and JobRemoved

* 任務的 ID (jobID)
* 任務的路徑 (jobPath)
* 任務的單元名稱 (jobUnit)

<h2>
    Code
</h2>

<h3>
    host_state_manager_main.cpp
</h3>

```cpp=
#include "config.h"

#include "host_state_manager.hpp"

#include <sdbusplus/bus.hpp>

#include <cstdlib>
#include <exception>
#include <experimental/filesystem>
#include <iostream>

int main()
{
    namespace fs = std::experimental::filesystem;
    
    // 創建一個新的 D-Bus bus 連接。new_default 方法會使用預設的 D-Bus 環境進行初始化。
    auto bus = sdbusplus::bus::new_default();

    // For now, we only have one instance of the host
    auto objPathInst = std::string{HOST_OBJPATH} + '0';

    // 創建一個 ObjectManager 實例，用於管理這個 D-Bus bus 上的對象。
    sdbusplus::server::manager::manager objManager(bus, objPathInst.c_str());

    // 創建了 phosphor::state::manager::Host 的實例 manager
    phosphor::state::manager::Host manager(bus, objPathInst.c_str());

    // 這行代碼創建了一個 fs::path 對象 dir，它表示持久化目錄的父目錄路徑
    auto dir = fs::path(HOST_STATE_PERSIST_PATH).parent_path();
    
    // 這行代碼使用 fs::create_directories 函數創建了持久化目錄。如果該目錄已經存在，則不會執行任何操作；如果不存在，則會創建整個目錄結構
    fs::create_directories(dir);

    // 這行代碼通過 bus.request_name 函數註冊了一個 D-Bus 名稱，名稱由常量 HOST_BUSNAME 定義。
    bus.request_name(HOST_BUSNAME);

    // 這部分代碼構成了程序的主事件處理循環。它使用了 bus.process_discard() 函數來處理丟棄的消息，並通過 bus.wait() 函數等待新的消息或事件到來。
    while (true)
    {
        bus.process_discard();
        bus.wait();
    }
    return 0;
}
```

<h3>
    host_state_manager.hpp
</h3>

```cpp=
Host(sdbusplus::bus::bus& bus, const char* objPath) :
    HostInherit(bus, objPath, true), bus(bus),
    systemdSignalJobRemoved(
        bus,
        sdbusRule::type::signal() + sdbusRule::member("JobRemoved") +
            sdbusRule::path("/org/freedesktop/systemd1") +
            sdbusRule::interface("org.freedesktop.systemd1.Manager"),
        std::bind(std::mem_fn(&Host::sysStateChangeJobRemoved), this,
                  std::placeholders::_1)),
    systemdSignalJobNew(
        bus,
        sdbusRule::type::signal() + sdbusRule::member("JobNew") +
            sdbusRule::path("/org/freedesktop/systemd1") +
            sdbusRule::interface("org.freedesktop.systemd1.Manager"),
        std::bind(std::mem_fn(&Host::sysStateChangeJobNew), this,
                  std::placeholders::_1)),
    settings(bus)
{
    // Enable systemd signals
    subscribeToSystemdSignals();

    // Will throw exception on fail
    determineInitialState();

    attemptsLeft(BOOT_COUNT_MAX_ALLOWED);

    // We deferred this until we could get our property correct
    this->emit_object_added();
}
```

他做了下面幾點事情
* 初始化
* 綁定 systemd 的 JobRemoved 和 JobNew 信號，以便在系統狀態變化時更新主機狀態

**subscribeToSystemdSignals()**

訂閱 systemd 的信號，以便在相關狀態變化時接收通知。

![image](https://hackmd.io/_uploads/BkyoHJDrA.png)

**determineInitialState()**

這個方法會根據當前系統狀態設置初始的 host 狀態，並且會在初始化過程中檢查和設置必要的屬性。

![image](https://hackmd.io/_uploads/Sym3ryDBC.png)

**attemptsLeft(BOOT_COUNT_MAX_ALLOWED)**

將重啟計數設置為最大允許的重啟次數。這樣可以確保在系統啟動時，重啟計數被正確初始化。

![image](https://hackmd.io/_uploads/ByuTHkwH0.png)

**this->emit_object_added()**

這會將此對象註冊到 DBus 上，並通知其他服務此對象已經準備好。

<h3>
    CurrentState
</h3>

```
/home/tareokuo/AST26x0_OPF-RV22.11/build/intel-ast2600-evb/tmp/work/armv7ahf-vfpv4d16-openbmc-linux-gnueabi/mctp-emulator/1.0+gitAUTOINC+94437a678a-r0/recipe-sysroot/usr/include/xyz/openbmc_project/State/Host
```

server.hpp
![image](https://hackmd.io/_uploads/Sy0cV0IrA.png)

<h3>
    host_state_manager.cpp
</h3>

如果有偵測到 JobNew 變更 `systemdSignalJobNew` 就會執行 `sysStateChangeJobNew`

`if (newStateUnit == HOST_STATE_DIAGNOSTIC_MODE)`

![image](https://hackmd.io/_uploads/r17IIRUHR.png)

<h3>
    sysStateChangeJobNew and sysStateChangeJobRemoved
</h3>

**sysStateChangeJobNew**

![image](https://hackmd.io/_uploads/BJs8ykDB0.png)

如果 JobNew 信號有改變會執行這個 function, 他會去確認 JobNew 信號是不是與 HOST_STATE_DIAGNOSTIC_MODE 一樣, 如果是則調整 currentHostState

**sysStateChangeJobRemoved**

這個的部份跟上面差不多

<h2>
    Modify
</h2>

如果要修改 Host 的狀態會使用到這個 Action

`/redfish/v1/Systems/system/Actions/ComputerSystem.Reset/`

他對應到的 code function 是這個
`requestRoutesSystemActionsReset`

```cpp=
inline void requestRoutesSystemActionsReset(App& app)
{
    /**
     * Function handles POST method request.
     * Analyzes POST body message before sends Reset request data to D-Bus.
     */
    BMCWEB_ROUTE(app,
                 "/redfish/v1/Systems/system/Actions/ComputerSystem.Reset/")
        .privileges(redfish::privileges::postComputerSystem)
        .methods(
            boost::beast::http::verb::
                post)([](const crow::Request& req,
                         const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) {
            std::string resetType;
            if (!json_util::readJson(req, asyncResp->res, "ResetType",
                                     resetType))
            {
                return;
            }

            // Get the command and host vs. chassis
            std::string command;
            bool hostCommand;
            if ((resetType == "On") || (resetType == "ForceOn"))
            {
                command = "xyz.openbmc_project.State.Host.Transition.On";
                hostCommand = true;
            }
            else if (resetType == "ForceOff")
            {
                command = "xyz.openbmc_project.State.Chassis.Transition.Off";
                hostCommand = false;
            }
            else if (resetType == "ForceRestart")
            {
                command =
                    "xyz.openbmc_project.State.Host.Transition.ForceWarmReboot";
                hostCommand = true;
            }
            else if (resetType == "GracefulShutdown")
            {
                command = "xyz.openbmc_project.State.Host.Transition.Off";
                hostCommand = true;
            }
            else if (resetType == "GracefulRestart")
            {
                command = "xyz.openbmc_project.State.Host.Transition."
                          "GracefulWarmReboot";
                hostCommand = true;
            }
            else if (resetType == "PowerCycle")
            {
                command = "xyz.openbmc_project.State.Host.Transition.Reboot";
                hostCommand = true;
            }
            else if (resetType == "Nmi")
            {
                doNMI(asyncResp);
                return;
            }
            else
            {
                messages::actionParameterUnknown(asyncResp->res, "Reset",
                                                 resetType);
                return;
            }

            if (hostCommand)
            {
                crow::connections::systemBus->async_method_call(
                    [asyncResp, resetType](const boost::system::error_code ec) {
                        if (ec)
                        {
                            BMCWEB_LOG_ERROR << "D-Bus responses error: " << ec;
                            if (ec.value() ==
                                boost::asio::error::invalid_argument)
                            {
                                messages::actionParameterNotSupported(
                                    asyncResp->res, resetType, "Reset");
                            }
                            else
                            {
                                messages::internalError(asyncResp->res);
                            }
                            return;
                        }
                        messages::success(asyncResp->res);
                    },
                    "xyz.openbmc_project.State.Host",
                    "/xyz/openbmc_project/state/host0",
                    "org.freedesktop.DBus.Properties", "Set",
                    "xyz.openbmc_project.State.Host", "RequestedHostTransition",
                    std::variant<std::string>{command});
            }
            else
            {
                crow::connections::systemBus->async_method_call(
                    [asyncResp, resetType](const boost::system::error_code ec) {
                        if (ec)
                        {
                            BMCWEB_LOG_ERROR << "D-Bus responses error: " << ec;
                            if (ec.value() ==
                                boost::asio::error::invalid_argument)
                            {
                                messages::actionParameterNotSupported(
                                    asyncResp->res, resetType, "Reset");
                            }
                            else
                            {
                                messages::internalError(asyncResp->res);
                            }
                            return;
                        }
                        messages::success(asyncResp->res);
                    },
                    "xyz.openbmc_project.State.Chassis",
                    "/xyz/openbmc_project/state/chassis0",
                    "org.freedesktop.DBus.Properties", "Set",
                    "xyz.openbmc_project.State.Chassis",
                    "RequestedPowerTransition",
                    std::variant<std::string>{command});
            }
        });
}
```

他一開始會先去判斷你輸入的是關於 Host 的還是 Chassis 的, 如果是 Host 的就會去跟 Dbus 溝通

假如有修改成功後面會去 call

![image](https://hackmd.io/_uploads/B1TdNCwrA.png)

`executeTransition(value)` 他會透過 Dbus 跟 systemd 他說要做什麼動作

```
curl -k -u root:0penBmc -X POST https://127.0.0.1/redfish/v1/Systems/system/Actions/ComputerSystem.Reset -H "Content-Type: application/json" -d '{"ResetType": "On"}'
```

<h2>
    Redfish Power Control Chassis
</h2>

sever.hpp
![image](https://hackmd.io/_uploads/rJdLBz_rC.png)

Chassis 的 Off 在 System.hpp 中, 要在這個 Action 使用
`/redfish/v1/Systems/system/Actions/ComputerSystem.Reset/`

chassis.hpp 內的 Action 只有提供 PowerCycle 功能
`/redfish/v1/Chassis/<str>/Actions/Chassis.Reset/`

![Redfish-Dbus-Chassis.drawio](https://hackmd.io/_uploads/rJ-nOMdSC.png)

```
curl -k -u root:0penBmc -X POST https://127.0.0.1/redfish/v1/Chassis/EVB_AST2600_Baseboard/Actions/Chassis.Reset -H "Content-Type: application/json" -d '{"ResetType": "PowerCycle"}'
```

<h2>
    Other Command
</h2>

Check the currently active target
```
systemctl list-units --type=target --state=active
```

<h2>
    Reference
</h2>

[OpenBMC & Systemd](https://github.com/openbmc/docs/blob/master/architecture/openbmc-systemd.md)

[BMC, Host, and Chassis State Management](https://github.com/openbmc/phosphor-dbus-interfaces/blob/master/yaml/xyz/openbmc_project/State/README.md)

[Phosphor State Manager Documentation](https://github.com/openbmc/phosphor-state-manager)