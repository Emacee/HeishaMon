[![Jion the Slack chat room](https://img.shields.io/badge/Slack-Join%20the%20chat%20room-orange)](https://join.slack.com/t/panasonic-wemos/shared_invite/enQtODg2MDY0NjE1OTI3LTgzYjkwMzIwNTAwZTMyYzgwNDQ1Y2QxYjkwODg3NjMyN2MyM2ViMDM3Yjc3OGE3MGRiY2FkYzI4MzZiZDVkNGE)


# Panasonic H Series Aquarea air-water heat pump protocol



## Connection details:
CN-CNT TTL UART 9600,8,E,1  \
Pin-out (from top to bottom) \
1 - +5V (250mA)  \
2 - 0-5V TX  \
3 - 0-5 RX  \
4 - +12V (250mA) \
5 - GND

## Where to get connectors

CN-CNT female connector PAP-05V-S - JST Female Connector Housing - PA, 2mm Pitch, 5 Way, 1 Row - https://uk.rs-online.com/web/p/pcb-connector-housings/4766798/ \
Pre-made crimp leads 150 mm crimp-free end lead PA 2.0 can be used - https://uk.rs-online.com/web/p/pre-crimped-leads/5128721/ \
The Panasonic CZ-TAW1 appears to use a JST B05B-XASK-1 male header - https://uk.rs-online.com/web/p/pcb-headers/6027679/

For Conrad orders:

| JST Type | Conrad article number | Description |
| ----- | ---- | ----- |
|B05B-XASK-1| 741366 | Header matching CZ-TAW1, for soldering on PCB. S05B-XASK-1 (not orderable at conrad) is the side-ways version	|
|B05B-PASK-1| 1426194 | Header matching CN-CNT (that what is on the heatpump itself). Only need this if you want to build a proxy-forward PCB|
|XAP-05V-1| 741233 | Cable connector on CZ-TAW1/PCB side |
|PAP-05V-1| 1426227 | Cable connector on CN-CNT side | 
|BXA-01T-P0.6| 741295 |Connector pins on CZ-TAW1/PCB side|
|SPHD-002T-P0.5| 1426240 | Connector pins on CN-CNT side |

And add some 24 AWG shielded 4-conductors cable.

## How to connect

Currently the PCB's are in (beta) production. We suggest to wait a while for them to be tested. Soon you will be cable to order one of the PCB design directly from the project owners but ofcourse we will share the schematics also. For now these are some schematics we are testing or have tested.

PCB design based on through hole soldering, Wemos shield type. With basic components (resitors and transistors). https://easyeda.com/igor_6537/panasonic-cn-cnt-to-wemos-through-hole

PCB design based on SMD soldering, Wemos shield type. With SMD components (mosfet instead of transistors) https://easyeda.com/igor_6537/panasonic-cn-cnt-to-wemos

PCB design based on SMD soldering, ESP-12f type.  https://easyeda.com/kompiuteriu/new-cn-cnt

## using the test arduino image
The current arduino test image is able to read from the Panasonic Aquarea H-series CN-CNT connector. \
You need to build this image with support for a filesystem on the esp8266 so select the correct flash option in arduino ide for that. \
When starting for the first time a open-wifi-hotspot will be visible allowing you to config your wifi network and your mqtt server. \
If you ever want to factory reset, just double reset the esp8266 within 0.1 second. It will then format the filesystem and remove the wifi setting and start the wifi hotspot again. \
After configuring and booting the image will be able to read and talk to your heatpump. The GPIO13/GPIO15 connection will be used for communications so you can keep your computer/uploader connected to the board if you want. \
Serial 1 (GPIO2) can be used to connect another serial line (GND and TX from the board only) to read some debugging data.

## libs for building the test arduino image
boards: \
esp8266 by esp8266 community version 2.6.3 https://github.com/esp8266/Arduino/releases/tag/2.6.3

libs: \
wifimanager by tzapu version 0.15.0-beta https://github.com/tzapu/WiFiManager/releases/tag/0.15.0-beta \
pubsubclient by nick o'leary version 2.7.0 https://github.com/knolleary/pubsubclient/releases/tag/v2.7 \
doubleresetdetect by jens-christian skibakk version 1.0.0 https://github.com/jenscski/DoubleResetDetect/releases/tag/1.0.0 \
arduinojson by benoit blanchon version 6.13.0 https://github.com/bblanchon/ArduinoJson/releases/tag/v6.13.0

## MQTT topics
[Current list of documented MQTT topics can be found here](MQTT-Topics.md)

## Integration Examples for Opensource automation systems
[Openhab2](Integrations\Openhab)

## Protocol info packet:

To get information from a heat pump, "magic" packet should be send to CN-CNT: 

`71 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 12`

## Protocol byte decrypt info:

|  Topic# | Byte# | Possible Value | Value decrypt | Value Description |
| ----: | ---- | ---- | ----- | ----:|
|  TOP | 00 | 71 |   | Header  |
|  TOP | 01 | c8 | Data length ( Packet length = Data length + 3 )  |  Header |
|  TOP | 02 | 01|   | Header  |
|  TOP | 03 | 10 |   | Header   |
|  TOP0+TOP2 | 04 | 56 | Force DHW status 56=off,96=on, 55 = heat pump off, 56= heat pump on | Force dhw status + Heat pump on/off status|
|  TOP19+TOP13 | 05 | 55 | (hex) Holiday mode off and weekly shedule off =55, Holiday mode off and weekly shedule on =95, Holiday mode on and weekly shedule off = 65, Holiday mode on and active and weekly shedule off =75, Holiday mode on and weekly shedule on =B5, Holiday mode off and weekly shedule off + pump works on heater =59 | Holiday mode and Sheduler status |
|  TOP4 | 06 | 62 | If 62 Heat+DHW, If 52 Only Heat, If 61 only DHW, If 69 Auto+DHW, If 63 Cool+DHW, If 53 Cool, If 59 Auto   | Mode status   |
|  TOP18+TOP17 | 07 | 49 | Left 5 bits = quiet level (0b01001 = Off, 0b01010 = level 1, 0b01011 = level 2, 0b01100 - level 3, 0b10001 = scheduled) last 3 bits is powermode level (0b001= Off, 0b010 - power mode 30min, 0b011 -60min, 0b100-90 min) | Quiet Mode status + Powerfull mode status |
|  TOP | 08 | 00 |   | ? |
|  TOP | 09 | 05 | HEX values - should be Low Byte (2nd value)Floor heater off + Tank heater off=55, Floor heater on + Tanks heater off=56, Floor Heater off + Tank heater on=59, Floor heater on + Tank heater on=5A | heaters status|
|  TOP | 10 | 00 |   | ? |
|  TOP | 11 | 00 |   | ? |
|  TOP | 12 | 00 |   | ? |
|  TOP | 13 | 00 |   | ? |
|  TOP | 14 | 00 |   | ? |
|  TOP | 15 | 00 |   | ? |
|  TOP | 16 | 00 |   | ? |
|  TOP | 17 | 00 |   | ? |
|  TOP | 18 | 00 |   | ? |
|  TOP | 19 | 00 |   | ? |
|  TOP | 20 | 19 | (hex) Water as medium Antifreezing off Optional PCB off=15, Antifreezing off Optional PCB on=16, Antifreezing on Optional PCB off=19, Antifreezing on Optional PCB on=1A, Glikol as medium High byte from 1 changes to 9, Antifreezing on Optional PCB off and External out temp on=29 (+10 for all values if External temp sensor selected) | Anti freezing |
|  TOP | 21 | 15 |   | ? |
|  TOP | 22 | 11 | (hex) 11 - water temperature, 13 - Internal Thermostat, 12 - External Thermostat, 14 - Thermistor  | Zone & sensor settings ( system setup - Installer ) | 
|  TOP | 23 | 55 | (hex) Off=55, On=56  | External Switch |
|  TOP | 24 | 16 | (hex) Tank connected=16, not connected=15 | Tank Connection status |
|  TOP | 25 | 5e | (hex) Tank Heater internal and 3kW=95, Tank Heater external and 3kW=96, Tank Heater internal and 6kW=99, Tank Heater external and 6kW=9A, Tank Heater internal and 9kW=9D, Tank Heater External and 9KW -9E, Tank Heater external and 9KW + Heater pad Type A on=AE, Tank Heater external and 9KW + Heater pad Type B on=BE  | Power of internal heater + tank heater Internal/External + Heater for external pad |
|  TOP | 26 | 55 | (hex) Biwalent Off=55, Biwalent alternative =56, Biwalent parallel=5A | Biwalent settings |
|  TOP | 27 | 05 |   | ? |
|  TOP | 28 | 09 | (hex) 09 - Compensation curve heat and direct cool, 05 - both compensation curves , 0a - direct heat and direct cool, 06 - heat direct, cool compensation curve  | Operation Setup -Installer -water temperature heating on status and cooling |
|  TOP | 29 | 00 |   | ? |
|  TOP | 30 | 00 |   | ? |
|  TOP | 31 | 00 |   | ? |
|  TOP | 32 | 00 |   | ? |
|  TOP | 33 | 00 |   | ? |
|  TOP | 34 | 00 |   | ? |
|  TOP | 35 | 00 |   | ? |
|  TOP | 36 | 00 |   | ? |
|  TOP | 37 | 00 |   | ? |
|  TOP27 | 38 | 80 | Convert to DEC 128-128 = 0  | Zone 1 water shift set or direct mode value Temperature For Heat Mode [°C] |
|  TOP28 | 39 | 8f | Convert to DEC 143-128 = 15 in direct mode set temp or shift value  | Zone 1 water shift set Temperature For Cool Mode [°C] |
|  TOP34 | 40 | 80 | Convert to DEC 128-128 = 0  | Zone 2 water shift set Temperature For Heat Mode [°C] |
|  TOP35 | 41 | 8a | Convert to DEC 138-128 = 10  | Zone 2 water shift set Temperature For Cool Mode [°C] |
|  TOP9 | 42 | b2 | Convert to DEC 178-128 = 50  | Tank Target Temperature [°C] |
|  TOP45 | 43 | 71 | Convert to DEC 113-128 =-15   | Heat Shift for Holiday mode |
|  TOP25 | 44 | 71 | Convert to DEC 113-128 =-15  | Heat Shift for Tank mode  |
|  TOP | 45 | 97 | (hex) 96 = 97  | Maximum set pump speed |
|  TOP | 46 | 99 |   | ? |
|  TOP | 47 | 00 |   | ? |
|  TOP | 48 | 00 |   | ? |
|  TOP | 49 | 00 |   | ? |
|  TOP | 50 | 00 |   | ? |
|  TOP | 51 | 00 |   | ? |
|  TOP | 52 | 00 |   | ? |
|  TOP | 53 | 00 |   | ? |
|  TOP | 54 | 00 |   | ? |
|  TOP | 55 | 00 |   | ? |
|  TOP | 56 | 00 |   | ? |
|  TOP | 57 | 00 |   | ? |
|  TOP | 58 | 80 |   | ? |
|  TOP | 59 | 85 |   | ? |
|  TOP | 60 | 15 | Convert to DEC X-1   | Time set for external heaters 20min-3h, step 5min. |
|  TOP | 61 | 8a |   | ? |
|  TOP | 62 | 85 |  Convert to DEC | ? | 
|  TOP | 63 | 85 |   | ? |
|  TOP | 64 | d0 |   | ? |
|  TOP | 65 | 7b | Convert to DEC-128  | Outdoor Temperature to turn on Biwalent device -15-35[°C]|
|  TOP | 66 | 78 |   | ? |
|  TOP | 67 | 1f |   | ? |
|  TOP | 68 | 7e |   | ? |
|  TOP | 69 | 1f |   | ? |
|  TOP | 71 | 79 |   | ? |
|  TOP | 72 | 79 |   | ? |
|  TOP | 73 | 8d |   =13| ? |
|  TOP | 74 | 8d |   =13| ? |
|  TOP29 | 75 | 9e | Convert to DEC 158-128 =30 | Heating Curve Outlet Water Temperature Highest Set [°C] |
|  TOP30 | 76 | 96 | Convert to DEC 150-128 =22 | Heating Curve Outlet Water Temperature Lowest Set [°C] |
|  TOP32 | 77 | 71 | Convert to DEC 113-128 =-15 | Heating Curve Outside Temperature Lowest Set [°C] |
|  TOP31 | 78 | 8f | Convert to DEC 143-128 =15  | Heating Curve Outside Temperature Highest Set [°C] |
|  TOP | 79 | b7 | Convert to DEC 183-128 =55  | ? Possible Direct Zone2 water Temperature set [°C] |
|  TOP | 80 | a3 |   =35| ? |
|  TOP | 81 | 7b |   =-5| ? |
|  TOP | 82 | 8f |   =15| ? |
|  TOP | 83 | 8e | Convert to DEC-128  | Outdoor Temperature to stop heating 5-35 [°C] |
|  TOP23 | 84 | 80 | Convert to DEC 133-128 =5  | Floor heating set delta [°C] |
|  TOP | 85 | 80 | Convert to DEC 128-128=0 | ? Possible Outdoor temperature for heater ON [°C]  |
|  TOP | 86 | 8f | Convert to DEC 143-128 =15 | Set temperature for Cooling direct mode [°C] |
|  TOP | 87 | 8a |   =10| ?  Possible cooling curve |
|  TOP | 88 | 94 |  =20 | ? Possible cooling curve |
|  TOP | 89 | 9e |  =30 | ? Possible cooling curve  |
|  TOP | 90 | 8a |  =10 | ? |
|  TOP | 91 | 8a |  =10| ? |
|  TOP | 92 | 94 |  =20 | ? |
|  TOP | 93 | 9e |  =30 | ? |
|  TOP24 | 94 | 82 | Convert to DEC 130-128 =2  | Floor cooling set delta [°C] |
|  TOP | 95 | 90 | Convert to DEC 144-128=16|  | Outdoor temperature for (heat to cool)   [°C]  |
|  TOP | 96 | 8b |  Convert to DEC 139-128=11|  Outdoor temperature for (cool to heat) [°C] |
|  TOP | 97 | 05 | Donvert to DEC (X-1) x 30   | Tank settings - Room operation max time [min] |
|  TOP | 98 | 65 | Convert to DEC 101-1=100   | Tank heat up time (max) [min] |
|  TOP22 | 99 | 78 | Convert to DEC 120-128=-8 | DHW delta for re-heat  [°C] |
|  TOP | 100 | c1 |  Convert to DEC 193-128=65  | Sterilization boiling temperature [°C] |
|  TOP | 101 | 0b |  Convert to DEC 11 - 1 = 10 | Sterilization max operation time [min] |
|  TOP | 102 | 79 |   | ? |
|  TOP | 103 | 79 |   | ? |
|  TOP | 104 | 79 |   | ? |
|  TOP | 105 | 79 |   | ? |
|  TOP | 106 | 79 |   | ? |
|  TOP | 107 | 79 |   | ? |
|  TOP | 108 | 79 |   | ? |
|  TOP | 109 | 79 |   | ? |
|  TOP | 110 | 79 |   | ? |
|  TOP20+TOP26 | 111 | 56 |  right 2 bits: 0b10=Tank 0b01=Room 3-Way Valve. Next 2 bits (from right) is defrosting state (0b01 = defrosting not active, 0b10 = defrosting active) | 3 way valve + Defrost status |
|  TOP | 112 | 79 |  Hex 59 - external active, 55 - external and internal not active, 56 - internal active (room or tank) | Heater status |
|  TOP44 | 113 | 31 | Hex B1 - F type error, A1 - H type error. After H error reset value 21, F error reset 31  | Error code type |
|  TOP44 | 114 | 56 | F45 error in HEX 56, calulation 45 treat as HEX and convert to DEC 69 + 17 = 86 (Hex 56) | Error code number |
|  TOP | 115 | 79 |   | ? |
|  TOP | 116 | 79 |   | ? |
|  TOP | 117 | 79 |   | ? |
|  TOP | 118 | 79 |   | ? |
|  TOP | 119 | 79 |   | ? |
|  TOP | 120 | 79 |   | ? |
|  TOP | 121 | 79 |   | ? |
|  TOP | 122 | 79 |   | ? |
|  TOP | 123 | 79 |   | ? |
|  TOP | 124 | 79 |   | ? |
|  TOP | 125 | 79 |   | ? |
|  TOP | 126 | 79 |   | ? |
|  TOP | 127 | 79 |   | ? |
|  TOP | 128 | 79 |   | ? |
|  TOP | 129 | 79 |   | ? |
|  TOP | 130 | 79 |   | ? |
|  TOP | 131 | 79 |   | ? |
|  TOP | 132 | 79 |   | ? |
|  TOP | 133 | 79 |   | ? |
|  TOP | 134 | 79 |   | ? |
|  TOP | 135 | 79 |   | ? |
|  TOP | 136 | 79 |   | ? |
|  TOP | 137 | 79 |   | ? |
|  TOP | 138 | 79 |   | ? |
|  TOP6 | 139 | 79 |  to DEC-128 | Zone1: Actual (Water Outlet/Room/Pool) Temperature [°C] |
|  TOP | 140 | 79 |  to DEC-128 |  Zone2: Actual (Water Outlet/Room/Pool) Temperature [°C] |
|  TOP10 | 141 | 79 |  to DEC-128 | Actual Tank Temperature [°C] \ |
|  TOP14 | 142 | 79 |  to DEC-128 | Actual Outdoor Temperature [°C] |
|  TOP5 | 143 | 79 |  to DEC-128 | Zone1: Inlet Water Temperature [°C] |
|  TOP | 144 | 79 |  to DEC-128 | Zone1: Outlet Water Temperature [°C] |
|  TOP36 | 145 | 79 |  to DEC-128 | Zone1: Water Temperature [°C] |
|  TOP37 | 146 | 79 |  to DEC-128 | Zone2: Water Temperature [°C] |
|  TOP42 | 147 | 79 |  to DEC-128 | Zone1: Water Temperature (Target) [°C] |
|  TOP43 | 148 | 79 |  to DEC-128 | Zone2: Water Temperature (Target) [°C]  |
|  TOP46 | 149 | 79 |  to DEC-128 | Buffer Tank: Water Temperature [°C] |
|  TOP47 | 150 | 79 |  to DEC-128 | Solar: Water Temperature [°C]  |
|  TOP48 | 151 | 79 |  to DEC-128| Pool: Water Temperature [°C] |
|  TOP | 152 | 80 |   | ? |
|  TOP7 | 153 | 79 |  to DEC-128 | Zone1: Outlet Water Temperature (Target) [°C] |
|  TOP49 | 154 | 79 |  to DEC-128 | Outlet 2 Temperature [°C] (Sensor between heater and exchanger) |
|  TOP50 | 155 | 79 |  to DEC-128 | Discharge Temperature [°C] |
|  TOP33 | 156 | 79 |  to DEC-128 | Room Thermostat Internal Sensor Temperature [°C] |
|  TOP51 | 157 | 79 |  to DEC-128 | Indoor Piping Temperature [°C] |
|  TOP21 | 158 | 79 |  to DEC-128 | Outdoor Piping Temperature [°C] |
|  TOP52 | 159 | 79 |  to DEC-128 | Defrost Temperature [°C] |
|  TOP53 | 160 | 79 |  to DEC-128 | Eva Outlet Temperature [°C] |
|  TOP54 | 161 | 79 |  to DEC-128 | Bypass Outlet Temperature [°C] |
|  TOP55 | 162 | 79 |  to DEC-128 | Ipm Temperature [°C]  |
|  TOP | 163 | 79 |  to DEC (x-1)/5 |  High Pressure [Kgf/Cm2] |
|  TOP | 164 | 79 |  to DEC x-1 | Low Pressure [Kgf/Cm2] |
|  TOP | 165 | 79 |  to DEC (X-1)/5 | Outdoor Current [A] |
|  TOP8 | 166 | 79 | to DEC x-1  | Compressor Frequency [Hz] |
|  TOP | 167 | 79 |   | ? |
|  TOP | 168 | 79 |   | ? |
|  TOP1 | 169 | 79 | to DEC (X -1)/5 X 2 | 2nd Value for Pump Flow Rate [L/Min]  |
|  TOP1 | 170 | 79 | to DEC | 1st Value for Pump Flow Rate [L/Min] |
|  TOP | 171 | 79 | to DEC (X-1) X 100 /2  | Pump Speed [R/Min] |
|  TOP | 172 | 79 | to DEC X-1   | Pump Duty [Duty] |
|  TOP | 173 | 79 | to DEC (X-1) X10  | Fan Motor Speed 1 [R/Min |
|  TOP | 174 | 79 | to DEC x-1  | Fan Motor Speed 2 [R/Min] |
|  TOP | 175 | 79 |   | ? |
|  TOP | 176 | 79 |   | ? |
|  TOP | 177 | 79 |   | ? |
|  TOP | 178 | 79 |   | ? |
|  TOP12 | 179 | be | combine both bytes (180 byte) 08 (179 byte) be = 08be= 2238(DEC) - 1 = 2237  | number of operations |
|  TOP12 | 180 | 08 |  look at 179 | number of operations |
|  TOP | 181 | 79 |   | ? |
|  TOP11 | 182 | 25 | combine both bytes (183) 0b  (182) 25 = 2853 - 1 = 2852  | Operating time in h |
|  TOP11 | 183 | 0b | look at 182  | Operating time in h |
|  TOP | 184 | 79 |   | ? |
|  TOP | 185 | 79 | to DEC X-1  | Room Heater operation time in h |
|  TOP | 186 | 79 |   | ? |
|  TOP | 187 | 79 |   | ? |
|  TOP | 188 | 79 | to DEC X-1   | Tank Heater operation time in h  |
|  TOP | 189 | 79 |   | ? |
|  TOP | 190 | 79 |   | ? |
|  TOP | 191 | 79 |   | ? |
|  TOP | 192 | 79 |   | ? |
|  TOP16 | 193 | 01 | to DEC (x-1) / 5   | Energy Consumption for Heat in [kw]  |
|  TOP15 | 194 | 07 | to DEC (x-1) / 5   | Energy Generation for Heat in [kw] |
|  TOP38 | 195 | 79 | to DEC (x-1) / 5   | Energy Consumption for Cool in [kw] |
|  TOP39 | 196 | 79 | to DEC (x-1) / 5   | Energy Generation for Cool in [kw] |
|  TOP40 | 197 | 79 | to DEC (x-1) / 5   | Energy Consumption for DHW in [kw] |
|  TOP41 | 198 | 79 | to DEC (x-1) / 5   | Energy Generation for DHW in [kw] |
|  TOP | 199 | 79 |   | ? |
|  TOP | 200 | 79 |   | ? |
|  TOP | 201 | 79 |   | ? |
|  TOP | 202 | 79 |  CHECKSUM |  |




To get decimal values you must convert from hexadecimal and do some calulation depending on value. Most of them need just -128(DEC). \
As example 43 byte value to get DHW tank water set temperature b1 (HEX) = 177(DEC) - 128 = 49 C  \
Panasonic query, answer and commands are using 8-bit Checksum to verify serial data ( sum(all bytes) & 0xFF == 0 ). Last byte is checksum value.


## Query Examples:

Panasonic query:

`71 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 12`

Panasonic answer example:

`71 c8 01 10 56 55 62 49 00 05 00 00 00 00 00 00
00 00 00 00 19 15 11 55 16 5e 55 05 09 00 00 00
00 00 00 00 00 00 80 8f 80 8a b2 71 71 97 99 00
00 00 00 00 00 00 00 00 00 00 80 85 15 8a 85 85
d0 7b 78 1f 7e 1f 1f 79 79 8d 8d 9e 96 71 8f b7
a3 7b 8f 8e 85 80 8f 8a 94 9e 8a 8a 94 9e 82 90
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 55 56
55 21 53 15 5a 05 12 12 19 00 00 00 00 00 00 00
00 e2 ce 0d 71 81 72 ce 0c 92 81 b0 00 aa 7c ab
b0 32 32 9c b6 32 32 32 80 b7 af cd 9a ac 79 80
77 80 ff 91 01 29 59 00 00 3b 0b 1c 51 59 01 36
79 01 01 c3 02 00 dd 02 00 05 00 00 01 00 00 06
01 01 01 01 01 0a 14 00 00 00 77`

Quiet mode 1

`f1 6c 01 10 00 00 00 10 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 82`

Quiet mode 2

`f1 6c 01 10 00 00 00 18 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 7a`

Quiet mode 3

`f1 6c 01 10 00 00 00 20 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 72`

Quiet off

`f1 6c 01 10 00 00 00 08 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 8a`

Set -2C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7e 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 14`

set -1C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7f 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 13`

Set 0C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 12`

set -3C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7d 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 15`

set -4C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7c 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 16`

Set -5C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 17`

set +1C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 81 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 11`

set +2C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 82 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 10`

set +3C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 83 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 0f`

set +4C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 84 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 0e`

set +5C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 85 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 0d`

set tank to 48C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 b0 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e2`

set tank to 47C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 af 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e3`

set tank to 49C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 b1 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e1`

set tank to 40C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 a8 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 ea`

set tank to max 75C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 ca 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 c8`

heat on - tank off

`f1 6c 01 10 02 00 52 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 3e`

heat off - tank off (all off command)

`f1 6c 01 10 01 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 91`

heat off - tank on

`f1 6c 01 10 02 00 21 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 6f`

heat on - tank on

`f1 6c 01 10 02 00 62 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 2e`

set cool to 19C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 93 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 ff`

set cool to 18C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 92 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00`

set cool to 6C 

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 86 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 0c`


cool mode

`f1 6c 01 10 00 00 03 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 8f`


auto mode

`f1 6c 01 10 00 00 08 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 8a`


heat mode

`f1 6c 01 10 00 00 02 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 90`

From service cloud commands:

Tank mode only

`f1 6c 01 10 42 54 21 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e6`


heat only

`f1 6c 01 10 42 54 12 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 f5`



cool only

`f1 6c 01 10 42 54 13 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 f4`

auto

`f1 6c 01 10 42 54 18 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 ef`

heat + dhw

`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e5`


cool + dhw

`f1 6c 01 10 42 54 23 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e4`


auto + dhw


`f1 6c 01 10 42 54 28 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 df`


holiday mode ON

`f1 6c 01 10 42 64 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 d5`


holiday mode OFF

`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e5`


powerfull 30 min

`f1 6c 01 10 42 54 22 4a 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e4`


powerfull 60 min

`f1 6c 01 10 42 54 22 4b 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e3`



powerfull 90 min


`f1 6c 01 10 42 54 22 4c 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e2`


powerfull off


`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e5`



force DHW on

`f1 6c 01 10 82 54 21 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 a6`


Force DHW OFF

`f1 6c 01 10 42 54 21 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e6`

Heat pump Power ON

`f1 6c 01 10 42 54 12 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 94 9c 00 00 b1 80 80 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 bb`

Heat pump Power OFF

`f1 6c 01 10 41 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 8a 00 00 b1 80 80 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 d2`

Command sequence to set Zone & Sensor

Set Internal Thermostat

`f1 6c 01 10 01 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 91`

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 19 05 03 55 16 12 55 05 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 15 8a 85 85
d0 7b 78 1f 7e 1f 1f 79 79 8d 8d 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 47`


`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 19 05 03 55 16 12 55 05 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 15 8a 85 85
d0 7b 78 1f 7e 1f 1f 79 79 8d 8d 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 47`

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 92`


Set Water Temperature

sends off command  (same pattern with other setting commands)

`f1 6c 01 10 01 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 91`

sets value

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 19 05 01 55 16 12 55 05 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 15 8a 85 85
d0 7b 78 1f 7e 1f 1f 79 79 8d 8d 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 49`

this should be confirmation (same pattern with other setting commands)

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 92`


Room Sensor External

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 19 05 02 55 16 12 55 05 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 15 8a 85 85
d0 7b 78 1f 7e 1f 1f 79 79 8d 8d 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 48`

Room Sensor Thermistor

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 19 05 04 55 16 12 55 05 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 15 8a 85 85
d0 7b 78 1f 7e 1f 1f 79 79 8d 8d 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 46`

Operation Setup --> Installer

Water temperature for heating on Compensationio Curve

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 89 85 80 8a 8a 94 9e 8a 00 00 00 82 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 a8`
 

Heat Direct

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 0a 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f b7
00 00 00 89 85 80 8a 8a 94 9e 8a 00 00 00 82 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 f0`



Water temperature for Cooling on Compensation curve set

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 05 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 89 85 80 8a 8a 94 9e 00 00 00 00 82 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 36`

Cool Direct

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 89 85 80 8a 8a 94 9e 8f 00 00 00 82 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 a3`

set Delta T for heating 6c

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 89 86 80 8a 8a 94 9e 8f 00 00 00 82 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 a2`

outdoor temperature for heating off 10C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 82 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 a2`


delta T for cooling 3C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 a1`


cool zone1 temperature set dircet 20C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 94 8a 94 9e 8f 00 00 00 83 94
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 97`

Auto mode
Outdoor temperature mode for (heat to cool) 21C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 95
8b 05 65 78 c1 0b 00 00 00 00 00 00 00 00 a0`


for cool to heat set 12C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 95
8c 05 65 78 c1 0b 00 00 00 00 00 00 00 00 9f`

Tank settings

Room Operation time maximum 90 min

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 95
8c 04 65 78 c1 0b 00 00 00 00 00 00 00 00 a0`

tank heat up tank maximum 105 min

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 95
8c 05 6a 78 c1 0b 00 00 00 00 00 00 00 00 9a`

sterilization boiling temp set 66C

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 95
8c 05 6a 78 c2 0b 00 00 00 00 00 00 00 00 99`

set sterilization time 15 min

`f1 6c 01 10 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 09 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 9e 96 71 8f 00
00 00 00 8a 85 80 8a 8a 94 9e 8f 00 00 00 83 95
8c 05 6a 78 b7 10 00 00 00 00 00 00 00 00 9f`

better to use these instead from user cloud

quiet off


`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e5`


quiet 1


`f1 6c 01 10 42 54 22 51 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 dd`


quiet 2

`f1 6c 01 10 42 54 22 59 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 d5`



quiet 3

`f1 6c 01 10 42 54 22 61 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 cd`

Force defrost request

`f1 6c 01 10 42 54 22 49 02 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e3`

Sterilization request

`f1 6c 01 10 42 54 22 49 04 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 80 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e1`

Weekly timer on (byte 5 changes)

`f1 6c 01 10 42 94 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 aa`

Weekly timer off (byte 5 changes)

`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 ea`

Quiet timer on (byte 7 changes)

`f1 6c 01 10 42 94 22 89 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 6a`


Quiet timer off (byte 7 changes)

`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 ea`

Tank heater on (byte 9 changes)

`f1 6c 01 10 42 54 22 49 00 09 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 e6`

Tank heater off (byte 9 changes)

`f1 6c 01 10 42 54 22 49 00 05 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 7b 94 00 00 b1 71 71 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 ea`

Error reset (reboot)

`f1 6c 01 10 00 00 00 00 01 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
00 00 00 00 00 00 00 00 00 00 00 00 00 00 91
`
