![Redfish-Dbus-Led.drawio](https://hackmd.io/_uploads/r1UPk3CrR.png)

<h2>
    LED
</h2>

**dbus-sensors** : Implement DBus sensors function

**callback-manager** : Implement sensors threshold event link to LED group

**phosphor-led-manager** : Manage DBus LED controller service

**phosphor-led-sysfs** : Control LED physical status service, 它通過 sysfs 來監控和操作 LED，並通過 D-Bus 來提供對這些 LED 的控制。

LED control feature is achieved by following three parts:

* Physical LEDs
* LED Groups
* Associations of Events and LED Actions in Groups

<h3>
    Config File
</h3>

DTS LED config
* aspeed-bmc-intel-ast2600.dts

LED group config
* led-group-config.json

Threshold of sensors event
* AC-Baseboard.json

<h3>
    Configure Physical LED Manager
</h3>

```
aspeed-ast2600-a0-evb-common.dts
```

![image](https://hackmd.io/_uploads/S12iimarR.png)

<h3>
    LED configuration relations
</h3>

```
led.yaml
```
![image](https://hackmd.io/_uploads/S1YDhmpHA.png)


<h2>
    phosphor-led-manager
</h2>

LED Dbus 的部份主要的 source code 是 led-main.cpp

如果是使用 Json 去配置的話, 他一開始會從 JSON 獲取 LED 的配置資訊

```cpp
#ifdef LED_USE_JSON
    auto systemLedMap = getSystemLedMap();
#endif
```