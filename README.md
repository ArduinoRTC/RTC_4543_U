# EPSON 4543SA/SB

This library is a driver for the Epson RTC4543SA/SB.


## Functional verification
For the RTC, I used Akizuki Denshi's [AkizukiRTC_4543][32 kHz Output Serial RTC Board Module (uses RT4543)].

The following three models have been confirmed to work.

| CPU Architecture | Model Used |
|---|---|
| AVR | Arduino Mega |
| SAMD | Arduino MKR WiFi 1010 |
| SAM | Arduino Due |

### Restrictions
The function `int checkLowPower(void)`, which reads the flag that detects a power loss, sometimes returns a result indicating that power has not been lost (even though the power was turned off), which is suspicious behavior. This occurred on all models (Mega, Due, etc.).

Since this is the first bit of the data access sequence, it might be a timing issue, but I haven’t been able to resolve it yet.

Since I don’t have a logic analyzer or similar equipment, it is practically impossible to resolve communication-related issues.

# API

For detailed information on how the system works, please refer to the application manual published by Epson in addition to this document.

## Regarding initialization
### Object creation
```
RTC_4543_U(uint8_t _dataPin, uint8_t _clkPin, uint8_t _wrPin, uint8_t _cePin, uint8_t _fsel, int32_t _rtcID=-1)
```

Provide the pin numbers connected to each terminal used by the 4543 and the  ID of rtc as arguments.
Since `_fsel` is used for clock signal output, please refer to that section.

### initialization
```
bool  begin(bool init, uint32_t addr)
```

The first argument is a flag indicating whether to configure the time, timer, or alarm.

The second argument is intended for I2C devices and the like, so it is ignored by this RTC driver.

| Return Value | Meaning |
|---|---|
|true|Initialization successful|
|false|Initialization failed|

## Retrieving RTC Information
A member function that retrieves information about the type and features of the RTC chip.
```
void  getRtcInfo(rtc_u_info_t *info)
```

## Information about time
### Time Settings
```
bool  setTime(rtc_date_t* time)
```
Sets the RTC to the time specified in the argument.
| Return Value | Meaning |
|---|---|
|true|Set successfully|
|false|Set failed|

Note that the data types of the arguments are defined in ``dateUtils.h`` as shown below. Most RTCs, including this one, can not  use time in milliseconds.

```
typedef struct  {
  uint16_t  year;
  uint8_t   month;
  uint8_t   mday;
  uint8_t   wday;
  uint8_t   hour;
  uint8_t   minute;
  uint8_t   second;
  int16_t   millisecond;
} date_t;
```

### Get the time
```
bool  getTime(rtc_date_t* time)
```
Store the time information obtained from the RTC in a structure passed as an argument.
| Return Value | Meaning |
|---|---|
|true|Retrieval successful|
|false|Retrieval failed|


## Frequency signal output
### Clock Output Settings
```
int   setClockOut(uint8_t num, uint8_t freq, int8_t pin=-1)
```

In this RTC (4543SA/SB), there is only one type of clock signal output. The output frequency is determined by the voltage setting on an external pin, and the output status (on or off) is determined by the voltage on a different pin. 
Therefore, the first argument must be 0. Specify the Arduino pin number connected to the RTC's FOE terminal in `pin`.

| freq value | Output frequency |
|---|---|
| 0 | 32.768 kHz |
| Others | 1 Hz |

In practice, the output frequency of the RTC clock signal is controlled by setting the voltage on the ``_fsel`` pin—specified when creating an object of this class—to either ``HIGH`` or ``LOW`` according to the value of ``freq``. Therefore, please note that if `_fsel` or the `pin` of this function is not properly connected to the RTC, the signal will not be output correctly.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_ILLEGAL_PARAM | Invalid value for `num` |

### Clock Frequency Settings
```
int   setClockOutMode(uint8_t num, uint8_t freq)
```
The meaning of each argument and the details of its operation are the same as those of ``setClockOut()``.

| freq value | Output frequency |
|---|---|
| 0 | 32.768 kHz |
| Other | 1 Hz |

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_ILLEGAL_PARAM | Invalid value for `num` |

### Controlling the Clock Output
```
int   controlClockOut(uint8_t num, uint8_t mode)
```
Since there is only one terminal that outputs frequency signals, `num` must be 0. Additionally, the second argument has the meanings shown in the table below.

|``mode`` value|Meaning|
|---|---|
|0|Clock output stopped|
|1|Clock output started|


In practice, the signal output from the RTC is turned on or off by setting the voltage of the `pin` (foe pin) specified in `setClockOut()` to `HIGH` or `LOW`.

| Return Value | Meaning |
|---|---|
|RTC_U_SUCCESS | Configuration successful |
|RTC_U_FAILURE | Configuration failed |
|RTC_U_ILLEGAL_PARAM | Setting of unsupported parameters, etc. |


## Power Supply Voltage
### Retrieving the power flag
```
int checkLowPower(void)
```
Retrieves and returns the ``FDT``value from the RTC data, which indicates a drop in the power supply voltage.

| Return Value | Meaning |
|---|---|
|0 | No voltage drop |
|1 | Voltage drop detected |

In this RTC, the ``FDT`` is cleared when all RTC data is read, so it cannot be read multiple times.

### Clearing the power flag
```
int clearPowerFlag(void)
```
A function that clears the flag indicating a power loss (``FDT``). It always returns ``RTC_U_SUCCESS``.

In this RTC, since the ``FDT`` is cleared when all RTC data is read, there is no need to call this function if ``checkLowPower()`` has been executed.


[AkizukiRTC_4543]:https://akizukidenshi.com/catalog/g/gK-10722/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[AdafruitUSD]:https://github.com/adafruit/Adafruit_Sensor
[shield]:https://www.seeedstudio.com/Base-Shield-V2-p-1378.html
[M0Pro]:https://store.arduino.cc/usa/arduino-m0-pro
[Due]:https://store.arduino.cc/usa/arduino-due
[Uno]:https://store.arduino.cc/usa/arduino-uno-rev3
[UnoWiFi]:https://store.arduino.cc/usa/arduino-uno-wifi-rev2
[Mega]:https://store.arduino.cc/usa/arduino-mega-2560-rev3
[LeonardoEth]:https://store.arduino.cc/usa/arduino-leonardo-eth
[ProMini]:https://www.sparkfun.com/products/11114
[ESPrDev]:https://www.switch-science.com/catalog/2652/
[ESPrDevShield]:https://www.switch-science.com/catalog/2811/
[ESPrOne]:https://www.switch-science.com/catalog/2620/
[ESPrOne32]:https://www.switch-science.com/catalog/3555/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[Arduino]:http://https://www.arduino.cc/
[Sparkfun]:https://www.sparkfun.com/
[SwitchScience]:https://www.switch-science.com/



<!--- コメント

## 動作検証

|CPU| 機種 |ベンダ| 結果 | 備考 |
| :--- | :--- | :--- | :---: | :--- |
|AVR| [Uno R3][Uno]  |[Arduino][Arduino]|    ○  |      |
|       | [Mega2560 R3][Mega] |[Arduino][Arduino] |  ○    |      |
|       | [Leonardo Ethernet][LeonardoEth] |[Arduino][Arduino] |  △   |   割り込みピンが使えない? (D3,D4はだめだった)   |
|       | [Uno WiFi][UnoWiFi] |[Arduino][Arduino] |    ○  | |
|       | [Pro mini 3.3V][ProMini] | [Sparkfun][Sparkfun] | ○     |      |
| ARM/M0+ | [M0 Pro][M0Pro] |[Arduino][Arduino] |○||
|ESP8266|[ESPr developer][ESPrDev]| [スイッチサイエンス][SwitchScience] |○|D14|
|ESP32 | [ESPr one 32][ESPrOne32] | [スイッチサイエンス][SwitchScience] |○|　D13|

## 外部リンク

- Seeed Studio技術Wiki [http://wiki.seeedstudio.com/Grove-3-Axis_Digital_Accelerometer-1.5g/][SeedWiki]
- センサ商品ページ [https://www.seeedstudio.com/Grove-3-Axis-Digital-Accelerometer-1-5-p-765.html][ProductPage]
- ソースリポジトリ [https://github.com/Seeed-Studio/Accelerometer_MMA7660][github]
- Adafruit Unified Sensor Driver - [https://github.com/adafruit/Adafruit_Sensor][AdafruitUSD]
- Groveシールド - [https://www.seeedstudio.com/Base-Shield-V2-p-1378.html][shield]
- Arduino M0 Pro - [https://store.arduino.cc/usa/arduino-m0-pro][M0Pro]
- Arduino Due - [https://store.arduino.cc/usa/arduino-due][Due]
- Arduino Uno R3 - [https://store.arduino.cc/usa/arduino-uno-rev3][Uno]
- Arduino Uno WiFi - [https://store.arduino.cc/usa/arduino-uno-wifi-rev2][UnoWiFi]
- Arduino Leonardo Ethernet - [https://store.arduino.cc/usa/arduino-leonardo-eth][LeonardoEth]
- Arduino Mega2560 R3 - [https://store.arduino.cc/usa/arduino-mega-2560-rev3][Mega]
- Arduino Pro mini 328 - 3.3V/8MHz - [https://www.sparkfun.com/products/11114][ProMini]
- ESPr developer - [https://www.switch-science.com/catalog/2652/][ESPrDev]
- ESPr Developer用GROVEシールド - [https://www.switch-science.com/catalog/2811/][ESPrDevShield]
- ESpr one - [https://www.switch-science.com/catalog/2620/][ESPrOne]
- ESPr one 32 - [https://www.switch-science.com/catalog/3555/][ESPrOne32]
- Grove - [https://www.seeedstudio.io/category/Grove-c-1003.html][Grove]
- Seed Studio - [https://www.seeedstudio.io/][SeedStudio]
- Sparkfun Electronics - [https://www.sparkfun.com/][Sparkfun]
- スイッチサイエンス - [https://www.switch-science.com/][SwitchScience]


[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[ProductPage]:https://www.seeedstudio.com/Grove-3-Axis-Digital-Accelerometer-1-5-p-765.html
[SeedWiki]:http://wiki.seeedstudio.com/Grove-3-Axis_Digital_Accelerometer-1.5g/
[github]:https://github.com/Seeed-Studio/Accelerometer_MMA7660
[AdafruitUSD]:https://github.com/adafruit/Adafruit_Sensor
[shield]:https://www.seeedstudio.com/Base-Shield-V2-p-1378.html
[M0Pro]:https://store.arduino.cc/usa/arduino-m0-pro
[Due]:https://store.arduino.cc/usa/arduino-due
[Uno]:https://store.arduino.cc/usa/arduino-uno-rev3
[UnoWiFi]:https://store.arduino.cc/usa/arduino-uno-wifi-rev2
[Mega]:https://store.arduino.cc/usa/arduino-mega-2560-rev3
[LeonardoEth]:https://store.arduino.cc/usa/arduino-leonardo-eth
[ProMini]:https://www.sparkfun.com/products/11114
[ESPrDev]:https://www.switch-science.com/catalog/2652/
[ESPrDevShield]:https://www.switch-science.com/catalog/2811
[ESPrOne]:https://www.switch-science.com/catalog/2620/
[ESPrOne32]:https://www.switch-science.com/catalog/3555/
[Grove]:https://www.seeedstudio.io/category/Grove-c-1003.html
[SeedStudio]:https://www.seeedstudio.io/
[Arduino]:http://https://www.arduino.cc/
[Sparkfun]:https://www.sparkfun.com/
[SwitchScience]:https://www.switch-science.com/


[Adafruit Unified Sensor Driver][AdafruitUSD]
[Groveシールド][shield]
[Arduino M0 Pro][M0Pro]
[Arduino Due][Due]
[Arduino Uno R3][Uno]
[Arduino Mega2560 R3][Mega]
[Arduino Leonardo Ethernet][LeonardoEth]
[Arduino Pro mini 328 - 3.3V/8MHz][ProMini]
[ESpr one][ESPrOne]
[ESPr one 32][ESPrOne32]
[Grove][Grove]
[Seed Studio][SeedStudio]
[Arduino][Arduino]
[Sparkfun][Sparkfun]
[スイッチサイエンス][SwitchScience]
--->
