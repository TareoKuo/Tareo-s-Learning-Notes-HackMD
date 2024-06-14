<h2>
    Redfish Dbus
</h2>

[Reference](<https://iris123321.blogspot.com/2022/04/openbmc-dbus.html>)

OpenBMC uses D-Bus as an inter-process communication (IPC).

![image](https://hackmd.io/_uploads/B1HkGDSrC.png)

![image](https://hackmd.io/_uploads/rJlgfvSSA.png)

**sd-bus** : sd-bus 是最小的 D-Bus IPC C library，和systemd 一起released 

**sdbusplus & sdbus++** : sdbusplus 是用於與 D-Bus 交互的 C++ library (libsdbusplus)，構建在 systemd 的 sd-bus library之上，這樣C++可以直接呼叫API，更方便好用

另外 **sdbus++** 是一個生成 C++ 綁定的工具，以簡化基於 D-Bus 的應用程序的開發，搭配 phosphor-dbus-interfaces 使用

<h3>
    The Object Mapper
</h3>

這個很像BMC內部小型的database，主要兩個功能是

* Methods : 提供 D-Bus 發現相關功能。
* Associations : 將兩個不同的對象相互關聯。

object mapper 會監測 dbus 上的動作

```
GetObject : 查找實現特定object path的service及其interface。
GetSubTree: 在指定的subtree中查找實現某個 interface 的object、service 和 interface。
GetSubTreePaths: 這與 GetSubTree 相同，但只返回object path。
GetAncestors: 查找實現特定interface的object的所有祖先。

如果需要讀值，還是要透過 dbus api 去讀取
```

<h3>
    Redfish and Dbus
</h3>

![image](https://hackmd.io/_uploads/SyVsZ5Br0.png)

<h3>
    Tree
</h3>

![image](https://hackmd.io/_uploads/rk15oqHNA.png)


<h3>
    Makefile
</h3>

```makefile=
SRC = main.cpp
OUTPUT=dbus-object
CFLAGS+=-O3 -Wno-unused-result
LIB = -lsdbusplus -lsystemd


all: $(SRC)
	$(CXX) $(SRC) -o  $(OUTPUT) $(CFLAGS) $(LIB)
clean:
	-rm -rf $(OUTPUT)
```

<h3>
    dbus-object.bb
</h3>

```bb=
FILESEXTRAPATHS:prepend := "${THISDIR}/${PN}:"
DESCRIPTION = "Test app for dbus object"
LICENSE = "CLOSED"
TARGET_CC_ARCH += "${LDFLAGS}"

SRC_URI += " \
            file://main.cpp \
            file://Makefile \
          "
S = "${WORKDIR}"

DEPENDS += " boost sdbusplus systemd"
inherit pkgconfig obmc-phosphor-systemd

do_compile () {
    oe_runmake
}

do_install () {
    install -d ${D}${bindir}/
    install -m 0755 ${S}/dbus-object ${D}${bindir}/
}

SYSTEMD_SERVICE:${PN} += "dbus-object.service"
```

<h3>
    dbus-object.service
</h3>

```service=
[Unit]
Description=Test Dbus Object Service
[Service]
Type=dbus
BusName=xyz.openbmc_project.TareoDbusTest
ExecStart=/usr/bin/dbus-object
[Install]
WantedBy=multi-user.target
```

<h3>
    main.cpp
</h3>

```cpp=
#include <boost/asio.hpp>
#include <sdbusplus/asio/connection.hpp>
#include <sdbusplus/asio/object_server.hpp>
#include <sdbusplus/bus.hpp>
#include <sdbusplus/message.hpp>
#include <sdbusplus/server.hpp>
#include <iostream>


constexpr auto busName = "xyz.openbmc_project.TareoDbusTest";
constexpr auto objPath = "/xyz/openbmc_project/TareoDbusTest";
constexpr auto ifcName = "xyz.openbmc_project.TareoDbusTest";

uint32_t id = 37;
std::string name = "Tareo";
std::string status = "Enabled";

auto helloIndex = 0;
boost::container::flat_map<std::string, std::shared_ptr<sdbusplus::asio::dbus_interface>> helloList;


int main(int argc, char*argv[])
{
    // Create io_context object. Used for managing I/O operations
    boost::asio::io_context ioCtx;
    // Create sdbusplus::asio::connection object. Used for establishing a connection with D-Bus
    std::shared_ptr<sdbusplus::asio::connection> conn = std::make_shared<sdbusplus::asio::connection>(ioCtx);
    // Request D-Bus name
    conn->request_name(busName);
    // Create D-bus object Service
    sdbusplus::asio::object_server hostServer = sdbusplus::asio::object_server(conn);
    // add interface
    std::shared_ptr<sdbusplus::asio::dbus_interface> interface = hostServer.add_interface(objPath, ifcName);
    
    
    interface->register_property(
        "Id",
        id,
        sdbusplus::asio::PropertyPermission::readWrite);
        
    interface->register_property(
        "Name",
        name,
        sdbusplus::asio::PropertyPermission::readOnly);
        
    interface->register_property(
        "Status",
        status,
        [&](const std::string &requested, std::string &resp) -> int
        {
            if(requested == "0" || requested == "Disabled") {
                resp = "Disabled";
            } else if (requested == "1" || requested == "Enabled") {
                resp = "Enabled";
            } else {
                return 0;
            }
            return 1;
        },
        [&](std::string &resp) -> std::string
        {
            return resp;
        });
        
    interface->register_method("CreateHello",
        [&](std::string helloId)
        {
            if(helloId.empty() || helloIndex == 3) {
                return -1;
            }
            try {
                // new object
                std::string hellopath = std::string(objPath) + "/" + helloId;
                auto hello_interface = hostServer.add_interface(hellopath, ifcName);
                uint32_t num = static_cast<uint32_t>(std::stoul(helloId));
                hello_interface->register_property("Id", num, sdbusplus::asio::PropertyPermission::readWrite);
                hello_interface->register_property("Name", name, sdbusplus::asio::PropertyPermission::readOnly);
                hello_interface->register_property("Status", status, sdbusplus::asio::PropertyPermission::readWrite);
                hello_interface->initialize();
                helloList.insert(std::make_pair(hellopath, hello_interface));
                helloIndex++;
                return helloIndex;
            }
            catch(...) {
                return -1;
            }
        });
        
    interface->register_method("DeleteHello",
        [&](std::string delpath)
        {
            try {
                boost::container::flat_map<std::string, 
                std::shared_ptr<sdbusplus::asio::dbus_interface>>::const_iterator it = helloList.find(delpath);
                if(helloList.end() != it) { // 如果使用 find(), 他有找到會 return 位置, 沒找到會返回 .end()
                    std::shared_ptr<sdbusplus::asio::dbus_interface> interfaceHello = it->second;
                    if (nullptr != interfaceHello) {
                        bool err = hostServer.remove_interface(interfaceHello);
                        if (!err) {
                            printf("Fail to call remove_interface()!");
                        }
                        else {
                            helloList.erase(delpath);
                            if(helloIndex > 0) {
                                helloIndex--;
                            }
                        }
                    }
                }
                return helloIndex;
            }
            catch(...) {
                return -1;
            }
        });
    
    // initialize interface
    interface->initialize();
    // run ioCtx
    ioCtx.run();
    return 0;
}
```

<h2>
    Related Command
</h2>

<h3>
    register_method
</h3>

```cpp=
// 註冊dbus方法,方法名為 MyTestMethodAddInt， 接受兩個參數 ，參數1型別為int，參數2為int,傳回int
interface->register_method("DemoMethodAddInt", [&](const int &a, const int &b) -> int
                           { return a + b; });
// 註冊dbus方法,方法名為 MyTestMethodAddInt， 接受兩個參數 ，參數1型別為string，參數2為string,回傳string
interface->register_method("DemoMethodAddString", [&](const std::string &s1, const std::string &s2) -> std::string
                           { return s1 + s2; });
```

<h3>
    new_method_call
</h3>

new_method_call 用於建立一個方法呼叫訊息。

這些訊息通常用於請求服務執行某個操作（例如取得屬性值、呼叫方法等）。

建立的訊息包含目標服務的名稱、目標物件路徑、目標介面以及方法名稱。

```cpp
sdbusplus::message::message method = bus.new_method_call(
    const char* service,       // 服务名
    const char* objectPath,    // 对象路径
    const char* interface,     // 接口名
    const char* methodName     // 方法名
);
```

Example : 
```cpp
#include <sdbusplus/bus.hpp>
#include <sdbusplus/message.hpp>

int main()
{
    // 创建一个默认的 D-Bus 总线连接
    sdbusplus::bus::bus bus = sdbusplus::bus::new_default();

    // 创建一个方法调用消息，调用某个服务的属性获取方法
    sdbusplus::message::message method = bus.new_method_call(
        "org.freedesktop.DBus",          // 服务名
        "/org/freedesktop/DBus",         // 对象路径
        "org.freedesktop.DBus.Properties",  // 接口名
        "Get"                            // 方法名
    );

    // 将需要获取的属性传递给方法调用
    method.append("org.freedesktop.DBus", "SomeProperty");

    // 调用方法并处理返回值
    try
    {
        sdbusplus::message::message reply = bus.call(method);
        std::string value;
        reply.read(value);
        std::cout << "Property Value: " << value << std::endl;
    }
    catch (const std::exception& e)
    {
        std::cerr << "Error calling method: " << e.what() << std::endl;
    }

    return 0;
}
```

Step : 
```
建立訊息：呼叫 new_method_call 方法建立一個新的方法呼叫訊息物件。
附加參數：使用 append 方法將參數新增至訊息。例如，指定要取得的屬性名。
傳送呼叫：使用 bus.call(method) 方法傳送方法呼叫請求，並取得傳回的訊息。
處理回覆：解析回覆訊息中的數據，通常使用 read 方法讀取傳回的參數值。
```
