<h2>
    Topic
</h2>

[TOC]

<h2>
    Fan speed control (FSC)
</h2>

最近有遇到一個問題是關於 FAN PID 的控制 , 一開始想說用 phosphor-pid-control 就可以去做
但是搞成 stepwise 了... 所以寫個紀錄一下

<h3>
    Fan Control
</h3>

![image](https://hackmd.io/_uploads/BycsLOrvA.png)

底下有一個 Reference 他寫得很清楚, 所以只把我有用到的貼上來, 其他自己去那邊看

關於 **PWM** 跟 **TACH** 那邊也有解釋

文中有提到說用有沒有 feedback 來看是 `open loop control` 還是 `closed loop control`

open loop control   -> **stepwise**
closed loop control -> **PID control**

[How to Configure Phosphor-pid-control](<https://gerrit.openbmc.org/plugins/gitiles/openbmc/phosphor-pid-control/+/6a7261f1228ceb84a2f4c19c67b9304dcfe3c645/configure.md>)

寫完的 `config.json` 放到 `/usr/share/swampd` 底下就好了, 然後重啟 service

<h3>
    Open loop control  演算法 (Stepwise)
</h3>

Open loop，會根據 input 來直接求出 output 並輸出，不會參考 feedback 或任何的 actuating error

```json=
"zones" :
    [
        {
            "id": 0,
            "minThermalOutput": 40,
            "failsafePercent": 100.0,
            "pids":             [
                {
                    "name": "NODE_FAN",
                    "type": "fan",
                    "inputs": ["FAN_PWM_1",
                               "FAN_PWM_2",
                               "FAN_PWM_3",
                               "FAN_PWM_4"],
                    "setpoint": 90.0,
                    "pid": {
                        "samplePeriod": 1.0,
                        "proportionalCoeff": 0.0,
                        "integralCoeff": 0.0,
                        "feedFwdOffsetCoeff": 0.0,
                        "feedFwdGainCoeff": 1.0,
                        "integralLimit_min": 0.0,
                        "integralLimit_max": 0.0,
                        "outLim_min": 0.0,
                        "outLim_max": 100.0,
                        "slewNeg": 0.0,
                        "slewPos": 0.0
                    }
                },
                {
                    "name": "TEMP",
                    "type": "stepwise",
                    "inputs": ["TEMP"],
                    "setpoint": 20.0,
                    "pid":
                    {
                        "samplePeriod": 1.0,
                        "positiveHysteresis": 0.0,
                        "negativeHysteresis": 0.0,
                        "isCeiling": false,
                        "reading":
                        {
                            "0": 24,
                            "1": 25,
                            "2": 30,
                            "3": 35,
                            "4": 40,
                            "5": 45,
                            "6": 50,
                            "7": 100
                    },
                        "output":
                        {
                            "0": 50,
                            "1": 60,
                            "2": 70,
                            "3": 80,
                            "4": 90,
                            "5": 100,
                            "6": 100,
                            "7": 100
                        }
                    }
                }
            ]
        }
    ]
```

<h3>
    closed loop control  演算法 (PID)
</h3>

這是一種更複雜的控制算法，通過調整比例（P）、積分（I）和微分（D）參數來實現精確控制

[Phosphor-pid-control PID 算法](<https://github.com/openbmc/phosphor-pid-control/blob/master/pid/ec/pid.cpp >)

```json=
"zones": [
        {
            "id": 0,
            "minThermalOutput": 3000,
            "failsafePercent": 750.0,
            "pids": [
                {
                    "name": "SYS_TEMP_CONTROL",
                    "type": "temp",
                    "inputs": ["SYS0_TEMP", "SYS1_TEMP", "SYS2_TEMP"],
                    "setpoint": 70.0,
                    "pid": {
                        "samplePeriod": 1.0,
                        "proportionalCoeff": 5.0,
                        "integralCoeff": 0.5,
                        "feedFwdOffsetCoeff": 0.0,
                        "feedFwdGainCoeff": 0.0,
                        "integralLimit_min": -100.0,
                        "integralLimit_max": 100.0,
                        "outLim_min": 30.0,
                        "outLim_max": 100.0,
                        "slewNeg": 0.0,
                        "slewPos": 0.0
                    }
                },
                {
                    "name": "NODE_FAN",
                    "type": "fan",
                    "inputs": ["FAN_PWM_1", "FAN_PWM_2", "FAN_PWM_3", "FAN_PWM_4"],
                    "setpoint": 90.0,
                    "pid": {
                        "samplePeriod": 1.0,
                        "proportionalCoeff": 5.0,
                        "integralCoeff": 0.5,
                        "feedFwdOffsetCoeff": 0.0,
                        "feedFwdGainCoeff": 0.0,
                        "integralLimit_min": 0.0,
                        "integralLimit_max": 0.0,
                        "outLim_min": 30.0,
                        "outLim_max": 100.0,
                        "slewNeg": 0,
                        "slewPos": 0
                    }
                }
            ]
        }
    ]
```

一般來說 PID 的算法每家都不一樣, 參數也不一樣, 所以通常這些數值都是 Thermal Team 去弄得, 我們無法幫別人去填, 因為我們不知道他們的機殼, 風扇等等


<h3>
    Phosphor-pid-control
</h3>

![image](https://hackmd.io/_uploads/S1srZz5D0.png)

![image](https://hackmd.io/_uploads/Bk5mZz9v0.png)

Sensor 的部分他是去讀 DBUS 上的數值, 使用 ObjectMapper 去 find 的,
要確保 Sensor 的值是正確的, 如果 timeout 前沒有正確獲取值 FailSafe 會掛起來
PWM 的部分就會是按照 `"failsafePercent": 75` 去調整

通常設定 PID 會自動去控制, 如果想要手動控制 PWM 把 Manual 改成 true 就可以了

![image](https://hackmd.io/_uploads/BySs-GcwA.png)

```
busctl introspect xyz.openbmc_project.State.FanCtrl /xyz/openbmc_project/settings/fanctrl/zone0
```

<h3>
    PID Field
</h3>

If the PID **type** is not **stepwise** then the PID field is defined as follows:



| field              |  type  |                                 meaning                                  |
| ------------------ |:------:|:------------------------------------------------------------------------:|
| samplePeriod       | double | How frequently the value is sampled. 0.1 for fans, 1.0 for temperatures. |
| proportionalCoeff  | double |                      The proportional coefficient.                       |
| integralCoeff      | double |                        The integral coefficient.                         |
| feedFwdOffsetCoeff | double |                   The feed forward offset coefficient.                   |
| feedFwdGainCoeff   | double |                    The feed forward gain coefficient.                    |
| integralLimit_min  | double |                    The integral minimum clamp value.                     |
| integralLimit_max  | double |                    The integral maximum clamp value.                     |
| outLim_min         | double |                     The output minimum clamp value.                      |
| outLim_max         | double |                     The output maximum clamp value.                      |
| slewNeg            | double |                  Negative slew value to dampen output.                   |
| slewPos            | double |                Positive slew value to accelerate output.                 |

The units for the coefficients depend on the configuration of the PIDs.


<h3>
    Issue
</h3>

目前在 QEMU 底下測變這樣, 他要確保 BMC Temp 可以正確拿到資料
不過我目前還沒有實體機台可以測QQ

![image](https://hackmd.io/_uploads/SkemtGqwC.png)

<h3>
    完整 config Example
</h3>

```json
{
    "version" : "2600evb-Stepwise-v01",
    "sensors" :
    [
        {
            "name": "FAN_1_1",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm1",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm1",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_2",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm2",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm2",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_3",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm3",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm3",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_4",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm4",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm4",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_5",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm5",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm5",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_6",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm6",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm6",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_13",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm13",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm13",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_14",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm14",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm14",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_15",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm15",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm15",
            "min": 0,
            "max": 255
        },
        {
            "name": "FAN_1_16",
            "type": "fan",
            "readPath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm16",
            "writePath": "/sys/devices/platform/ahb/ahb:apb/1e610000.pwm-tacho-controller/hwmon/**/pwm16",
            "min": 0,
            "max": 255
        },
        {
            "name": "BMC_Temp",
            "type": "temp",
            "readPath": "/xyz/openbmc_project/sensors/temperature/BMC_Temp",
            "ignoreDbusMinMax": true,
            "timeout": 0
        }
    ],
    "zones" :
    [
        {
            "id": 0,
            "minThermalOutput": 40,
            "failsafePercent": 100.0,
            "pids":             [
                {
                    "name": "NODE_FAN",
                    "type": "fan",
                    "inputs": ["FAN_1_1",
                               "FAN_1_2",
                               "FAN_1_3",
                               "FAN_1_4",
                               "FAN_1_5",
                               "FAN_1_6",
                               "FAN_1_13",
                               "FAN_1_14",
                               "FAN_1_15",
                               "FAN_1_16"],
                    "setpoint": 90.0,
                    "pid": {
                        "samplePeriod": 1.0,
                        "proportionalCoeff": 0.0,
                        "integralCoeff": 0.0,
                        "feedFwdOffsetCoeff": 0.0,
                        "feedFwdGainCoeff": 1.0,
                        "integralLimit_min": 0.0,
                        "integralLimit_max": 0.0,
                        "outLim_min": 0.0,
                        "outLim_max": 100.0,
                        "slewNeg": 0.0,
                        "slewPos": 0.0
                    }
                },
                {
                    "name": "BMC_Temp",
                    "type": "stepwise",
                    "inputs": ["BMC_Temp"],
                    "setpoint": 20.0,
                    "pid": 
                    {
                        "samplePeriod": 1.0,
                        "positiveHysteresis": 0.0,
                        "negativeHysteresis": 0.0,
                        "isCeiling": false,
                        "reading":
                        {
                            "0": 24,
                            "1": 25,
                            "2": 30,
                            "3": 35,
                            "4": 40,
                            "5": 45,
                            "6": 50,
                            "7": 100
                    },
                        "output":
                        {
                            "0": 50,
                            "1": 60,
                            "2": 70,
                            "3": 80,
                            "4": 90,
                            "5": 100,
                            "6": 100,
                            "7": 100
                        }
                    }
                }
            ]
        }
    ]
}

```

<h2>
    phosphor-pid-control_%.bbappend
</h2>

```json
FILESEXTRAPATHS_prepend := "${THISDIR}/${PN}:"

SRC_URI_append += " file://test-config.json"

FILES_${PN}_append = " \
    ${datadir}/swampd/test-config.json \
    /usr/share/swampd \
    /usr/share/swampd/config.json \
"

do_install_append() {
    install -d ${D}/${bindir}

    install -d ${D}${datadir}/swampd
    install -m 0644 -D ${WORKDIR}/test-config.json \
        ${D}${datadir}/swampd/config.json
}
```

```
❯ tree
.
├── phosphor-pid-control
│   └── test.json
└── phosphor-pid-control_%.bbappend

1 directory, 2 files
```

<h2>
    Reference
</h2>

https://github.com/openbmc/phosphor-pid-control

https://iris123321.blogspot.com/2019/02/fan-speed-control-pid-algorithm-and.html