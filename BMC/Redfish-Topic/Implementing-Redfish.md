<h2>
    Output
</h2>

![image](https://hackmd.io/_uploads/HyTxSRTXC.png)

![image](https://hackmd.io/_uploads/ryZQBCpQ0.png)

<h2>
    Command
</h2>

```
curl -k -u root:0penBmc -X GET https://192.168.9.227/redfish/v1/Oem/TareoService/TareoHello | jq .
```


<h2>
    Modified file.
</h2>

Create : 
* TareoHello_v1.xml
* TareoHelloCollection_v1.xml (optional)
* tareo_hello.hpp

Modify
* redfish.hpp
* privilege_registery.hpp
* service_root.hpp

>Dbus 的部分請看 Redfish Dbus Pages

要注意 Dbus 的部分
```cpp
    crow::connections::systemBus->async_method_call(
        [](const boost::system::error_code ec)
        {
            if(ec) {
                BMCWEB_LOG_DEBUG << "[Set] Bad D-Bus request error: " << ec;
            }
        },
        "xyz.openbmc_project.checkobmc", "/xyz/openbmc_project/checkobmc",
        "org.freedesktop.DBus.Properties", "Set",
        "xyz.openbmc_project.checkobmc", "solconsole", std::variant<bool>(true));
```

Set 的部分需要使用 `std::variant<bool>(true)` 才會成功

>D-Bus 的 Set 方法要求參數是某一種特定的類型。這些參數是基於 DBus 標準類型來處理的，在 OpenBMC 中，這些類型通常使用 `std::variant` 來封裝，以便能夠處理不同的數據類型。
>
>D-Bus 在處理複雜類型（如 bool、int、string 等）時，通常會使用 `std::variant`，這樣就能夠將多個不同的數據類型安全地傳遞到方法中。

<h3>
    TareoHello_v1.xml
</h3>

```xml=
<?xml version="1.0" encoding="UTF-8"?>
<!---->
<!--###############################################################################-->
<!-- Redfish Schema: SchemaName v1.0.0 -->
<!-- -->
<!-- Copyright 2019 DMTF. -->
<!-- For the full DMTF copyright policy, see http://www.dmtf.org/about/policies/copyright -->
<!--############################################################################### -->
<!---->
<edmx:Edmx xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" Version="4.0">
<!-- This section consists of references to standard Redfish Schema bits. -->
<!-- Do not modify these -->
  <edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/errata03/csd01/complete/vocabularies/Org.OData.Core.V1.xml">
    <edmx:Include Namespace="Org.OData.Core.V1" Alias="OData"/>
  </edmx:Reference>
  <edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/errata03/csd01/complete/vocabularies/Org.OData.Capabilities.V1.xml">
    <edmx:Include Namespace="Org.OData.Capabilities.V1" Alias="Capabilities"/>
  </edmx:Reference>
  <edmx:Reference Uri="http://redfish.dmtf.org/Schemas/v1/Resource_v1.xml">
    <edmx:Include Namespace="Resource"/>
    <edmx:Include Namespace="Resource.v1_0_0"/>
  </edmx:Reference>
  <edmx:Reference Uri="http://redfish.dmtf.org/Schemas/v1/RedfishExtensions_v1.xml">
    <edmx:Include Namespace="RedfishExtensions.v1_0_0" Alias="Redfish"/>
    <edmx:Include Namespace="Validation.v1_0_0" Alias="Validation"/>
  </edmx:Reference>
  <edmx:Reference Uri="/redfish/v1/schema/TareoHelloCollection_v1.xml">
    <edmx:Include Namespace="TareoHelloCollection"/>
  </edmx:Reference>
<!-- You can add your own schema-specific references here -->


<!-- This section contains all the data properties for your schema -->
  <edmx:DataServices>

    <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="TareoHello">
      <Annotation Term="Redfish.OwningEntity" String="DMTF"/>

      <EntityType Name="TareoHello" BaseType="Resource.v1_0_0.Resource" Abstract="true">
        <Annotation Term="OData.Description" String="The TareoHello schema describes about Tareo hello."/>
        <Annotation Term="OData.LongDescription" String="This resource shall represent a TareoHello for a Redfish implementation."/>
        <Annotation Term="Capabilities.InsertRestrictions"> <!-- POST -->
          <Record>
            <PropertyValue Property="Insertable" Bool="false"/> 
          </Record>
        </Annotation>
        <Annotation Term="Capabilities.UpdateRestrictions"> <!-- PATCH -->
          <Record>
            <PropertyValue Property="Updatable" Bool="true"/>
          </Record>
        </Annotation>
        <Annotation Term="Capabilities.DeleteRestrictions"> <!-- DELETE -->
          <Record>
            <PropertyValue Property="Deletable" Bool="false"/>
          </Record>
        </Annotation>
        <Annotation Term="Redfish.Uris">
          <Collection>
            <String>/redfish/v1/Oem/TareoService/TareoHello/{TareoHelloId}</String>
          </Collection>
        </Annotation>
      </EntityType>

    </Schema>

    <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="TareoHello.v1_0_0">
        <Annotation Term="Redfish.OwningEntity" String="DMTF"/>

        <EntityType Name="TareoHello" BaseType="TareoHello.TareoHello">

            <Property Name="Hello" Type="Collection(TareoHello.v1_0_0.Hello)" Nullable="false">
              <Annotation Term="OData.Description" String="The status and health of the resource and its subordinate or dependent resources."/>
              <Annotation Term="OData.LongDescription" String="This property shall contain any status or health properties of the resource."/>
            </Property>

        </EntityType>

        <ComplexType Name="Hello">
          <Annotation Term="OData.AdditionalProperties" Bool="false"/>
          <Annotation Term="OData.Description" String="Hello Test."/>
          <Annotation Term="OData.LongDescription" String="Hello Test Hello Test Hello Test Hello Test."/>

          <Property Name="Id" Type="Edm.String" Nullable="false">
            <Annotation Term="OData.Permissions" EnumMember="OData.Permission/ReadWrite"/>
            <Annotation Term="OData.Description" String="TEST ID."/>
            <Annotation Term="OData.LongDescription" String="TEST ID TEST ID TEST ID TEST ID."/>
            <Annotation Term="Validation.Minimum" Int="0"/>
            <Annotation Term="Validation.Manimum" Int="100"/>
          </Property>

          <Property Name="Name" Type="Edm.String" Nullable="false">
            <Annotation Term="OData.Permissions" EnumMember="OData.Permission/Read"/>
            <Annotation Term="OData.Description" String="The name of the resource or array member."/>
            <Annotation Term="OData.LongDescription" String="This object represents the name of this resource or array member. The resource values shall conform with the Redfish Specification-described requirements. This string value shall be of the 'Name' reserved word format."/>
            <Annotation Term="Redfish.Required"/>
          </Property>

          <Property Name="Status" Type="Resource.Status" Nullable="false">
            <Annotation Term="OData.Permissions" EnumMember="OData.Permission/ReadWrite"/>
            <Annotation Term="OData.Description" String="The status and health of the resource and its subordinate or dependent resources."/>
            <Annotation Term="OData.LongDescription" String="This property shall contain any status or health properties of the resource."/>
          </Property>

        </ComplexType>

        <Action Name="SayHello" IsBound="true"> <!-- IsBound : 此屬性指定操作是否綁定到實體類型 -->
          <Annotation Term="OData.Description" String="This action to say hello."/>
          <Annotation Term="OData.LongDescription" String="This action will change some hello properties."/>
          <Parameter Name="TareoHello" Type="TareoHello.v1_0_0.Actions" />

          <Parameter Term="Id" Type="Edm.String" Nullable="false">
            <Annotation Term="OData.Description" String="TEST ID."/>
            <Annotation Term="OData.LongDescription" String="TEST ID TEST ID TEST ID TEST ID."/>
            <Annotation Term="Validation.Minimum" Int="0"/>
            <Annotation Term="Validation.Manimum" Int="100"/>
          </Parameter>

          <Parameter Term="Name" Type="Edm.String" Nullable="false">
            <Annotation Term="OData.Description" String="The name of the resource or array member."/>
            <Annotation Term="OData.LongDescription" String="This object represents the name of this resource or array member. The resource values shall conform with the Redfish Specification-described requirements. This string value shall be of the 'Name' reserved word format."/>
          </Parameter>

          <Parameter Term="Status" Type="Edm.String" Nullable="false">
            <Annotation Term="OData.Description" String="The status and health of the resource and its subordinate or dependent resources."/>
            <Annotation Term="OData.LongDescription" String="This property shall contain any status or health properties of the resource."/>
          </Parameter>

        </Action>

    </Schema>

  </edmx:DataServices>
</edmx:Edmx>
```

<h3>
    TareoHelloCollection_v1.xml
</h3>

```xml=
<?xml version="1.0" encoding="UTF-8"?>
<!---->
<!--###############################################################################-->
<!-- Redfish Schema: SchemaName v1.0.0 -->
<!-- -->
<!-- Copyright 2019 DMTF. -->
<!-- For the full DMTF copyright policy, see http://www.dmtf.org/about/policies/copyright -->
<!--############################################################################### -->
<!---->
<edmx:Edmx xmlns:edmx="http://docs.oasis-open.org/odata/ns/edmx" Version="4.0">
<!-- This section consists of references to standard Redfish Schema bits. -->
<!-- Do not modify these -->
  <edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/errata03/csd01/complete/vocabularies/Org.OData.Core.V1.xml">
    <edmx:Include Namespace="Org.OData.Core.V1" Alias="OData"/>
  </edmx:Reference>
  <edmx:Reference Uri="http://docs.oasis-open.org/odata/odata/v4.0/errata03/csd01/complete/vocabularies/Org.OData.Capabilities.V1.xml">
    <edmx:Include Namespace="Org.OData.Capabilities.V1" Alias="Capabilities"/>
  </edmx:Reference>
  <edmx:Reference Uri="http://redfish.dmtf.org/Schemas/v1/Resource_v1.xml">
    <edmx:Include Namespace="Resource"/>
    <edmx:Include Namespace="Resource.v1_0_0"/>
  </edmx:Reference>
  <edmx:Reference Uri="http://redfish.dmtf.org/Schemas/v1/RedfishExtensions_v1.xml">
    <edmx:Include Namespace="RedfishExtensions.v1_0_0" Alias="Redfish"/>
    <edmx:Include Namespace="Validation.v1_0_0" Alias="Validation"/>
  </edmx:Reference>
  <edmx:Reference Uri="/redfish/v1/schema/TareoHello_v1.xml">
    <edmx:Include Namespace="TareoHello"/>
  </edmx:Reference>
<!-- You can add your own schema-specific references here -->


<!-- This section contains all the data properties for your schema -->
  <edmx:DataServices>

    <Schema xmlns="http://docs.oasis-open.org/odata/ns/edm" Namespace="TareoHelloCollection">
      <Annotation Term="Redfish.OwningEntity" String="DMTF"/>

      <EntityType Name="TareoHelloCollection" BaseType="Resource.v1_0_0.Resource" Abstract="true">
        <Annotation Term="OData.Description" String="The collection of Hello resource instances."/>
        <Annotation Term="OData.LongDescription" String="This resource shall represent a resource collection of TareoHello instances."/>
        <Annotation Term="Capabilities.InsertRestrictions"> <!-- POST -->
            <Record>
              <PropertyValue Property="Insertable" Bool="false"/> 
            </Record>
          </Annotation>
          <Annotation Term="Capabilities.UpdateRestrictions"> <!-- PATCH -->
            <Record>
              <PropertyValue Property="Updatable" Bool="true"/>
            </Record>
          </Annotation>
          <Annotation Term="Capabilities.DeleteRestrictions"> <!-- DELETE -->
            <Record>
              <PropertyValue Property="Deletable" Bool="false"/>
            </Record>
          </Annotation>
        <Annotation Term="Redfish.Uris">
          <Collection>
            <String>/redfish/v1/Oem/TareoService/TareoHello</String>
          </Collection>
        </Annotation>
        <NavigationProperty Name="Members" Type="Collection(TareoHello.TareoHello)">
            <Annotation Term="OData.Permissions" EnumMember="OData.Permission/Read"/>
            <Annotation Term="OData.Description" String="The members of this collection."/>
            <Annotation Term="OData.LongDescription" String="This property shall contain an array of links to the members of this collection."/>
            <Annotation Term="OData.AutoExpandReferences"/>
            <Annotation Term="Redfish.Required"/>
          </NavigationProperty>
      </EntityType>

    </Schema>

  </edmx:DataServices>
</edmx:Edmx>

```

<h3>
    tareo_hello.hpp
</h3>

```cpp=
#pragma once

#include <app.hpp>
#include <optional>
#include <variant>
#include <string>
#include <iostream>

#include <sdbusplus/sdbus.hpp>
#include <sdbusplus/bus.hpp>
#include <sdbusplus/exception.hpp>

#include <future>
#include <chrono>

namespace redfish
{
    size_t HelloId = 0;
    std::string HelloName = "";
    std::string HelloStatus = "";

inline void getTareoDbusTestData(const std::shared_ptr<bmcweb::AsyncResp>& asyncResp)
{
    BMCWEB_LOG_DEBUG << "Get TareoDbusTest Data";
    
    //
    // GET Dbus Id
    //
    crow::connections::systemBus->async_method_call(
        [asyncResp](const boost::system::error_code ec, std::variant<uint32_t>& id)
        {
            if(ec) {
                BMCWEB_LOG_ERROR << "D-BUS response error " << ec;
                messages::internalError(asyncResp->res);
                return;
            }

            const uint32_t* value = std::get_if<uint32_t>(&id);

            if (value == nullptr)
            {
                messages::internalError(asyncResp->res);
                return;
            }

            HelloId = static_cast<size_t>(*value);;

            BMCWEB_LOG_DEBUG << "HelloStatus Id value = " << HelloId;
            asyncResp->res.jsonValue["Id"] = HelloId;
        },
        "xyz.openbmc_project.TareoDbusTest", "/xyz/openbmc_project/TareoDbusTest",
        "org.freedesktop.DBus.Properties", "Get",
        "xyz.openbmc_project.TareoDbusTest", "Id");

    //
    // GET Dbus Name
    //
    crow::connections::systemBus->async_method_call(
        [asyncResp](const boost::system::error_code ec, std::variant<std::string>& name)
        {
            if(ec) {
                BMCWEB_LOG_ERROR << "D-BUS response error " << ec;
                messages::internalError(asyncResp->res);
                return;
            }

            const std::string* value = std::get_if<std::string>(&name);

            if (value == nullptr)
            {
                messages::internalError(asyncResp->res);
                return;
            }
            HelloName = *value;

            BMCWEB_LOG_DEBUG << "HelloStatus Name value = " << HelloName;
            asyncResp->res.jsonValue["Name"] = HelloName;
        },
        "xyz.openbmc_project.TareoDbusTest", "/xyz/openbmc_project/TareoDbusTest",
        "org.freedesktop.DBus.Properties", "Get",
        "xyz.openbmc_project.TareoDbusTest", "Name");

    //
    // GET Dbus Status
    //
    crow::connections::systemBus->async_method_call(
        [asyncResp](const boost::system::error_code ec, std::variant<std::string>& status)
        {
            if(ec) {
                BMCWEB_LOG_ERROR << "D-BUS response error " << ec;
                messages::internalError(asyncResp->res);
                return;
            }

            const std::string* value = std::get_if<std::string>(&status);

            if (value == nullptr)
            {
                messages::internalError(asyncResp->res);
                return;
            }
            HelloStatus = *value;

            BMCWEB_LOG_DEBUG << "HelloStatus Status value = " << HelloStatus;
            asyncResp->res.jsonValue["Status"] = HelloStatus;
        },
        "xyz.openbmc_project.TareoDbusTest", "/xyz/openbmc_project/TareoDbusTest",
        "org.freedesktop.DBus.Properties", "Get",
        "xyz.openbmc_project.TareoDbusTest", "Status");
} // getTareoDbusTestData

//
// SET Id Dbus Data
//
inline void setTareoDbusIntData(const std::shared_ptr<bmcweb::AsyncResp>& asyncResp, const uint32_t& data)
{
    BMCWEB_LOG_DEBUG << "Set Id Dbus Data";
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
        "xyz.openbmc_project.TareoDbusTest", "Id", dbusPropertyValue);
}

//
// SET Status Dbus Data
//
inline void setTareoDbusStringData(const std::shared_ptr<bmcweb::AsyncResp>& asyncResp, const std::string& data)
{
    BMCWEB_LOG_DEBUG << "Set Status Dbus Data";
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
        "xyz.openbmc_project.TareoDbusTest", "Status", dbusPropertyValue);
}

inline void requestRoutesTareoService(App& app)
{
    BMCWEB_LOG_DEBUG << "Hello TareoService";
    
    //
    //  GET
    //
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoService/TareoHello/")
        .privileges(redfish::privileges::getTareoHello)
        .methods(boost::beast::http::verb::get)(
            [](const crow::Request&, 
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
            {

                BMCWEB_LOG_DEBUG << "Hello Tareo GET";

                asyncResp->res.jsonValue["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello";
                asyncResp->res.jsonValue["@odata.type"] = "#TareoHello.v1_0_0.TareoHello"; // TareoService.v1_0_0 內的 Entity Type
                asyncResp->res.jsonValue["Id"] = "HelloService";
                asyncResp->res.jsonValue["Name"] = "Hello Service";
                asyncResp->res.jsonValue["Description"] = "Tareo Hello Service";
                asyncResp->res.jsonValue["Status"] = {
                    {"Health", "OK"},
                    {"HealthRollup", "OK"},
                    {"State", "Enabled"}
                };
                asyncResp->res.jsonValue["Actions"] = {
                    {"#TareoHello.SayHello", {
                        {"target", "/redfish/v1/Oem/TareoService/TareoHello/Actions/TareoService.SayHello"},
                        {"@Redfish.ActionInfo", "/redfish/v1/Oem/TareoService/TareoHello/SayHelloActionInfo"}
                    }}
                };
                
                // nlohmann::json helloArray = nlohmann::json::array();
                // helloArray.push_back({{"@odata.id", "/redfish/v1/Oem/TareoService/TareoHello/Hello1"}});
                // helloArray.push_back({{"@odata.id", "/redfish/v1/Oem/TareoService/TareoHello/Hello2"}});

                // asyncResp->res.jsonValue["Hello"]["Members"] = helloArray;
                // asyncResp->res.jsonValue["Hello"]["Members@odata.count"] = helloArray.size();
                asyncResp->res.jsonValue["Hello"]["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello/Hello1";
                asyncResp->res.jsonValue["Hello2"]["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello/Hello2";
                // getTareoDbusTestData(asyncResp);
            }
        );

    //
    //  POST
    //
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoService/TareoHello/Actions/TareoService.SayHello/")
        .privileges(redfish::privileges::postTareoHello)
        .methods(boost::beast::http::verb::post)(
            [](const crow::Request&,
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
            {
                messages::success(asyncResp->res);
            }
        );

} // requestRoutesTareoService

inline void requestRoutesTareoHello(App& app)
{
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoService/TareoHello/Hello1")
        .privileges({{"Login"}})
        .methods(boost::beast::http::verb::get)(
            [](const crow::Request&, 
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
            {
                BMCWEB_LOG_DEBUG << "TareoHello TareoHello Hello GET";
                asyncResp->res.jsonValue["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello/Hello1";
                asyncResp->res.jsonValue["@odata.type"] = "#TareoHello.v1_0_0.Hello";
                asyncResp->res.jsonValue["Description"] = "TEST HELLO 1";
                getTareoDbusTestData(asyncResp);
            }
        );
    //
    //  PATCH
    //
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoService/TareoHello/Hello1")
        .privileges(redfish::privileges::patchTareoHello)
        .methods(boost::beast::http::verb::patch)(
            [](const crow::Request& req,
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
            {
                BMCWEB_LOG_DEBUG << "Hello Tareo PATCH";
                std::optional<size_t> Id; // optinal 用於表示可能存在也可能不存在的值。
                std::optional<std::string> Name;
                std::optional<std::string> Status;

                // The phase of getting input data
                if(!json_util::readJson(req, asyncResp->res,
                   "Id", Id,
                   "Name", Name,
                   "Status", Status))
                {
                    BMCWEB_LOG_ERROR << "Failed to call json_util::readJson(req)";
                    return;
                }

                // The phase of setting modified data to backend
                if(Id) {
                    HelloId = Id.value();
                    if(Id < 0 || Id > 100) {
                        messages::propertyValueIncorrect("Id", std::to_string(Id.value()));
                        return;
                    }
                    setTareoDbusIntData(asyncResp, HelloId);
                }

                if(Name) {
                    messages::propertyValueIncorrect("Name", Name.value());
                }

                if(Status) {
                    HelloStatus = Status.value();
                    if(HelloStatus == "Enabled" || HelloStatus == "1") {
                        HelloStatus = "Enabled";
                        setTareoDbusStringData(asyncResp, HelloStatus);
                    } else if(HelloStatus == "Disabled" || HelloStatus == "0") {
                        HelloStatus = "Disabled";
                        setTareoDbusStringData(asyncResp, HelloStatus);
                    } else {
                        messages::propertyValueIncorrect("Status", HelloStatus);
                        return;
                    }
                }

                asyncResp->res.result(boost::beast::http::status::ok);
                redfish::CupsService::addMessageToErrorJson(asyncResp->res.jsonValue, redfish::messages::success());
            }
        );
}// requestRoutesTareoService

//
// Get HelloCollection
//
inline void requestRoutesTareoCollection(App& app)
{
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoService/TareoHello/Hello2/")
        .privileges({{"Login"}})
        .methods(boost::beast::http::verb::get)(
            [](const crow::Request&,
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
            {
                // std::vector<std::string> TestCount = {"TEST1", "TEST2"};
                asyncResp->res.jsonValue["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello/Hello2";
                asyncResp->res.jsonValue["@odata.type"] = "#TareoHelloCollection.TareoHelloCollection";
                asyncResp->res.jsonValue["Name"] = "Tareo Hello Collection";
                asyncResp->res.jsonValue["Description"] = "A Collection of Tareo Hello instances";

                // nlohmann::json& memberArray = asyncResp->res.jsonValue["Members"];
                // memberArray = nlohmann::json::array();
                // asyncResp->res.jsonValue["Members@odata.count"] = TestCount.size();

                collection_util::getCollectionMembers(
                    asyncResp,
                    "/redfish/v1/Oem/TareoService/TareoHello/Hello2",
                    {"xyz.openbmc_project.TareoDbusTest"},
                    "/xyz/openbmc_project/TareoDbusTest"
                );

                // for(const std::string& id : TestCount)
                // {
                //     memberArray.push_back({{"@odata.id", "/redfish/v1/Oem/TareoService/TareoHello/Hello2/" + id}});
                // }
            }
        );
} // requestRoutesTareoCollection

} // namespace redfish

```

<h3>
    redfish.hpp
</h3>

![image](https://hackmd.io/_uploads/H1hI40aXA.png)

<h3>
    privilege_registery.hpp
</h3>

![image](https://hackmd.io/_uploads/r1owVR6Q0.png)

<h3>
    service_root.hpp
</h3>

![image](https://hackmd.io/_uploads/ryAO4RTXC.png)


<h2>
    POST
</h2>

![image](https://hackmd.io/_uploads/SJ11qJ7VA.png)

```
curl -k -u root:0penBmc -X POST https://192.168.9.227/redfish/v1/Oem/TareoService/TareoHello/Actions/TareoService.SayHello -d '{"Id": 37,"Name": "Rita", "Status": "Enable"}' --header "Content-Type:application"
```

![image](https://hackmd.io/_uploads/ry7Z5ym4R.png)


![image](https://hackmd.io/_uploads/H1SwhkmVA.png)

<h2>
    Collection
</h2>

![image](https://hackmd.io/_uploads/B1r-OksEC.png)

```cpp=
inline void requestRoutesTareoCollection(App& app)
{
    BMCWEB_ROUTE(app, "/redfish/v1/Oem/TareoService/TareoHello/Hello2")
        .privileges({{"Login"}})
        .methods(boost::beast::http::verb::get)(
            [](const crow::Request&,
               const std::shared_ptr<bmcweb::AsyncResp>& asyncResp) -> void
            {
                std::vector<std::string> TestCount = {"TEST1", "TEST2"};
                asyncResp->res.jsonValue["@odata.id"] = "/redfish/v1/Oem/TareoService/TareoHello/Hello2";
                asyncResp->res.jsonValue["@odata.type"] = "#TareoHelloCollection.TareoHelloCollection";
                asyncResp->res.jsonValue["Name"] = "Tareo Hello Collection";
                asyncResp->res.jsonValue["Description"] = "A Collection of Tareo Hello instances";

                nlohmann::json& memberArray = asyncResp->res.jsonValue["Members"];
                memberArray = nlohmann::json::array();
                asyncResp->res.jsonValue["Members@odata.count"] = TestCount.size();

                for(const std::string& id : TestCount)
                {
                    memberArray.push_back({{"@odata.id", "/redfish/v1/Oem/TareoService/TareoHello/Hello2/" + id}});
                }
            }
        );
} // requestRoutesTareoCollection
```